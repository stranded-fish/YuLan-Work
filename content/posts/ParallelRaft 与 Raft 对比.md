---
title: "PolarFS-ParallelRaft 与 Raft 对比"
date: 2022-06-21T11:34:00+08:00
draft: false
tags: ["Raft", "Consensus Algorithm", "Distributed System"]
slug: "ParallelRaft vs Raft"
---

ParallelRaft 是 PolarFS 基于 Raft 开发的一种能够允许乱序执行的共识协议，该协议打破了 Raft 需要严格序列化的约束，并继承了 Raft 的可理解性和易实现性，得益于该协议 PolarFS 的 I/O 并发性得到了显著的提升。

本文主要根据 [PolarFS 论文](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf) 介绍 ParallelRaft 与 Raft 的区别与联系。

目录：

- [应用场景介绍](#应用场景介绍)
- [朴素 Raft 缺点](#朴素-raft-缺点)
- [乱序 Apply 前提](#乱序-apply-前提)
- [ParallelRaft 模块](#parallelraft-模块)
  - [日志复制](#日志复制)
    - [Raft 串行化要求](#raft-串行化要求)
    - [ParallelRaft 乱序实现](#parallelraft-乱序实现)
    - [ParallelRaft 乱序示例（N = 2）](#parallelraft-乱序示例n--2)
  - [Leader 选举](#leader-选举)
    - [Raft 选举](#raft-选举)
    - [ParallelRaft 选举](#parallelraft-选举)
  - [Catch Up](#catch-up)
    - [Raft 快照](#raft-快照)
    - [ParallelRaft 实现](#parallelraft-实现)
- [ParallelRaft 正确性](#parallelraft-正确性)
- [参考资料](#参考资料)

## 应用场景介绍

PolarFS 使用 ParallelRaft 协议复制 ChunkServer 的 I/O 请求并形成共识组。一个 ChunkServer 可能因为各种原因离开其共识组，需要根据具体的原因进行处理：

* 由偶然和临时故障引起的，例如网络暂时不可用，或服务器升级并重新启动。这种情况最好等待断开的 ChunkServer 重新上线，重新加入并赶上其他人；
* 故障是永久性的或往往会持续很长时间，例如服务器损坏或脱机。这时候应该将对应的 ChunkServer 上的所有 chunk 迁移到其他 ChunkServer 上，以重新获得足够数量的副本。

**补充：其他复制日志的实现方案**

**eg 1. 基于语句的复制**

主节点记录所执行的每个写请求（操作语句）并将该操作语句作为日志发送给从节点。对于关系数据库，这意味着每个 INSERT、UPDATE 或 DELETE 语句都会转发给从节点，然后每个从节点都会分析并执行这些 SQL 语句。

缺点，这种复制方式不适用以下场景：

* 任何调用非确定性函数的语句，如 `NOW()` 获取当前时间，或 `RAND()` 获取一个随机数等，可能会在不同的副本上产生不同的值。
* 如果语句中使用了自增列，或者依赖于数据库的现有数据（例如，`UPDATE ... WHERE <某些条件>`），则所有副本必须按照完全相同的顺序执行，否则可能会出现不同的结果。进而，如果有多个同时并发执行的事务时，会有很大的限制。
* 有副作用的语句（例如，触发器、存储过程、用户定义的函数等），可能会在每个副本上产生不同的副作用。

解决方法：

主节点可以在记录操作语句时将非确定性函数替换为执行之后的确定的结果，以保证所有节点直接使用相同的结果值。但该方法，需要考虑非常多的边界条件，因此基于语句的复制不是当前首选的复制实现方法。

> MySQL 基于 SQL 语句（binlog_format = STATEMENT）级别的 binlog 采用的就是基于语句的复制，该格式记录日志的逻辑 SQL 语句。

**eg 2. 基于预写日志（WAL）传输**

对数据库写入的字节序列都被记入日志。因此可以使用完全相同的日志在另一个节点上构建副本，即主节点除了将日志写入磁盘之外，还通过网络将其发送给从节点。

缺点：

预写日志（WAL）描述的数据非常底层：包含哪些磁盘块的哪些字节发生改变等。这使得复制方案和存储引擎紧密耦合，系统通常无法支持主从节点上运行不同版本的软件。

> MySQL InnoDB 存储引擎基于 redo、undo 日志实现 WAL。

**eg 3. 基于行的逻辑日志复制**

复制和存储引擎采用不同的日志格式，将复制与存储逻辑剥离。这种复制日志称为逻辑日志，以区分物理存储引擎的数据表示。

关系数据库的逻辑日志通常是指一系列记录来描述数据表行级别的写请求（包括：行插入、行删除、行更新），如果一条事务涉及多行的修改，则会产生多条这样的日志记录，并在后面跟着一条记录，指出该事务已经提交。

优点：

* 逻辑日志与存储引擎解耦，因此可以更加容易地保持向后兼容，从而使主从节点能够运行不同版本的软件甚至是不同的存储引擎。
* 对于外部应用程序而言，逻辑日志格式将更容易解析。

> MySQL ROW 格式下（binlog_format = ROW）binlog 采用的就是基于行的逻辑日志复制，该格式记录表的行更改情况。

**eg 4. 基于触发器的复制**

`eg 1-3` 的复制方法均是由数据库系统实现的，不涉及任何应用程序代码。但是，在某些情况下，可能需要更高的灵活性，例如：

* 只想复制数据的一部分；
* 从一种数据库复制到另一种数据库；
* 需要订制、管理冲突解决逻辑，此时需要将复制控制权交给应用程序层。

实现方法：

* 部分工具，如 Oracle GoldenGate，可以通过读取数据库日志让应用程序获取数据变更；
* 大部分关系型数据库支持的功能：触发器和存储过程。
  * 触发器支持注册自己的应用层代码，使得当数据库系统发生数据更改（写事务）时自动执行上述自定义代码，触发器优缺点：
    * 优点：高度灵活性。
    * 缺点：通常比其他复制方式开销更大，也比数据库内置复制更容易出错，或者暴露一些限制。

## 朴素 Raft 缺点

Raft 被设计为高度序列化，以达到简单和易于理解的目的。leader 和 follower 上的日志都不允许有空洞，这意味着 **日志项在 follower 确认（ack），leader 提交（commit）及应用（apply）到状态机时都是严格顺序执行的。**

因此当 follower 收到乱序到达的日志时，在前面所有缺失的日志项都按序收到之前，队列尾部（提前到达的日志项）的那些请求都将无法响应，这会 **增加平均延迟并降低吞吐量**。而对于高并发的环境来说，由于网络拥塞出现数据包乱序到达的情况是很常见且无法避免的。

## 乱序 Apply 前提

MySQL、AliSQL 等数据库并不关心底层存储的 I/O 顺序。数据库的锁机制可以保证在任何时间点，只有一个线程可以在一个特定的 page 上工作。当不同的线程同时在不同的 page 上工作时，数据库只需要成功执行 I/O，它们的完成顺序无关紧要。

PolarFS 正是利用这一点放宽 Raft 中的严格顺序性限制，开发出更适合高 I/O 并发的 ParallelRaft。

> 页（page）是 MySQL InnoDB 存储引擎磁盘管理的最小单位，每个页默认 16 KB。

## ParallelRaft 模块

### 日志复制

#### Raft 串行化要求

* follower 确认（ack）：leader 发送一条日志项到 follower 之后，follower 需要对此进行确认，表明这条日志项被收到和记录了，同时也显式地表明之前所有的日志项都已经收到和记录了。
* leader 提交（commit）：当 leader 提交一条日志项，并将此事件广播给所有的 follower ，也表明之前所有的日志项都已经提交了。

与此同时，以上两方面的串行化也保证了 Raft 节点本地日志条目的串行化，最终应用层状态机将严格按照日志顺序进行应用（apply）。

#### ParallelRaft 乱序实现

**ParallelRaft 打破了上述限制，使其可以乱序确认（ack），leader 提交（commit）及应用（apply）。**

**因此，ParallelRaft 和 Raft 有些本质区别，如：**

* ParallelRaft 中 follower 在收到日志项时，不必再根据 `preLogIndex` 判断是否乱序，同时 follower 确认该日志项，也不再代表已经收到和记录了之前的日志项。
* ParallelRaft 中 leader 在确认某一日志项获得超半数确认，可以提交时，并不意味着之前的所有项都已经成功提交。即 `last_committed_index` 失去了原有的语义。

**为了确保协议的正确性，需要保证：**

* 去掉严格序列化的限制后，所有的提交状态应该不违反传统关系型数据库中的存储语义。
* 所有已经提交的修改在任何极端场景下都不会丢失。

**乱序执行规则：**

ParallelRaft 的乱序日志执行遵循以下规则：如果日志项的写入范围彼此没有交集，则认为这些日志条目没有冲突，可以按任意顺序执行。否则，有冲突的项将严格按照到达顺序执行。这样，较新的数据永远不会被旧版本覆盖。

**冲突判断规则：**

ParallelRaft 可以很容易地知道冲突，因为它存储了所有未应用的日志项的 LBA 范围统计。

**执行步骤：**

以下部分描述了如何优化 ParallelRaft 的 Ack-Commit-Apply 步骤以及如何维持必要的一致性语义。

**Out-of-Order Acknowledge：**

* Raft follower 在收到 leader 复制的日志项后，要等到所有之前的日志条目都持久化存储后才会确认（ack），这会引入额外的等待时间，并且当有大量并发 I/O 写入执行时，平均延迟会显著增加。
* 但是，在 ParallelRaft 中，一旦日志条目写入成功，follower 就可以立即确认，这样就避免了额外的等待时间，从而优化了平均延迟。

**Out-of-Order Commit：**

* Raft leader 按顺序提交日志条目，一条日志项只有在它之前所有的日志项提交后，才能提交。
* 而在 ParallelRaft 中，可以在大多数副本确认后就立即提交（commit）日志项。

**Apply with Holes in the Log：**

* Raft 中，所有日志项都严格按照 Raft 中记录日志的顺序被应用（apply），因此数据文件在所有副本中都是一致的。
* 然而，通过乱序的日志复制和提交，ParallelRaft 允许日志中存在空洞。
  * 为确保在某些先前的日志项仍然缺失的情况下安全地应用某条日志项，ParallelRaft 在每个日志条目中，引入了一个名为 look behind buffer 的新数据结构来解决这个问题。look behind buffer 包含被之前的 N 个日志项修改过的 LBA，look behind buffer 在日志中可能存在的空洞的情况下，扮演了沟通桥梁的角色。
  * N 是这座桥的跨度，同时也是允许的最大空洞的数量。请注意，尽管日志中可能存在多个空洞，但所有日志项的 LBA 统计始终是完整的，除非某些空洞的大小大于 N。
  * 通过这个数据结构，follower 可以判断一个日志项是否有冲突，冲突意味着这个日志项修改的 LBAs 与一些它之前缺失的某些日志项之间有交集。
  * 可以安全地应用与任何其他条目不冲突的日志项，否则应该将这些日志项添加到待处理列表中，并在之前缺失的日志项被应用后再进行处理。
  * 根据我们使用 RDMA 网络的 PolarFS 的经验，N 设置为 2 足以满足其 I/O 并发性。

#### ParallelRaft 乱序示例（N = 2）

![ParallelRaft 乱序示例](https://yulan-img-work.oss-cn-beijing.aliyuncs.com/img/202205061624750.png)

* 顺序提交
  * LogEntry 1、2、3 顺序到达 Follower，可直接同朴素 Raft 流程一样顺序提交。
* 乱序提交
  * LogEntry 5 只被 Follower 1 收到，此时针对 Follower 1 而言，虽然没有收到 LogEntry 4，但 LogEntry 5 所附带的 look behind buffer 结构记录了 LogEntry 3、4 所修改的 LBA，并且经过与自己所要修改的 LBA 进行比较发现没有冲突，故可以返回 Leader ACK，而 Leader 在收到 Follower 1 的 ACK 后，满足了过半数确认要求，即可 commit LogEntry 5。
  * LogEntry 6 同理，由于 LogEntry 6 附带了丢失的 LogEntry 4、LogEntry 5 的信息，且发现没有冲突地址，故可以直接返回 ACK。
  * 针对 Follower 1 收到的 LogEntry 12，由于之前 LogEntry look behind buffer 维护了所有的缺失的 Entry 所涉及到的 LBA，故 LogEntry 12 可以判断出自己是否跟之前的缺失 Entry 有冲突。
* 乱序冲突
  * 针对 Follower 1 收到的 LogEntry 9，假设它与自己所记录 LogEntry 8 所涉及的 LBA，不冲突，但是与 LogEntry 5 所记录的 LogEntry 4 所涉及到的 LBA 冲突，那么其不能被提交，只能被放入到待处理队列中。
  * 针对 Follower 2 收到的 LogEntry 11，它与前一个收到的日志 7 的中间空缺超过了 2，此时 11 并不能知道自己是否跟 LogEntry 8 冲突，故不能返回 Leader 确认，同样只能放入到待处理队列中。
  * 针对 Follower 2 收到的 LogEntry 12，同样因为无法确定是否跟 LogEntry 8 冲突，也不能返回确认。

图中日志状态说明：

* 针对已 commit 日志，ParallelRaft 应用层可以安全应用（apply），不用担心冲突。
* 针对待处理列表中的冲突日志，Follower 会等到之前缺失的日志项被应用后再处理。
* 针对带决议日志，Leader 会不断重发，直到能够 commit，同时会维护一个窗口保证其大小可控。

### Leader 选举

#### Raft 选举

Raft 通过投票的方式保证新 leader 在当选时就包含了之前所有任期号中已经提交的日志条目。通过这种保证，大大简化了 Raft 选举阶段实现，Raft 的日志条目的传送始终是单向的，只从 leader 发送到 follower。

具体实现：

* 候选人为了赢得选举必须联系集群中的过半数节点，这意味着每一个已经提交的日志条目在这些服务器节点中肯定存在于至少一个节点上（抽屉原理，两个过半数集合必有交集）。
* 如果候选人的日志至少和大多数的服务器节点一样新，那么他一定持有了所有已经提交的日志条目。关于日志新的定义：
  * 如果任期号相同，那么任期大的日志更新，即 term 更大的日志；
  * 如果任期号不同，那么日志比较长的那个就更加新，即 last_log_index 更大的日志。

#### ParallelRaft 选举

在进行新的 leader 选举时，ParallelRaft 和 Raft 一样，选择包含最新数据项、拥有的日志项最多的节点。但是，由于日志中可能存在空洞，ParallelRaft 中选出的 leader 开始可能无法到达这个要求。

因此，在开始处理请求之前，需要一个额外的合并阶段（merge stage）来使 leader 拥有之前已提交的所有条目。在合并阶段完成之前，新选出的节点只是 leader 的候选，当合并阶段执行完成，新选出的节点拥有所有之前已提交的项，它变成真正的 leader 。

**merge 阶段：**

在合并阶段，leader 候选人需要合并来自共识组其他成员的，之前不可见的项。之后，leader 开始将之前任期内的数据提交到大多数，这与 Raft 相同。

在执行合并的时候，ParallelRaft 也使用了和 Raft 类似的机制。具有相同 term 和 index 的条目将被视为相同的日志项，term 和 index 的概念参考 Raft 协议。

有几种异常情况：

* 对于一个已提交（提交指已被共识组中大多数节点确认）但不在 leader 中的项，leader 候选人总是可以从至少一个 follower 中找到，因为这个提交的项已被大多数接受。

* 对于没有在任何一个候选上提交的项，如果这个项也没有被任何一个候选保存，则 leader 可以安全地跳过，因为根据 ParallelRaft 或 Raft 机制，这个项不可能已经被提交。

* 如果某些候选人保存了一个未提交的项（ index 相同但 term 不同），则 leader 候选人选择其中其中 term 版本最高的，并认为该项有效。

    1. ParallelRaft 的合并阶段必须在新的 leader 可以为用户请求服务之前完成，如果版本更高的 term 被设置到一个条目，那么具有相同 index 但 term 版本更低的项之前一定没有被提交，并且较低 term 的条目一定从未参与过之前成功完成的合并阶段，否则 term 更高的条目不可能具有相同的 index。

    2. 当系统崩溃时，如果候选人的保存了未提交项，该项的确认可能已经发送给之前一个 leader 并返回给了用户，所以我们不能简单地放弃它，否则用户数据可能会丢失。 （更准确地说，如果失败的节点的总数加上拥有此未提交条目（具有相同 index 的条目的 term 版本最高的）的节点总数超过同一共识组中剩余节点的数量，则此条目可能已由之前的 leader 提交。因此，为了用户数据安全，我们应该提交它。）

### Catch Up

#### Raft 快照

针对朴素 Raft 而言，如果一个严重滞后的 follower 想要追赶上 leader（此时，leader 可能已经丢弃掉了 follower 所需要的下一条日志），leader 将会发送安装快照 InstallSnapshot RPC 以帮助 follower 快速更新到最新状态。

Raft 快照主要包括：

* 当前状态机的状态；
* 少量元数据（用于快照后的下一条日志的顺序检查）：
  * last_include_index：快照所保存的最后的日志条目的索引值
  * last_include_term：该日志条目的任期号

#### ParallelRaft 实现

针对 ParallelRaft 而言，当一个滞后的 follower 想要跟上 leader 的当前数据状态时，它会使用 `fast-catch-up` 和 `streaming-catch-up` 两种方法来与 leader 进行同步，具体使用哪一个取决于 follower 的状态有多旧：

* fast-catch-up - 当 leader 与 follower 之间的差距较小
  * 增量同步，此时 leader 通过 look behind buffer 帮助 follower 填补空洞。
* streaming-catch-up - 当 leader 与 follower 之间的差距较大
  * 全量同步，此时 leader 的检查点比 follower 最新日志更新，比 leader 检查点更旧的日志，没有同步的意义，leader 将直接使用检查点之后的数据块和日志条目的内容来重建状态。

## ParallelRaft 正确性

Raft 为了协议的正确性需要包含以下属性：

* Election Safety
  * 对于给定 Term，最多只会有一个 Leader 被选举出来。
* Leader Append-Only
  * Leader 绝不会删除或者覆盖自己的 Log，只会 Append。
* Log Matching
  * 如果两个 Log 拥有相同的 Term 和 Index，那么他们拥有相同的内容。
  * 如果两个 Log 拥有相同的 Term 和 Index，那么之前的 Log 也都是一样的。
* Leader Completeness
  * 如果一条 Log 在某个 Term 下被 Commit 了，那么这条 Log 必然存在于后面 Term 的 Leader 中。
* State Machine Safety
  * 如果一个节点已经 Apply 了一条 Log 到状态机，那么其他节点不会向状态机中 Apply 相同 Index 下的不同的 Log。

由于 ParallelRaft 主要引入了乱序提交，其他部分继承了原 Raft 设计，故同样拥有 Election Safety, Leader Append-Only, Log Matching 属性。

**针对 Leader Completeness：**

由于 ParallelRaft 可以乱序提交，故其日志中可能存在空洞，缺失部分的日志。因此 ParallelRaft 为 leader 选举增加了 merge 阶段。在 merge 阶段，leader 会从 follower 处复制缺失的日志条目，并重新确认这些条目是否应该提交。在合并阶段之后，Leader Completeness 可以得到保证。

**针对 State Machine Safety：**

虽然 ParallelRaft 允许节点独立执行日志的乱序应用，但是由于有 look behind buffer，有冲突的日志项只能严格按顺序执行，这意味着同一个共识组中的所有节点的状态机（数据以及已提交的日志项）是一致的。

## 参考资料

* [PolarFS: An Ultra-low Latency and Failure Resilient Distributed File System for Shared Storage Cloud Database](https://www.vldb.org/pvldb/vol11/p1849-cao.pdf)
* [阿里云PolarDB及其存储PolarFS技术实现分析](https://zhuanlan.zhihu.com/p/44874330)
* 《数据密集型应用系统设计》

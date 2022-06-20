---
title: "C++ 集合"
date: 2022-02-23T20:39:00+08:00
draft: false
tags: ["C++ STL", "C++"]
slug: "C++ set"
---

目录：

- [set](#set)
  - [元素访问](#元素访问)
- [unordered_set](#unordered_set)
  - [声明及初始化](#声明及初始化)
  - [遍历集合](#遍历集合)
  - [基本操作](#基本操作)
- [参考链接](#参考链接)

## set

`set` 是按照特定顺序存储唯一元素的容器。在集合 `set` 中，元素的值同时也是其键，必须是唯一的。**集合中元素的值始终为 const 不能修改，但可以插入或删除它们。**

在集合 `set` 内部，集合中的元素始终按照其内部比较对象（比较类型）指示的特定严格弱排序标准进行排序。所以只要求其保存的元素类型可以比较即可，不需要实现哈希函数，**故 `set` 相较于 `unordered_set` 的一大优点就是：可以保存 `pair`、`vector` 或是其他可以进行比较，但没有实现哈希函数的类型。**

### 元素访问

```C++
/* 1. 返回指向集合 首个 / past-the-end 元素的 iterator 指针
注意：past-the-end 不指向任何元素，故不能被解引用 */
      iterator begin() noexcept;
const_iterator begin() const noexcept;
      iterator end() noexcept;
const_iterator end() const noexcept;

// 示例
cout << *(--ss.end());            // 输出 set 最后一个元素
auto it = ss.end();
--it; --it;
cout << *it;                      // 输出 set 倒数第 2 个元素

/* 2. 反向迭代器，rbegin() 指向最后一个元素，rend() 指向第一个元素之前的理论元素，不可解引用 */
      reverse_iterator rbegin() noexcept;
const_reverse_iterator rbegin() const noexcept;
      reverse_iterator rend() noexcept;
const_reverse_iterator rend() const noexcept;

// 示例
cout << *ss.rbegin();    // 输出 set 最后一个元素
cout << *(++ss.rbegin()) // 输出 set 倒数第 2 个元素
```

> **注意：**
> **①** `set` 没有类似于 `vector` 的 `front()、back()` 方法，只能通过迭代器访问首尾元素。
>
> **②** `set` 为关联式容器，不能像 `vector` 之类的序列式容器那样实现随机访问（Random Access），
故其迭代器不能使用 `+=、-= 、- 、+` 运算符，只能使用 `++、--` 运算符。
>
> **③** `set` 迭代器指针的解引用返回值为常量，不能修改。

## unordered_set

`unordered_set` 是以无特定顺序存储唯一元素的容器，并且允许根据它们的值快速检索单个元素。在 `unordered_set` 中，**元素的值同时也是它的键，唯一地标识它，是不可变的，但是可以插入和删除它们。**

在 `unordered_set` 内部，`unordered_set` 中的元素没有按任何特定顺序排序，而是根据它们的哈希值组织成桶。**而由于 `pair`、`vector` 等类型没有实现默认哈希函数，故不能用 `unordered_set` 进行保存。**

### 声明及初始化

使用集合 `unordered_set` 时，需引入头文件 `<unordered_set>`。

```C++
#include <unordered_set>

// 初始化一个空的 set
unordered_set<int> set;

// 通过初始化列表中的元素，对其进行初始化
unordered_set<char> s{'a', 'b', 'c', 'd', 'e', 'f', 'g'};

// 范围构造函数 - 插入 [first,last) 范围内的元素
unordered_set<int> s(arr.begin(), arr.end());
```

### 遍历集合

**eg 1. 基于范围的 for 循环遍历**

```C++
// type val
for (auto val : set) {
    cout << val << endl;
}
```

> **注意：** 不能通过基于范围的 for 循环遍历修改集合元素，即使采用 for (auto &val : set) 形式，set 返回的将是 const type &val，即指向常量的引用，同样不能修改。

**eg 2. 基于迭代器遍历**

通过迭代器 `iterator` 遍历。

```C++
unordered_set<int> set;

// 可用 auto 关键字简化 auto iter = set.begin()
for (unordered_set<int>::iterator iter = set.begin(); iter != set.end(); ++iter) {
    cout << *iter << " ";
}
```

### 基本操作

```C++
// 返回集合中的元素个数
size_type size();

// 返回集合是否为空，如果为空返回 1，否则返回 0
bool empty();

// 如果 key 存在返回 1，否则返回 0
size_type count();

// 如果 key 存在，则返回一个指向该元素的迭代器，否则返回指向 unordered_set::end
      iterator find ( const key_type& k );
const_iterator find ( const key_type& k ) const;

// 向集合中插入一个元素 key
pair<iterator, bool> insert(const key_type& key);

// 删除集合中的元素 key，如果删除成功则返回 1，否则返回 0
size_type erase(const key_type& key);

// 将集合 size 设置为 0，但 capacity 不变（注意：不能用于清空集合）
void clear();
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)

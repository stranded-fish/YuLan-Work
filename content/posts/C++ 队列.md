---
title: "C++ 队列"
date: 2022-02-19T19:01:00+08:00
draft: false
tags: ["C++"]
slug: "C++ queue"
---

目录：

- [普通队列](#普通队列)
  - [声明及初始化](#声明及初始化)
  - [基本操作](#基本操作)
- [双端队列](#双端队列)
  - [声明及初始化](#声明及初始化-1)
  - [基本操作](#基本操作-1)
- [优先队列](#优先队列)
  - [声明及初始化](#声明及初始化-2)
  - [基本操作](#基本操作-2)
  - [自定义排序](#自定义排序)
- [参考链接](#参考链接)

使用队列 `queue` 或 优先队列 `priority_queue` 时，需引入头文件 `<queue>`。

## 普通队列

### 声明及初始化

```C++
#include <queue>

queue<int> q;
```

### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type size();

// 将元素加入队尾
void push(const value_type& val);
void emplace(Args&&... args);

// 返回队头元素的引用
value_type& front();

// 删除队头元素
void pop();
```

> **注意：** `pop()` 方法为 `void` 类型，不会返回被删除的元素，所以元素出队的一般做法为：
> `int val = q.front(); q.pop();`

## 双端队列

### 声明及初始化

```C++
#include <deque>

deque<int> dq;

// 范围构造函数 - 按序插入 [first,last) 范围内的元素
deque<int> dq(arr.begin(), arr.end());
```

### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type size();

// 添加元素
void push_back(const value_type& val);
void push_front(const value_type& val);
void emplace_back(Args&&... args);
void emplace_front(Args&&... args);

// 返回元素引用
reference back();
reference front();

// 删除元素
void pop_back();
void pop_front();

// 将队列 size 设置为 0，但 capacity 不变（注意：不能用于清空队列）
void clear();
```

`deque` 拥有 `begin()`、`end()` 方法，可以通过迭代器进行遍历。

## 优先队列

### 声明及初始化

定义：`priority_queue<Type, Container, Functional>`

* `Type`：数据类型
* `Container`：容器类型（默认 vector）
* `Functional`：比较方法的指针类型

```C++
#include <queue>

/* 对于基础类型，默认为大顶堆 - 降序队列
等价于 priority_queue<int, vector<int>, less<int>> q; */
priority_queue<int> q; 

// 小顶堆 - 升序队列
priority_queue<int, vector<int>, greater<int>> q;

// 大顶堆 - 降序队列
priority_queue<int, vector<int>, less<int>> q;

// 范围构造函数 - 插入 [first,last) 范围内的元素，并通过 make_heap 排序
vector<int> nums;
priority_queue<int, vector<int>, less<int>> q(nums.begin(), nums.end());
```

> **注意：**
> **①** 一般只有在使用自定义的数据类型时，才需要传入这三个参数，使用基本数据类型时，只需传入数据类型，**默认为大顶堆 - 降序队列。**
>
> **②** 自定义排序的大小比较与 sort 方法相反，同时需要对 sort 与 priority_queue 的自定义排序实现加以区分。
>
> * sort(arr.begin(), arr.end(), cmp) 方法的第三个参数 cmp 为可调用对象，即函数、函数指针、重载了函数调用运算符的类对象和 lambda 表达式。
> * 而 priority_queue<Type, Container, Functional> 中 Functional 为比较函数的指针类型，而非函数对象。可以通过 decltype(cmp)* 方法，声明函数指针类型，然后在构造函数中传递函数对象。即：
> `priority_queue<int, vector<int>, decltype(cmp)*> q(cmp);`

### 基本操作

```C++
// 返回队列是否为空
bool empty();

// 返回队列中元素的个数
size_type size();

// 将元素加入队列，并排序
void push(const value_type& val);
void emplace(Args&&... args);

// 返回队头元素的引用
value_type& top();

// 删除队头元素
void pop();
```

> **注意：** 队列没有 vector 的 begin() 方法，故不能使用 基于范围的 for 循环遍历 和 基于迭代器的遍历。只能通过 while 判断 empty() 方法 或 根据 size() 大小进行 for 循环 来进行遍历。

### 自定义排序

**eg 1. 函数指针**

```C++
bool cmp (ListNode *a, ListNode *b) { return a->val < b->val; }

/* eg 1.1 */
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)*> q(cmp);

/* eg 1.2 */
auto pf = &cmp;
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> q(cmp);
```

**eg 2. lambda 表达式**

```C++
auto cmp = [](ListNode *a, ListNode *b) { return a->vl > b->val; };
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> q(cmp);
```

**eg 3. 重载函数调用运算符 ()**

```C++
struct cmp {
    bool operator () (ListNode* a, ListNode* b) {
        return a->val > b->val; // 升序，小顶堆
    }
};

// 注意：与 sort 方法相区分，此处传递的是类，而非实例化对象
priority_queue<ListNode*, vector<ListNode*>, cmp> pq; 
```

**eg 4. 重载 operator <**

```C++
bool operator < (ListNode a, ListNode b){
    return a.val < b.val; // 降序，大顶堆
}

priority_queue<ListNode> pq; 
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [C++ 优先队列(priority_queue)用法详解](https://blog.csdn.net/weixin_36888577/article/details/79937886)

---
title: "C++ 列表"
date: 2022-02-19T18:56:00+08:00
draft: false
tags: ["C++ STL", "C++"]
slug: "C++ list"
---

目录：

- [声明及初始化](#声明及初始化)
- [遍历列表](#遍历列表)
- [基本操作](#基本操作)
- [参考链接](#参考链接)

## 声明及初始化

使用 `list (STL list)` 时，需引入头文件 `<list>`。

```C++
#include <list>

/* eg 1 initialize */

// 初始化一个 int 类型的空列表 nums
list<int> nums;

// 初始化一个大小为 n 的列表 nums，列表中的值默认都为 0
list<int> nums(10);

// 初始化一个大小为 n 的列表 nums，其值全都为 2
list<int> nums(n, 2);

/* eg 2 make a copy */
list<int> list_1(list_2.begin(), list_2.end());
list<int> list_1(list_2);

/* eg 3 cast an array to a list */
int a[5] = {0, 1, 2, 3, 4};
list<int> nums(a, *(&a + 1));
```

## 遍历列表

**eg 1. 基于范围的 for 循环遍历**

```C++
for (int &val : my_list) {
    cout << val << endl;
}
```

**eg 2. 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto i = my_list.begin()
for (list<int>::iterator i = my_list.begin(); i != my_list.end(); ++i) {
    cout << *i << endl;
}
```

> **注意：** `list` 容器未提供下标操作符 `[]` 和 `at()` 成员函数，也没有提供 `data()` 成员函数，故不支持随机访问，不能通过下标循环遍历。

## 基本操作

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

// 将列表 size 设置为 0，但 capacity 不变（注意：不能用于清空列表）
void clear();

// 插入单个元素
iterator insert(const_iterator position, const value_type& val);

// 填充 n 个元素
iterator insert(const_iterator position, size_type n, const value_type& val);

// 插入 [first,last) 范围内的所有元素
iterator insert(const_iterator position, InputIterator first, InputIterator last);
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)

---
title: "C++ 堆栈"
date: 2022-02-19T19:14:00+08:00
draft: false
tags: ["C++"]
slug: "C++ stack"
---

目录：

- [声明及初始化](#声明及初始化)
- [基本操作](#基本操作)
- [参考链接](#参考链接)

## 声明及初始化

使用堆栈 `stack` 时，需引入头文件 `<stack>`。

```C++
#include <stack>

stack<int> s;
```

## 基本操作

```C++
// 返回堆栈是否为空
bool empty();

// 返回堆栈中元素的个数
size_type size();

// 在栈顶添加元素
void push(const value_type& val);

// 返回栈顶元素的引用
value_type& top();

// 删除栈顶元素
void pop();
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)

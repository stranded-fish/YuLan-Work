---
title: "Go 数组"
date: 2022-03-09T19:32:00+08:00
draft: false
tags: ["Go"]
slug: "Go array"
---

Go 语言中，数组是一个 **长度固定** 的数据类型，用于存储一段具有相同的类型的元素的连续块。数组一旦声明，其存储的数据类型和数组长度就不能改变了。数组的存储类型可以是内置类型，也可以是某种结构类型。

目录：

- [声明及初始化](#声明及初始化)
- [基本操作](#基本操作)
  - [数组拷贝](#数组拷贝)
- [函数传递](#函数传递)
- [遍历数组](#遍历数组)
  - [一维数组](#一维数组)
  - [二维数组](#二维数组)
- [参考资料](#参考资料)

## 声明及初始化

声明数组：

```go
/* 一维数组 */

// eg 1. 声明一个数组，并设置为零值
var array [5]int

// eg 2. 使用数组字面量声明数组
array := [5]int{1, 2, 3, 4, 5}
array := [5]int{1, 2}       // 1 2 0 0 0

// eg 3. 自动计算声明数组长度
array := [...]int{1, 2, 3, 4, 5}

// eg 4. 声明数组并指定特定元素的值
array := [5]int{1:10, 2:20} // 0 10 20 0 0 0
array := [...]int{10, 3:20} // 10 0 0 20

/* 多维数组 */

// eg 1. 声明二维数组
var array [3][2]int

// eg 2. 使用数组字面量声明数组
array := [3][2]int{{1, 1}, {2, 2}, {3, 3}}

// eg 3. 声明并初始化外层数组指定元素
array := [3][2]int{0:{1,1}, 2:{2,2}}

// eg 4. 声明并初始化内层数组指定元素
array := [3][2]int{1:{1:100}}
```

## 基本操作

### 数组拷贝

数组变量的类型包括数组长度和每个元素的类型。只有这两部分都相同的数组，才是类型相同的数组，才能相互赋值。

```go
var array1 [3]string
array2 := [3]string{"a", "b", "c"}
array1 = array2

// 多维数组同理
```

## 函数传递

**eg 1. 值传递**

在函数间传递数组是一个很大的开销。因为在函数之间传递变量时，总是以值的方式传递，**如果这个变量是数组，意味着整个数组都会完整的复制，并传递给函数。**

```go
var array [1e6]int

foo(array)

func foo(array [1e6]int) {
    ......
}
```

**eg 2. 指针传递**

```go
var array [1e6]int

foo(&array)

func foo(array *[1e6]int) {
    ......
}
```

## 遍历数组

### 一维数组

**eg 1.1 下标 for 循环遍历**

```go
array := [5]int{1, 2, 3, 4, 5}
for i := 0; i < len(array); i++ {
    fmt.Println(array[i])
}
```

**eg 1.2 基于 range 的 for 循环遍历**

```go
for idx, val := range array {
    fmt.Println(idx, val)
}

// 若不需要 idx 索引，可用 _ 形式忽略
for _, val := range array {......}
```

### 二维数组

**eg 2.1 下标 for 循环遍历**

```go
array := [5][2]int{{1, 1}, {2, 2}, {3, 3}, {4, 4}, {5, 5}}
for i := 0; i < len(array); i++ {
	for j := 0; j < len(array[i]); j++ {
		fmt.Println(array[i][j])
	}
}
```

**eg 2.2 基于 range 的 for 循环遍历**

```go
for idx1, val1 := range array {
	for idx2, val2 := range val1 {
		fmt.Println(idx1, idx2, val2)
	}
}
```

## 参考资料

* 《Go 语言实战》

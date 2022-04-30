---
title: "Go 映射"
date: 2022-03-10T22:13:00+08:00
draft: false
tags: ["Go"]
slug: "Go map"
---

Go 映射用于存储一系列 **无序的键值对**。无序的原因是 Go 映射的实现使用了散列表。

目录：

- [声明及初始化](#声明及初始化)
- [基本操作](#基本操作)
- [遍历映射](#遍历映射)
- [函数传递](#函数传递)
- [参考资料](#参考资料)

## 声明及初始化

```go
/* eg 1. make */
dict := make(map[string]int)

/* eg 2. 字面量 */
dict := map[string]string{"Red":"#da1337", "Orange":"#e95a22"}
```

**注意：** 映射的键可以是任何能够使用 `==` 运算符做比较的类型（内置的类型、结构类型等）。切片、函数以及包含切片的结构类型，由于具有引用语义，不能作为映射的键。

## 基本操作

```go
// 映射赋值
colors := map[string]string{}
colors["Red"] = "#da1337"

// 注意：不能对以下 nil 映射进行赋值
var colors map[string]string

/* 映射取值 */

// eg 1. 同时获得值，以及表示该值是否存在的标志
value, exists := colors["Blus"]
if exists {
    fmt.Println(value)
}

// eg 2. 只返回值，判断该值是否是零值来确定键是否存在
value := colors["Blus"]
if value != "" {
    fmt.Println(value)
}

// 注意：Go 映射，通过键来索引映射时，即使该键不存在也总会返回一个该值对应的零值。所以 eg 2. 只能用在映射存储 值 都是非零值的情况。

// 删除键值对
delete(colors, "Blue")
```

## 遍历映射

对映射来说，`range` 返回的是键值对。

```go
for key, value := range colors {
    fmt.Println(key, value)
}
```

## 函数传递

```go
colors := map[string]string {
    ......
}

func myFunc(colors map[string]string) {
    ......
}

myFunc(colors)
```

映射函数传递特性和切片类似，不会制造一个副本，而是以较小的代价复制，同时在函数中修改会影响到原映射。

## 参考资料

* 《Go 语言实战》

---
title: "C++ double"
date: 2022-02-20T16:29:00+08:00
draft: false
tags: ["C++"]
slug: "C++ double"
---

目录：

- [表示格式](#表示格式)
- [比较操作](#比较操作)
- [输出精度](#输出精度)
- [参考链接](#参考链接)

## 表示格式

浮点数在计算机中不一定是精确表示的。根据 IEEE（Institute of Electrical and Electronic Engineers ）754 标准，标准浮点数的格式如下：

$$n = (-1)^S*M*2^E$$

* $(-1)^S$ 表示符号位，当 $S=0$，$V$ 为正数；当 $S=1$，$V$ 为负数；
* $M$ 表示有效数字，$1≤M<2$；
* $2^E$ 表示指数位。

例如：十进制的 $5.04$，转化为二进制为 $101.0$，等价于 $1.01 * 2^2$，按照上述格式，可得 $S=0，M=1.01，E=2$。

综上可知，除了能用 $2$ 的指数幂乘以整数表示的浮点数能够被精确的表示外，其余的浮点数都是近似表示的。

## 比较操作

由于浮点数在计算机中不一定是精确表示的，故：**永远不要尝试去比较两个浮点数是否相等。**

为了避免因为表示不精确而导致的错误，通常使用容错阈值来解决这个问题：

```C++
const double EPSILON = 1e-6;
if (abs(d2 - d1) < EPSILON){
    cout << "Equal!" << endl;
}
```

> **注意：**
> **①** C++ 中 `abs()` 已经被重载，因此可以适用于 `int, long, double` 等各种类型，如果是 C 则需要使用 `fabs()` 用于浮点数的绝对值。
>
> **②** `EPSILON` 为精度常量值，如令 `EPSILON = 1e-6`，这个精度值是我们用来判断两个数是否足够接近以至于可以被认为是相等的。动态 `EPSILON` 设置方法，可参见：[c++ 浮点数相等判断背后的陷阱和原理](https://www.jianshu.com/p/b4f0dc31fd4e)。

## 输出精度

`cout` 浮点数默认保留 `6` 位有效数字输出，可通过 `fixed` 和 `setprecision()` 方法修改输出精度，`setprecision()` 用于设置保留的有效数字位数，将其与 `fixed` 共同使用，即可设置小数点后有效数字。使用该方法时，需引入头文件 `<iomanip>` 。

```C++
// 设置保留小数点后两位（设置一次后，对后续所有输出均有效）
cout << fixed << setprecision(2);
cout << 1.2345; // 1.23
cout << 1.2395; // 1.24
cout << 0.1;    // 0.10
cout << -0.128; // -0.13
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [阮一峰 - 浮点数的二进制表示](https://www.ruanyifeng.com/blog/2010/06/ieee_floating-point_representation.html)

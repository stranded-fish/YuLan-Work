---
title: "C++ 字符串"
date: 2022-02-26T11:12:00+08:00
draft: false
tags: ["C++ STL", "C++"]
slug: "C++ string"
---

目录：

- [声明及初始化](#声明及初始化)
- [遍历字符串](#遍历字符串)
- [基本操作](#基本操作)
- [扩展操作](#扩展操作)
  - [字符串逆转](#字符串逆转)
  - [处理 string 对象中的字符](#处理-string-对象中的字符)
  - [字符串分割](#字符串分割)
    - [C `strtok` 函数](#c-strtok-函数)
    - [C++ 流](#c-流)
- [类型转换](#类型转换)
  - [数值 转换为 string](#数值-转换为-string)
  - [string 转换为 数值](#string-转换为-数值)
  - [char 转化为 string](#char-转化为-string)
- [参考链接](#参考链接)

## 声明及初始化

使用 `string` 类时，需引用头文件 `<string>`。

```C++
#include <iostream>
#include <string>

string s1;                     // 初始化为空串
string s2 = "abc";             // 初始化为 "abs"
string s3(s2);                 // 用 s2 初始化 s3
string s4(s3, pos, num);       // 从 s3 中的第 pos 个位置开始，拷贝 num 个字符
string s5(num, ch);            // 拷贝 num 个 ch 字符
string s6(10, ' ');            // 声明字符数量为 10 的空字符串
string s7; s7.resize(10, ' '); // 声明字符数量为 10 的空字符串

/* 构造函数将 char 数组转化为 string */
char chs[] = {'a', 'b', 'c'}; // form 1
const char *chs = "abc";      // form 2
string str(chs);

/* 使用 new 分配内存，注意：返回的是 string * 类型，且使用后需要通过 delete 释放内存 */
string *str = new string("abc");
delete(str);
```

## 遍历字符串

**eg 1. 下标 for 循环遍历**

```C++
for (int i = 0; i < a.size(); ++i) {
    cout << a[i] << " ";
}
```

**eg 2. 基于范围的 for 循环遍历**

```C++
for (char c : a) {
    cout << c << "  ";
}
```

**eg 3. 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto i = a.begin();
for (string::iterator i = a.begin(); i != a.end(); ++i) {
    cout << *i << " ";
}
```

## 基本操作

```C++
// 返回字符串的长度
size_t size();

// 判断字符串是否为空
bool empty();

// 返回字符串最后一个字符的引用（可修改）
char &std::string::back();

// 在字符串尾部插入一个字符 c
void push_back(char c);

// 删除字符串尾部的字符
void pop_back();

// 返回从索引 pos 开始，长度为 len 的子字符串，省略参数 len 返回从 pos 到末尾的所有字符
string substr(size_t pos, siez_t len);

// 查找子串（返回 str2 在 str1 中的下标位置，如果没有找到，则返回 string::npos（int 型））
str1.find(str2);
if (str1.find(str2) == string::npos) cout << "Not find!";

// 连接 字符串 或 num 个字符 ch
s1 = s1 + s2;
s1.append(s2);
s1.append(num, ch);

// string 比较（相等时返回 0；s1 > s2 返回 1，s1 < s2 返回 -1）
if (s1 > s2) ......
s1.compare(s2);

/* insert */

// 在迭代器 p 指向的元素之前创建一个值为 t 的元素，返回指向新添加的元素的迭代器
str.insert(p, t);

// 在迭代器 p 指向的元素之前插入 n 个值为 t 的元素，返回指向新添加的第一个元素的迭代器
str.insert(p, n, t);

// 将迭代器 b 和 e 指定范围内的元素插入到迭代器 p 指向的元素之前，返回指向新添加的第一个元素的迭代器
str.insert(p, b, e);
```

> **Tips:** 在部分需要用到栈 `stack` 处理字符串的地方，通常可以直接使用 `string` 充当栈结构，以简化操作。
>
> `stack<char> stk <-> string str;`
>
> * `stack.push(c) <-> str.push_back(c)`
> * `stack.pop()   <-> str.pop_back()`
> * `stack.top()   <-> str.back()`

## 扩展操作

### 字符串逆转

使用 `reverse()` 函数时，需引入头文件 `<algorithm>`。

```C++
#include <algorithm>
void reverse<_BidIt>(const _BidIt _First, const _BidIt _Last);

string str = "hello world";

// 逆转整个字符串
reverse(str.begin(), str.end()); // dlrow olleh

// 逆转部分字符串
reverse(str.begin(), str.begin() + 5); // olleh world
```

### 处理 string 对象中的字符

* `isalpha(c)` 当 c 是字母时为真
* `isdigit(c)` 当 c 是数字时为真
* `islower(c)` 当 c 是小写字母时为真
* `isupper(c)` 当 c 是大写字母时为真
* `tolower(c)` 如果 c 是大写字母，返回对应的小写字母；否则原样返回 c
* `toupper(c)` 如果 c 是小写字母，返回对应的大写字母；否则原样返回 c

### 字符串分割

C++ 中没有直接的 `split` 函数，字符串分割可借助以下方法实现：

#### C `strtok` 函数

利用 `strtok` 函数实现字符串分割时，需引入 C 头文件 `<string.h>`。该函数原型如下：

```C
char* strtok(char *str, const char *delim);
```

返回值：从 `str` 开头开始的一个个被分割的串。当 `str` 中的字符查找到末尾时，返回 `NULL`，如果查找不到 `delim` 中的字符时，则返回当前 `strtok` 中的字符串的指针。

在首次调用 `strtok` 时，需传递字符串参数 `str`，往后的调用则将参数 `str` 设置成 `NULL`。注意，由于当 `strtok` 发现对应分隔符时，会将该字符改为 `\0` 字符，即传入 `str` 参数会被修改。故在调用该方法前，需要将字符串复制一份再传入。

```C++
vector<string> split(const string &str, const char *delim) {
    vector<string> res;
    char *buffer = new char[str.size() + 1]; // 定义 C 风格字符串，用于修改
    strcpy(buffer, str.c_str());             // 复制字符串
    char *p = strtok(buffer, delim);         // 首次调用，传入复制字符串

    while (p) {
        res.emplace_back(p);
        p = strtok(NULL, delim);             // 再次调用，修改 str 参数为 NULL
    }

    delete buffer;
    return res;
}

int main() {
    string test = "a,b,c,d,e";
    vector<string> res = split(test,  ',');
    for (auto &val : res) {
        cout << val << " "; // 输出：a b c d e
    }
    return 0;
}
```

#### C++ 流

**eg 1.** 利用 `stringstream` + `getline` 方法实现字符串按指定分隔符分割，需引入头文件 `<sstream>`。

```C++
vector<string> split(const string &str, const char &delim) {
    vector<string> res;
    stringstream ss(str); // 读取 str 到字符串流中
    string tmp;

    /* 使用 getline 方法从字符串流中读取，当读取到分隔符时停止
    注意：getline 默认可以读取空格，并且多个分隔符相邻时会读取到空字符串 */
    while (getline(ss, tmp, delim)) {
        if (!tmp.empty()) res.push_back(std::move(tmp));
    }

    return res;
}

int main() {
    string str = "a,b,c,d,e";
    vector<string> res = split(str,  ',');
    for (auto &val : res) {
        cout << val << " "; // 输出：a b c d e
    }
    return 0;
}
```

**eg 2.** 当分隔符仅为默认分隔符，即：`空格、tab、回车换行` 时，可直接使用 `>>` 运算符加以简化。

```C++
vector<string> res;
string str = "  hello world   sperated by spaces \t tab and \n Enter  ";
stringstream ss(str);
string tmp;
while (ss >> tmp) {
    res.push_back(std::move(tmp));
}
```

**eg 3.** 同时存在多种分隔符，利用额外 `char` 字符进行接收并跳过。

```C++
string num = "32.45+-4i";
stringstream ss1(num), ss2(num);

// Tips：该方法还可直接进行类型转换
double val_1; int val_2; char ch;
ss1 >> val_1 >> ch >> val_2;       // val_1 = 32.45, val_2 = -4
ss2 >> val_1 >> ch >> ch >> val_2; // val_1 = 32.45, val_2 =  4
```

**补充：** `stringstream::str` 用于 set/get content

```C++
void str (const string& s); // set
string str() const;         // get 

stringstream ss;
ss.str("Example string");
string s = ss.str();
cout << s; // 输出：Example string
```

## 类型转换

使用以下转化方法时，需引入头文件 `<string>`，并使用 `std` 命名空间。

### 数值 转换为 string

* `to_string`  : numerical value to string

```C++
string to_string (numeric_type val);
```

### string 转换为 数值

* `stoi`   : string to integer
* `stol`   : string to long int
* `stoul`  : string to unsigned integer
* `stoll`  : string to long long
* `stoull` : string to unsigned long long
* `stof`   : string to float
* `stod`   : string to double
* `stold`  : string to long double

```C++
numeric_type sto* (const string& str, [...]);
```

### char 转化为 string

由于字符串拼接操作，要求运算符两侧至少有一个 `string` 类型，故在字符串拼接中，有时候需要将 `char` 类型转化为 `string` 类型。

```C++
// 使用 string 的构造函数
const char c = 'a';
string s(1, c);
```

> **注意：** 不能使用 `to_string` 方法将 `char` 转化为 `string`，因为 `to_string` 只接受 `numerical` 数值形参数，所以传入 `char` 型字符，实际上是先将 `char` 转化为 `int` 型的 `ASCll` 码，然后再转变为 `string`。
>
> 示例：`cout << to_string('a') << endl; // 输出：97`

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [C++ string类（C++字符串）完全攻略](http://c.biancheng.net/view/400.html)
* [C++ 中实现类似split()的字符串分割函数](https://guopengzhen.com/%E7%A8%8B%E5%BA%8F%E7%8C%BF%E7%9A%84%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/8319/)

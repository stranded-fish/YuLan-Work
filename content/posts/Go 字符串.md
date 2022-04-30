---
title: "Go 字符串"
date: 2022-03-10T19:50:00+08:00
draft: false
tags: ["Go"]
slug: "Go string"
---

目录：

- [声明及初始化](#声明及初始化)
- [遍历字符串](#遍历字符串)
- [基本操作](#基本操作)
- [扩展操作](#扩展操作)
  - [查找](#查找)
  - [修剪](#修剪)
  - [字符串分割](#字符串分割)
  - [bytes.Buffer 高效拼接字符串](#bytesbuffer-高效拼接字符串)
  - [字符串修改](#字符串修改)
- [类型转换](#类型转换)
  - [数值 转化为 string](#数值-转化为-string)
  - [string 转换为 数值](#string-转换为-数值)
- [参考链接](#参考链接)

## 声明及初始化

```go
// eg 1. 声明一个字符串，并设置为零值
var str string

// eg 2. 使用字面值声明字符串
var str = "hello"

// eg 3. 短变量声明
str := "world"
```

## 遍历字符串

**eg 1. 下标 for 循环遍历 - 按 ASCII 遍历**

```go
for i := 0; i < len(str); i++ {
	fmt.Println(str[i])
}
```

**eg 2. 基于 range 的 for 循环遍历 - 按 Unicode 遍历**

```go
for _, s := range str {
    fmt.Printf("")
	fmt.Printf("%c", s)
}
```

## 基本操作

```go
// 返回字符串的长度
len(str)

// 判断字符串是否为空
if len(str) == 0

// 返回 [i, j) 范围内的子串
b := a[i:j]

// 连接 字符串
s1 := s2 + s3

// string 比较（可直接使用 == != > < >= <= 运算符）
a := "aaa"
b := "bbb"
if a < b {
	fmt.Println("a < b")
}

/* 字符串替换，用 new 替换 s 中的 old，一共替换 n 个，
如果 n < 0，则不限制替换次数，即全部替换 */
func Replace(s, old, new string, n int) string
```

## 扩展操作

### 查找

相关函数在 `package strings` 中。

**查找子串出现索引：**

```go
// 在 s 中查找 sep 的第一次出现，返回第一次出现的索引
func Index(s, sep string) int

// 在 s 中查找字节 c 的第一次出现，返回第一次出现的索引
func IndexByte(s string, c byte) int

// chars 中任何一个 Unicode 代码点在 s 中首次出现的位置
func IndexAny(s, chars string) int
```

**判断子串是否存在：**

```go
// 若子串 substr 在 s 中，返回 true
func Contains(s, substr string) bool

// 若 chars 中任何一个字符在 s 中，返回 true
func ContainsAny(s, chars string) bool

// 若 Unicode r 在 s 中，返回 true
func ContainsRune(s string, r rune) bool
```

### 修剪

```go
// 将 s 左侧和右侧中匹配 cutset 中的任一字符的字符去掉
func Trim(s string, cutset string) string

// 将 s 左侧的匹配 cutset 中的任一字符的字符去掉
func TrimLeft(s string, cutset string) string

// 将 s 右侧的匹配 cutset 中的任一字符的字符去掉
func TrimRight(s string, cutset string) string

// 如果 s 的前缀为 prefix 则返回去掉前缀后的 string , 否则 s 没有变化。
func TrimPrefix(s, prefix string) string

// 如果 s 的后缀为 suffix 则返回去掉后缀后的 string , 否则 s 没有变化。
func TrimSuffix(s, suffix string) string
```

### 字符串分割

可通过 `package strings` 中的 `split` 系列方法实现字符串分割。

```go
// 按照 sep 分割符分割整个字符串，返回 string 切片
func Split(s, sep string) []string { return genSplit(s, sep, 0, -1) }

// 分割整个字符串，但保留分隔符
func SplitAfter(s, sep string) []string { return genSplit(s, sep, len(sep), -1) }

// 只分割前 n 个字符串
func SplitN(s, sep string, n int) []string { return genSplit(s, sep, 0, n) }

// 只分割前 n 个字符串，并保留分隔符
func SplitAfterN(s, sep string, n int) []string { return genSplit(s, sep, len(sep), n) }
```

### bytes.Buffer 高效拼接字符串

bytes.Buffer 是 Golang 标准库中的缓冲区，具有读写方法和可变大小的字节存储功能。

**1. 声明 Buffer：**

```go
var b bytes.Buffer       				//直接定义一个 Buffer 变量，不用初始化，可以直接使用
b := new(bytes.Buffer)   				//使用 New 返回 Buffer 变量
b := bytes.NewBuffer(s []byte)   		//从一个 []byte 切片，构造一个 Buffer
b := bytes.NewBufferString(s string)	//从一个 string 变量，构造一个 Buffer
```

**2. 往 Buffer 写数据：**

```go
b.Write(d []byte) (n int, err error)   			//将切片 d 写入 Buffer 尾部
b.WriteString(s string) (n int, err error) 		//将字符串 s 写入 Buffer 尾部
b.WriteByte(c byte) error  						//将字符 c 写入 Buffer 尾部
b.WriteRune(r rune) (n int, err error)    		//将一个 rune 类型的数据放到缓冲区的尾部
b.ReadFrom(r io.Reader) (n int64, err error)	//从实现了 io.Reader 接口的可读取对象写入 Buffer 尾部
```

**3. 从 Buffer 读数据：**

```go
//读取 n 个字节数据并返回，如果 buffer 不足 n 字节，则读取全部
b.Next(n int) []byte

//一次读取 len(p) 个 byte 到 p 中，每次读取新的内容将覆盖p中原来的内容。成功返回实际读取的字节数，off 向后偏移 n，buffer 没有数据返回错误 io.EOF
b.Read(p []byte) (n int, err error)

//读取第一个byte并返回，off 向后偏移 n
b.ReadByte() (byte, error)

//读取第一个 UTF8 编码的字符并返回该字符和该字符的字节数，b的第1个rune被拿掉。如果buffer为空，返回错误 io.EOF，如果不是UTF8编码的字符，则消费一个字节，返回 (U+FFFD,1,nil)
b.ReadRune() (r rune, size int, err error)

//读取缓冲区第一个分隔符前面的内容以及分隔符并返回，缓冲区会清空读取的内容。如果没有发现分隔符，则返回读取的内容并返回错误io.EOF
b.ReadBytes(delimiter byte) (line []byte, err error)

//读取缓冲区第一个分隔符前面的内容以及分隔符并作为字符串返回，缓冲区会清空读取的内容。如果没有发现分隔符，则返回读取的内容并返回错误 io.EOF
b.ReadString(delimiter byte) (line string, err error)

//将 Buffer 中的内容输出到实现了 io.Writer 接口的可写入对象中，成功返回写入的字节数，失败返回错误
b.WriteTo(w io.Writer) (n int64, err error)
```

**4. 其他操作：**

```go
b.Bytes() []byte		//返回字节切片
b.Cap() int				//返回 buffer 内部字节切片的容量
b.Grow(n int)			//为 buffer 内部字节切片的容量增加 n 字节
b.Len() int				//返回缓冲区数据长度，等于 len(b.Bytes())
b.Reset() 				//清空数据
b.String() string		//字符串化
b.Truncate(n int)		//丢弃缓冲区中除前n个未读字节以外的所有字节。如果 n 为负数或大于缓冲区长度，则引发 panic
b.UnreadByte() error	//将最后一次读取操作中被成功读取的字节设为未被读取的状态，即将已读取的偏移 off 减 1
b.UnreadRune() error	//将最后一次 ReadRune() 读取操作返回的 UTF8 字符 rune设为未被读取的状态，即将已读取的偏移 off 减去 字符 rune 的字节数
```

### 字符串修改

* Go 语言的字符串是不可变的。
* 修改字符串时，可以将字符串转换为 `[]byte` 进行修改。
* `[]byte` 和 `string` 可以通过强制类型转换互转。

```go
angel := "Heros never die"

angleBytes := []byte(angel)

for i := 5; i <= 10; i++ {
    angleBytes[i] = ' '
}

fmt.Println(string(angleBytes)) // Heros       die
```

## 类型转换

### 数值 转化为 string

```go
a := 123
fmt.strconv.Itoa(a)
```

### string 转换为 数值

```go
func Atoi(s string) (int, error)
```

由于 `string` 可能无法转换为 `int`，所以这个函数有两个返回值：第一个返回值是转换成 `int` 的值，第二个返回值判断是否转换成功。

```go
// Atoi()转换失败
i,err := strconv.Atoi("a")
if err != nil {
    println("converted failed")
}
```

## 参考链接

* [strings — 字符串操作](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter02/02.1.html)
* [Go 语言修改字符串](http://c.biancheng.net/view/39.html)

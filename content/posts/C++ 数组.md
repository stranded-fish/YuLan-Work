---
title: "C++ 数组"
date: 2022-03-10T11:28:00+08:00
draft: false
tags: ["C++"]
slug: "C++ array"
---

目录：

- [静态数组](#静态数组)
  - [声明及初始化](#声明及初始化)
  - [遍历数组](#遍历数组)
    - [一维数组](#一维数组)
    - [二维数组](#二维数组)
- [动态数组 - vector](#动态数组---vector)
  - [声明及初始化](#声明及初始化-1)
  - [基本操作](#基本操作)
    - [基础操作](#基础操作)
    - [元素访问](#元素访问)
    - [数组逆转](#数组逆转)
    - [数组拷贝](#数组拷贝)
    - [插入元素](#插入元素)
    - [调整容量](#调整容量)
  - [遍历数组](#遍历数组-1)
    - [一维数组](#一维数组-1)
    - [二维数组](#二维数组-1)
- [数组排序](#数组排序)
- [参考链接](#参考链接)

## 静态数组

### 声明及初始化

声明数组：

```C++
type arrayName[arraySize];
```

在 C++ 中，可以逐个初始化数组元素，也可以使用一个初始化语句，如下所示：

```C++
// 大括号 { } 之间的值的数目不能大于在数组声明时在方括号 [ ] 中指定的元素数目。
double balance[5] = {1000.0, 2.0, 3.4, 7.0, 50.0}; 

// 如果省略掉了数组的大小，数组的大小则为初始化时元素的个数。
double balance[] = {1000.0, 2.0, 3.4, 7.0, 50.0}; 

// 数组值初始化为全 0
int array[10] = {};
```

### 遍历数组

#### 一维数组

**eg 1.1 下标 for 循环遍历**

```C++
int a[5] = {0, 3, 4, 6, 2};
int size = sizeof(a) / sizeof(*a);
for(int i = 0; i < size; ++i)
    cout << a[i] << " ";
```

**eg 1.2 基于范围的 for 循环遍历**

```C++
for(int &item : a)
    cout << item << " ";
```

**eg 1.3 基于指针的 for 循环遍历**

```C++
// 此处 auto p 等价于 int *p
for (auto p = a; p != a + size; ++p) {
    cout << *p;
}
```

#### 二维数组

**eg 2.1 下标 for 循环遍历**

```C++
int a[3][4] = {1,2,3,4,5,6,7,8};
for (int i = 0; i < 3; ++i) {
    for (int j = 0; j < 4; ++j) {
        cout << a[i][j] << endl;
    }
}
```

**eg 2.2 基于范围的 for 循环遍历**

```C++
for (auto &p : a) {
    for (auto &q : p) {
        cout << q << endl;
    }
}
```

> **注意：** 上述示例 `auto p` 被转化为指针（其类型为 `int(*)[4]`），而在内循环中指针是不可以范围 for 循环遍历的，所以会报错。故使用基于范围的 for 循环遍历时，除最里层循环以外的所有循环都必须用引用，才能正常遍历。并且如果还想对元素赋值，最里层循环也需使用引用。

**eg 2.3 基于指针的遍历**

```C++
int ia[3][4] = {1,2,3,4,5,6,7,8};

// 此处 auto p 等价于 int (*p)[4]
for (auto p = ia; p != ia + 3; ++p) {
    for (auto q = *p; q != *p + 4; ++q) {
        cout << *q << endl;
    }
}

for (auto p = begin(ia); p != end(ia); ++p) {
    for (auto q = begin(*p); q != end(*p); ++q) {
        cout << *q << endl;
    }
}
```

> **注意：** 多维数组，除最外层循环外，其余循环均需要对指针进行解引用。

## 动态数组 - vector

### 声明及初始化

使用 `vector` 容器时，需引入头文件 `<vector>`。

```C++
#include <vector>

/* eg 1 initialize */

// 初始化一个 int 类型的空数组 nums
vector<int> nums;

// 初始化一个大小为 n 的数组 nums，数组中的值默认都为 0
vector<int> nums(n);

// 初始化一个元素为 1、3、5 的数组 nums
vector<int> nums{1, 3, 5};

// 初始化一个大小为 n 的数组 nums，其值全都为 2
vector<int> nums(n, 2);

// 初始化一个二维 int 数组 dp
vector<vector<int>> dp;

// 初始化一个大小为 m*n 的布尔数组 dp，其值均为 true
vector<vector<bool>> dp(m, vector<bool>(n, true));

/* eg 2 make a copy */
vector<int> v(v1.begin(), v1.end());
vector<int> v(v2);

/* eg 3 cast an array to a vector */
int a[5] = {0, 1, 2, 3, 4};
vector<int> v(a, *(&a + 1));
```

### 基本操作

#### 基础操作

```C++
// 返回数组是否为空
bool empty();

// 返回数组的元素个数
size_type size();

// 返回数组的容量大小
size_type capacity();

// 在数组尾部插入一个元素 val
void push_back(const value_type &val);
void emplace_back(Args&&... args);

// 删除数组尾部的那个元素
void pop_back();

// 将数组 size 设置为 0，但 capacity 不变（注意：不能用于清空数组）
void clear(); 

// 交换两个 vector
newArray.swap(oldArray);
```

#### 元素访问

```C++
/* 1. 返回数组 首个/末尾 元素的引用（type &）*/
      reference front();
const_reference front() const;
      reference back();
const_reference back() const;

// 示例
int &val = v.back(); // 定义末尾元素的引用
v.back()++;          // 末尾元素 + 1
v.back() -= 1;       // 末尾元素 - 1

/* 2. 返回指向数组 首个 / past-the-end 元素的 iterator 指针
注意：past-the-end 不指向任何元素，故不能被解引用 */
      iterator begin() noexcept;
const_iterator begin() const noexcept;
      iterator end() noexcept;
const_iterator end() const noexcept;

/* 类似 vector 之类的序列式容器能够实现随机访问（Random Access），
故其迭代器及其指针能够使用 + - += -= ++ -- 等运算符，示例如下： */
auto it = v.begin();        // 声明指向数组首个元素的 iterator 指针
cout << *(--v.end());       // 输出数组最后一个元素
cout << *(v.end() - 2);     // 输出数组倒数第 2 个元素
cout << (*(v.end() - 2))++; // 输出数组倒数第 2 个元素，并使其 + 1

/* 3. 反向迭代器，rbegin() 指向最后一个元素，rend() 指向第一个元素之前的理论元素，不可解引用 */
      reverse_iterator rbegin() noexcept;
const_reverse_iterator rbegin() const noexcept;
      reverse_iterator rend() noexcept;
const_reverse_iterator rend() const noexcept;

// 示例
cout << *v.rbegin();      // 输出最后一个元素
cout << *(v.rbegin() + 1) // 输出倒数第 2 个元素
```

#### 数组逆转

使用 `reverse()` 函数时，需引入头文件 `<algorithm>`。

```C++
#include <algorithm>
void reverse<_BidIt>(const _BidIt _First, const _BidIt _Last);

vector<int> test{1, 2, 3};

// 逆转整个数组
reverse(test.begin(), test.end()); // test{3, 2, 1}

// 逆转部分数组
reverse(test.begin(), test.begin() + 2); // test{2, 1, 3}
```

#### 数组拷贝

```C++
vector<int> oldArray{1, 2, 3};
vector<int> newArray;

// eg 1 - 赋值运算符 深拷贝
newArray = oldArray;

// eg 2 - assign() 清空并深拷贝
newArray.assign(oldArray.begin(), oldArray.end());
```

#### 插入元素

```C++
vector<int> array{1, 2};

// 插入单个元素
iterator insert(const_iterator position, const value_type& val);
array.insert(array.begin() + 1, 3); // {1,3,2}

// 填充 n 个元素
iterator insert(const_iterator position, size_type n, const value_type& val);
array.insert(array.end(), 2, 5); // {1,2,5,5}

// 插入 [first,last) 范围内的所有元素
iterator insert(const_iterator position, InputIterator first, InputIterator last);
vector<int> test{7,8,9};
array.insert(array.end(), test.begin(), test.end()); // {1,2,7,8,9}
```

#### 调整容量

* `reserver` 调整容器预分配存储区大小，即 capacity 值，但并不进行初始化：

```C++
void reserve(const size_t _Newcapacity)
```

> **注意：** reserve 的参数 n 是推荐预分配内存的大小，实际分配的可能等于或大于这个值，即 n 大于 capacity 的值，就会 reallocate 内存，capacity 的值会大于或者等于 n 。这样，当容器调用 push_back 函数使得 size 超过原来的默认分配的 capacity 值时，就能避免内存重分配开销。
>
> 同时，reserve 函数分配出来的内存空间，只是表示 vector 可以利用这部分内存，但由于该部分内存还未进行初始化，故不能有效地访问这些内存空间，访问的时候会出现越界现象，导致程序崩溃。

* `resize` 调整容器大小，并且创建对象：

```C++
// 调整容器大小，使其包含 n 个元素，如果有新增元素，则将新元素进行默认初始化
void resize (size_type n); 

// 调整容器大小，使其包含 n 个元素，如果有新增元素，则将新元素初始化为 val 的副本
void resize (size_type n, const value_type& val);

/* 应用：调整多维数组容器大小 */
vector<int> tmp1(k + 1, -1);
vector<vector<int>> tmp2(n, tmp1);
vector<vector<vector<int>>> arr;
arr.resize(n, tmp2);

// 等价形式
vector<vector<vector<int>>> arr(n, vector<vector<int>>(n, vector<int>(k + 1, -1)));
```

* 如果 n 小于当前容器的大小，则将内容减少到其前 n 个元素，并删除超出范围的元素（并销毁它们）。
* 如果 n 大于当前容器的大小，则通过在末尾插入所需数量的元素来扩展内容，以达到 n 的大小。如果指定了 val，则将新元素初始化为 val 的副本，否则将对它们进行值初始化。
* 如果 n 也大于当前容器容量，将自动重新分配已分配的存储空间。

> **注意：** 该函数通过插入或擦除容器中的元素来更改容器的实际内容。

### 遍历数组

#### 一维数组

**eg 1.1 下标 for 循环遍历**

```C++
for (int i = 0; i < v1.size(); ++i) {
    cout << v1[i] << endl;
}
```

**eg 1.2 基于范围的 for 循环遍历**

```C++
for (int &item : v2) {
    cout << item << endl;
}
```

**eg 1.3 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto i = vec.begin()
for (vector<int>::iterator i = vec.begin(); i != vec.end(); ++i) {
    cout << *i << "  ";
}
```

迭代器按照定义方式分为以下四种：

* 正向迭代器

  ```C++
  容器类名::iterator  迭代器名;
  ```

* 常量正向迭代器

  ```C++
  容器类名::const_iterator  迭代器名;
  ```

* 反向迭代器

  ```C++
  容器类名::reverse_iterator  迭代器名;
  ```

* 常量反向迭代器

  ```C++
  容器类名::const_reverse_iterator  迭代器名;
  ```

#### 二维数组

**eg 2.1 下标 for 循环遍历**

```C++
for (int i = 0; i < vec.size(); ++i) {
    for(int j = 0; j < vec[0].size(); ++j) {
        cout << vec[i][j] << " ";
    }
}
```

**eg 2.2 基于范围的 for 循环遍历**

```C++
for (auto &p : vec) {
    for (auto &q : p) {
        cout << q << "  ";
    }
}
```

**eg 2.3 基于迭代器遍历**

```C++
for (vector<vector<int>>::iterator i = vec.begin(); i != vec.end(); ++i) {
    for (vector<int>::iterator j = (*i).begin(); j != (*i).end(); ++j) {
        cout << *j << " ";
    }
    cout << endl;
}

// 可使用 auto 关键字简化
for (auto p = vec.begin(); p != vec.end(); ++p) {
    for (auto q = (*p).begin(); q != (*p).end(); ++q) {
        cout << *q << "  ";
    }
}
```

## 数组排序

`sort()` 方法能够对容器或普通数组中 `[first, last)` 范围内的元素进行排序，默认进行升序排序。

该方法包含在 `<algorithm>` 头文件，并定义在 `std` 命名空间：

```C++
#include <algorithm>
using namespace std;
```

`sort()` 函数原型如下：

```C++
default (1)	
  void sort (RandomAccessIterator first, RandomAccessIterator last);
custom  (2)	
  void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```

* `first` - 待排序数组的首地址
* `last`  - 待排序数组的尾地址
* `comp`  - 比较方法，传递可调用对象（包含：函数、函数指针、重载了函数调用运算符的类对象、lambda 表达式），缺省为升序

其中 `custom (2)` 自定义排序，可使用 C++ 内置排序方法：

* `less<data-type>()`：升序
* `greater<data-type>()`：降序

也可以使用自定义排序方法：

**eg 1. 函数**

```C++
bool comp(const int &a, const int &b) { return a < b; }
sort(nums.begin(), nums.end(), comp);
```

**eg 2. 函数指针**

```C++
bool comp(const int &a, const int &b) { return a < b; }
auto pf = &comp;
sort(nums.begin(), nums.end(), pf);
```

**eg 3. 重载函数调用运算符 ()**

```C++
struct compare {
    bool operator () (int a, int b) {
        return a < b;
    }
};
compare cmp; // 注意：需传递类对象
sort(arr.begin(), arr.end(), cmp);
```

**eg 4. lambda 表达式**

```C++
/* eg 4.1 */
sort(nums.begin(), nums.end(),
    [](const int &a, const int &b) { return a < b; });

/* eg 4.2 */
auto comp_2 = [](const int &a, const int &b) { return a < b; };
sort(nums.begin(), nums.end(), comp_2);
```

其中自定义排序方法，比较入参 `a` 和 `b`：

* 升序：定义当 `a < b` 时，返回 `true`；
* 降序：定义当 `a > b` 时，返回 `true`；

> **注意：**
> **①** 出于效率考虑，为了避免多余拷贝，应采用引用传递。同时该函数不得修改其任何参数，故需要用 `const` 限定符加以限定。
>
> **②** 当在类中使用自定义 `comp` 方法进行排序时，不能直接调用类中定义的非静态 `comp` 方法（如：LeetCode 核心代码模式）。
>
> 因为非静态的成员函数必须要被绑定到一个类的对象或者指针上，才能得到被调用对象的 `this` 指针，为了区分是谁调用了成员函数，就必须要有 `this` 指针，`this` 指针是隐式添加到函数参数列表中的，而这就使得与所需 `comp` 方法入参不匹配。
>
> 解决方法：
>
> 1. 通过 `static` 修饰自定义排序方法 `comp`。由于类的静态成员不依赖于具体对象，所有实例化对象都共享同一个静态成员，故也就没有 `this` 指针的概念。
> 2. 将该方法定义到类外部，即定义为全局普通函数。
>
>
> **③** Effective STL no.21：总是让比较函数在等值情况下返回 false。
>
> sort 函数为了最大程度的提高效率，结合了快排、堆排和插入排序等多种排序方法，分为 std::__introsort_loop 和 std::__final_insertion_sort 两个阶段。
>
> 第一阶段使用 “快排+堆排” 的方法，第二阶段使用 “插入排序”，当元素个数小于等于 _S_threshold（enum {_S_threshold = 16 }）时，执行普通的插入排序，当大于 _S_threshold 时，执行两次的 “插入” 排序操作，首先使用普通的插入排序来排 [first,_S_threshold) 这个范围的元素，然后使用无保护的插入排序，完成 [_S_threshold, last) 这个范围的排序。
>
> 而最后使用的无保护的插入排序，为了提升效率，省略了越界的检查。如果在比较元素相等的情况下返回 true 会导致遍历无法停止，最终造成访问越界，程序崩溃。
>
> 解决方法，遵循自定义 [Compare 实现要求](https://en.cppreference.com/w/cpp/named_req/Compare)：
>
> 1. 对于任意元素 a，需满足 comp(a, a) == true；
> 2. 对于任意两个元素 a 和 b，若 comp(a, b) == true 则要满足 comp(b, a) == false；
> 3. 对于任意三个元素 a、b 和 c，若 comp(a, b) == true 且 comp(b, c) == true 则需要满足 comp(a, c) == true。
>
> 综上：如果定义两个元素相等时返回 true，将无法满足条件 2，故总是需要让比较函数在等值情况下返回 false。

**eg 1. 静态数组排序**

```C++
// arrays - 数组名（首地址），size - 数组大小
sort(arrays, arrays + size);                 // 默认升序排序
sort(arrays, arrays + size, less<int>());    // 升序排序
sort(arrays, arrays + size, greater<int>()); // 降序排序
```

**eg 2. 动态数组排序**

```C++
sort(v.begin(), v.end());                 // 默认升序排序
sort(v.begin(), v.end(), less<int>());    // 升序排序
sort(v.begin(), v.end(), greater<int>()); // 降序排序
```

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)
* [C++ 数组](https://www.runoob.com/cplusplus/cpp-arrays.html)
* [C++ 数组和vector的基本操作](https://www.cnblogs.com/HL-space/p/10546585.html)
* [C++ 中使用std::sort自定义排序规则时要注意的崩溃问题](https://blog.csdn.net/albertsh/article/details/119523587)
* [C++ 多维数组的遍历以及初始化](https://blog.csdn.net/anlian523/article/details/90549379)
* [C++ 迭代器（STL迭代器）iterator详解](http://c.biancheng.net/view/338.html)
* [C++ lambda表达式\prority_queue\decltype](https://www.shuzhiduo.com/A/RnJWBblwzq/)

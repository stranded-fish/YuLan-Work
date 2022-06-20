---
title: "C++ 映射"
date: 2022-03-09T19:39:00+08:00
draft: false
tags: ["C++ STL", "C++"]
slug: "C++ map"
---

目录：

- [map](#map)
  - [声明及初始化](#声明及初始化)
  - [遍历哈希表](#遍历哈希表)
  - [基本操作](#基本操作)
    - [基础操作](#基础操作)
    - [元素访问](#元素访问)
- [unordered_map](#unordered_map)
  - [声明及初始化](#声明及初始化-1)
  - [遍历无序哈希表](#遍历无序哈希表)
  - [基本操作](#基本操作-1)
- [参考链接](#参考链接)

## map

`map` 是关联容器，它按照特定顺序存储由键值和映射值组合形成的元素。在映射中，键值通常用于对元素进行排序和唯一标识，而映射值存储与该键关联的内容。`key` 和 `value` 的类型可能不同，并在成员类型 `value_type` 中组合成 `pair` 类型：

```C++
typedef pair<const Key, T> value_type;
```

在内部，`map` 中的元素始终按照其内部比较对象（Compare 类型）指示的特定严格弱排序标准按其 **键排序**。所以只要求其保存的 键 - key 元素类型可以比较即可，不需要实现哈希函数，**故 `map` 相较于 `unordered_map` 的一大优点就是：可以使用 `pair`、`vector` 或是其他可以进行比较，但没有实现哈希函数的类型作为键。**

### 声明及初始化

使用哈希表 `map` 时，需引入头文件 `<map>`。

```C++
#include <map>

// 初始化一个空的 map
map<int, int> mappings;

// 通过初始化列表中的键值对，对其进行初始化
map<char, char> pairs = {
    {')', '('},
    {']', '['},
    {'}', '{'}
};

// 范围构造函数 - 插入 [first,last) 范围内的元素
vector<pair<int, int>> arr = {{1, 2}, {3, 4}};
map<int, int> mappings(arr.begin(), arr.end());
```

### 遍历哈希表

**eg 1. 基于范围的 for 循环遍历**

```C++
// pair<const type, type> item
for (auto item : mappings) {
    cout << item.first << " " << item.second << endl;
}
```

**eg 2. 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto iter = mappings.begin()
for (map<int, int>::iterator iter = mappings.begin(); iter != mappings.end(); ++iter) {
    cout << iter->first << " " << iter->second << endl;
}
```

### 基本操作

#### 基础操作

```C++
// 返回哈希表中的键值对个数
size_type size();

// 返回哈希表是否为空，如果不为空返回 1，否则返回 0
bool empty();

// 如果 key 存在返回 1，否则返回 0
size_type count(const key_type& key);

// 通过 key 或 迭代器指针删除哈希表中的键值对，如果删除成功则返回 1，否则返回 0
size_type erase (const key_type& key);
iterator  erase (const_iterator position);
iterator  erase (const_iterator first, const_iterator last);

// 将哈希表 size 设置为 0，但 capacity 不变（注意：不能用于清空哈希表）
void clear();
```

#### 元素访问

```C++
/* 1. 返回指向 map 首个 / past-the-end 元素的 iterator 指针
注意：past-the-end 不指向任何元素，故不能被解引用 */
      iterator begin() noexcept;
const_iterator begin() const noexcept;
      iterator end() noexcept;
const_iterator end() const noexcept;

// 示例
cout << (--mappings.end())->first << " "  // 输出 map 最后一个元素
     << (--mappings.end())->second;
auto it = mappings.end();
--it; --it;
cout << it->first << " " << it->second;   // 输出 map 倒数第 2 个元素

/* 2. 反向迭代器，rbegin() 指向最后一个元素，rend() 指向第一个元素之前的理论元素，不可解引用 */
      reverse_iterator rbegin() noexcept;
const_reverse_iterator rbegin() const noexcept;
      reverse_iterator rend() noexcept;
const_reverse_iterator rend() const noexcept;

// 示例
cout << mappings.rbegin()->first << " "     // 输出 map 最后一个元素
     << mappings.rbegin()->second;  
cout << (++mappings.rbegin())->first << " " // 输出 map 倒数第 2 个元素
<< (++mappings.rbegin())->second;
```

> **注意：**
> **①** `map` 没有类似于 `vector` 的 `front()`、`back()` 方法，只能通过迭代器访问首尾元素。
>
> **②** `map` 为关联式容器，不能像 `vector` 之类的序列式容器那样实现随机访问（Random Access），
故其迭代器不能使用 `+=、-= 、- 、+` 运算符，只能使用 `++、--` 运算符。
>
> **③** `map` 迭代器指针的解引用返回值 `pair`，`first - key` 为常量不能修改，`second - value` 可修改。
>
> **④** `begin()`，`end()` 方法返回的迭代器指向类型为 `pair<const type, type>` 由于 `map` 内部以 `key` 为关键值排序，故可以通过 `begin()` 关键字修改头元素，并 `mappings.erase(mappings.begin())` 实现删除 `map` 头元素的功能，以实现等价于 优先队列 + 哈希计数 的效果。

## unordered_map

`unordered_map` 是关联容器，它存储由键值和映射值组合形成的元素，并允许基于键快速检索各个元素。在 `unordered_map` 中，键值一般用于唯一标识元素，而映射的值是与该键关联的内容的对象。键和映射值的类型可能不同。

在内部，`unordered_map` 中的元素并没有根据它们的键或映射值按任何特定顺序排序，而是根据它们的哈希值组织成桶。**而由于 `pair`、`vector` 等类型没有实现默认哈希函数，故不能作为 `unordered_map` 键 - key 进行保存。**

### 声明及初始化

使用无序哈希表 `unordered_map` 时，需引入头文件 `<unordered_map>`。

```C++
#include <unordered_map>

// 初始化一个空的 unordered_map
unordered_map<int, int> mapping;

// 通过初始化列表中的键值对，对其进行初始化
unordered_map<char, char> pairs = {
    {')', '('},
    {']', '['},
    {'}', '{'}
};

// 范围构造函数 - 插入 [first,last) 范围内的元素
vector<pair<int, int>> arr = {{1, 2}, {3, 4}};
unordered_map<int, int> mappings(arr.begin(), arr.end());
```

### 遍历无序哈希表

**eg 1. 基于范围的 for 循环遍历**

```C++
// pair<const type, type> item
for (auto item : mapping) {
    cout << item.first << " " << item.second << endl;
}
```

**eg 2. 基于迭代器遍历**

```C++
// 可用 auto 关键字简化 auto iter = mapping.begin()
for (unordered_map<int, string>::iterator iter = mapping.begin(); iter != mapping.end(); ++iter) {
    cout << iter->first << " " << iter->second << endl;
}
```

### 基本操作

```C++
// 返回哈希表中的键值对个数
size_type size();

// 返回哈希表是否为空，如果不为空返回 1，否则返回 0
bool empty();

// 如果 key 存在返回 1，否则返回 0
size_type count(const key_type& key);

// 如果 key 存在，则返回一个指向该元素的迭代器，否则返回指向 unordered_map::end
      iterator find ( const key_type& k );
const_iterator find ( const key_type& k ) const;

// 通过 key 删除哈希表中的键值对，如果删除成功则返回 1，否则返回 0
size_type erase(const key_type& key);

// 将哈希表 size 设置为 0，但 capacity 不变（注意：不能用于清空哈希表）
void clear();
```

> **注意：** 用 `[]` 访问其中的键 `key` 时，如果 `key` 不存在，则会自动创建 `key`，对应的值为该值类型的默认值。

## 参考链接

* [C++ Reference](http://www.cplusplus.com/reference/)

---
title: C++11新特性 - tuple
date: 2020-03-14 17:13:27
tags:
- C++
- C++11
---

C++11 引入了 `tuple`，作为 `pair` 的一个扩展，用于简单地把几个值打包到一个结构中

<!-- More -->

在 C++03 中，我们经常这样使用 `pair`

```C++
pair<int, char> p(1, 'a');
cout << p.first << ", " << p.second << endl; // 1, a
```

但是，假如我们要打包 3 个值的时候呢？有时我们会编写 `pair<T1, pair<T2, T3>>` 这样的代码

C++11，我们可以用 `tuple` 来实现这种功能

```C++
tuple<int, double, char> t(1, 2.0, 'a');
auto t2 = make_tuple(1, 2.0, 'b'); // 类似于 make_pair

// tuple 的比较也是按字典序的
if (t < t2) cout << "t < t2\n"; // t < t2

// 用 get<index>(tuple) 来获取 tuple 中第 index 个元素
cout << get<0>(t) << ", " << get<2>(t) << endl; // 1, a

get<0>(t) = 2; // get 返回引用，可以修改 tuple 中的元素
cout << get<0>(t) << endl; // 2
```

需要注意的是，`get` 函数中的 index 必须是一个合法的编译期常数

```C++
constexpr int i = 1;
get<i>(t); // 没问题

get<3>(t); // 报错，3 越界了

int j = 2;
get<j>(t); // 报错，j 不是常数
```

在 Python 中，我们可以编写 `_, x, y = (1, 2.0, 'a')` 这样的代码，把 `tuple` 中的元素分别赋值给多个元素

C++11 中，要达成类似的目的，需要使用 `tie` 函数

```C++
tie(ignore, x, y) = t; // ignore 代表该位置上的元素不需要赋值
cout << x << ", " << y << endl; // 2, a
```

使用 `tuple_cat` 函数，可以连接两个 `tuple`

```C++
auto t3 = tuple_cat(t, t2);
cout << get<5>(t3) << endl; // b
```

同时，如果想要获取一个 tuple 的长度，类似 Python 中的 `len(tuple)`，则需要

```C++
cout << std::tuple_size<decltype(t3)>::value << endl;  // 6
```

补充一下，假如可以使用 C++17 标准，将 tuple 赋值给多个变量时，我们甚至连 `tie` 函数都不太需要了，直接使用结构化绑定声明特性，只需一行代码
```C++
auto& [ignore, a, b] = t; // 需要 C++17
cout << a << ", " << b << endl; // 2, a
```
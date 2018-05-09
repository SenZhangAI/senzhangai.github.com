---
layout: post
title: "C++11 FAQ by Bjarne Stroustrup"
description: "Bjarne Stroustrup网站上的C++11 FAQ 阅读笔记"
keywords: C++, C++11
category: C++
tags: [c++, c++11]
---

## 前言
此文由C++之父 Bjarne Stroustrup 撰写，为学习C++11的较好的资料

网址： <http://www.stroustrup.com/C++11FAQ.html>

其他值得学习的参考资料包括：
[C++11-wiki中文](https://zh.wikipedia.org/zh-cn/C%2B%2B11)
[C++14-wiki中文](https://zh.wikipedia.org/wiki/C%2B%2B14)
wiki-中文是对英文版本的翻译和补充，适合阅读

## Lists of new feature

### __cpluscplus
宏 `__cpluscplus` 的值将被修改：

```cpp
// C++98: 第一个正式的C++标准
// C++03: 在C++98基础上的小幅度修订
define __cpluscplus 199711L
// C++11标准: 之前也被称为C++0x, 一次全面的大进化
define __cpluscplus 201103L
// C++14标准: 旨在作为C++11的一个小扩展
// 对C++11兼容，因此也没有提供特定的宏

// usage:
// 因C++11变动较大，主要考虑其可移植性，C++14、17标准以后补上
# if __cpluscplus < 201103L
    //code in C++03...
# else
    //code in C++11...
# endif
```

### auto --deduction of a type from an initializer
`auto x = 7;` x将会是int类型，因为7是int类型。
`auto x =expression;` x的类型将由表达式类型推导获得。

auto 通常用于书写麻烦或者类型很难确切知道的场合。

以下内容摘入自链接中**蓝色的**的答案：
@see [知乎-如何评价auto关键字](http://www.zhihu.com/question/35517805)

使用场合例如:
1. 容器的iter:

```cpp
vector<int> v;
//vector<int>::iterator iter = v.begin();
auto iter = v.begin();
```

2. 配合lambda，对于lambda来说，它是一个callable object,每一个类型都是独一无二的，
该类型只有编译器知道，因此可以：

```cpp
auto closure = [](const int&, const int&) {}
```

在C++14中，甚至允许**参数类型**为auto类型，如：

```cpp
auto closure = [](auto x, auto y) {return x * y;}
```
3. auto配合decltype, 例子如下：

对于tamplate返回值的情况：

```cpp
template <class T, class U>
/* what's the type? */ mul(T x, U y) {
    return x * y;
}
```

对于mul返回值，如何确定其类型呢？
按照C++11中decltype的做法，可以利用[decltype(expression)](http://baike.baidu.com/link?url=1G1YWpWe6tDspZZrbPznXRebPTQkdjE4LBnRm8z-4WfsWuyifYqxe_8LNn7cbYxKLZkZZdEQ2Bdu9mirHGiVMK)操作符取得类型，

补充：typeof与declype：
typeof是在C++11以前，诸如GCC等编译器自身实现的特性，不具备可移植性。
decltype与typeof类似，都是为了泛型编程中获知类型，因为typeof名字已被抢了，所以叫decltype，
否则，就和sizeof，alignof形式相同了。
declype出来后typeof可以说被淘汰了，decltype对应类型判断（例如 reference判断）更好，更适合配合lambda表达式。


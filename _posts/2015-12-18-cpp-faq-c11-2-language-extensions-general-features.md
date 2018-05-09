---
layout: post
title: "C++ FAQ C11 2 General Feature"
description: "C++ FAQ网站上的学习笔记"
keywords: C++, C++11
category: C++
tags: [C++, C++11]
---

## 前言
来自C++ FAQ官网的内容，同时融入C++11-wiki上的内容，
相关链接：

[isocpp FAQ C11 General Feature](https://isocpp.org/wiki/faq/cpp11-language)
[C++11 wiki](https://zh.wikipedia.org/zh-cn/C%2B%2B11)

## auto

## decltype

## Range for statement

## Initializer lists

## Uniform initialization

## Rvalue references and move semantics
概述：右值引用目的是提高性能，减少不必要的copy开销。

场景分析：比如函数内一临时变量作返回值时，通常会拷贝，如何减少开销呢？
如果是临时变量那么生命期结束就销毁，所以需要拷贝。
如果用static local呢？那么多线程不安全。

机制：右值的移动语义是将指向对象的指针的值给新对象，然后原指针置null来保证安全。

限制条件：该对象需要有move构造函数还行；为了安全具名对象不会被认定为右值，
为了获得右值需要用`std::move<T>`，例如：

```cpp
bool is_r_value(int&& ) { return true; }
bool is_r_value(const int&) { return false; }

void test(int&& i) {
    is_r_value(i);     //i为剧名对象，即使被宣告成右值也不是右值
    is_r_value( std::move<int&>(i) );   //使用std::move取出右值
}
```

### 左值与右值
历史：左值与右值的叫法来自于C++的前身CPL，所以沿用至今。等号左边左值、等号右边右值。
在C++98标准中，non-const ref绑定左值；const ref绑定左值或者右值。但都不能绑定 non-const右值

C++98中ref不绑定non-const右值的原因：之所以这样是为了安全，防止修改了临时变量的值，举例：

```cpp
void incr(int& a) { ++a; }
int i = 0;
incr(i);
incr(0);    //error. 0 is not an lvalue
```

## Lambdas

## noexcept to prevent exception propagation

## constexpr
简述-用途：简单来说，过去如果一个函数返回常值，编译器是无法将其优化的，视为运行期函数。
C++11引入constexpr修饰后，可以将一些不变化的量显式修饰为常值。这样便于编译器优化判断。
这样的常值可用于数组长度。

示例：

```cpp
int GetFive() { return 5; }  //error
constexpr int GetSix() { return 6; } //ok
int some_arr[GetSix() + 1];  //Ok
```



## nullptr - a null pointer literal

## copying and rethrowing exceptions

## inline namespaces

## User-defined literals



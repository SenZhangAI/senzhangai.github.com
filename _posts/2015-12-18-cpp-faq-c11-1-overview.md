---
layout: post
title: "C++ FAQ C11 1 Overview"
description: "C++ FAQ网站上的学习笔记"
keywords: C++, C++11
category: C++
tags: [C++, C++11]
---

### 前言
本部分内容是自C++ FAQ的学习笔记，部分内容因之前已学过，
此外也有部分与Stroustrup的FAQ重复。
这种情况只简略回顾一下。

链接： <https://isocpp.org/wiki/faq/cpp11>

### C++11: Purpose of this FAQ Section
主要是简略学习一下C++的语法特性，并不会深入细节，
所以只能算一个入门材料，会涉及一些例子。

### What is C++11
应该把它看成对C++98的一次很大的革新。

### What is C++0x
因为本预计标准10年前出来，所以非官方的说法也是C++0x，
结果没想到拖到11年才有正式的标准发布。其实跟C++11是一回事。

### When will compilers implement C++11
最先的完整实现C++11语法标准的应该是GCC 4.8.1(2013/05/31)，但其STL没有很好的支持。
完整支持(包括STL)的应该是Clang 3.3(2013.06.05)

**关于C++11实现的编译器对比** 参见：<http://wiki.apache.org/stdcxx/C%2B%2B0xCompilerSupport>

### How did the committee approach picking new language and library feature for C++11?
C++11标准并不能简简单单地堆砌一些有用的特性，它需要考虑兼容性以及语言的统一性。

一个特性是否会纳入C++11主要考虑是否符合语言的设计目标(set of specific design goals)

C++11着重于提高抽象机制，这不同于通常人们所理解的“类”或者“对象”。
C++11的含义远高于此：user-defined types的意义(range)能被干净地、安全地表示(expressed)
这依赖于添加的如下特性：

* [initializer-lists](https://isocpp.org/wiki/faq/cpp11-language#init-list)
* [uniform-initialization](https://isocpp.org/wiki/faq/cpp11-language#uniform-init)
* [template aliases](https://isocpp.org/wiki/faq/cpp11-language-templates#template-alias)
* [rvalue reference](https://isocpp.org/wiki/faq/cpp11-language#rval)
* [defaulted and deleted functions](https://isocpp.org/wiki/faq/cpp11-language-classes#default-delete)
* [variadic templates](https://isocpp.org/wiki/faq/cpp11-language-templates#variadic-templates)

而以上特性的实现也因一下特性变得简单：

* [auto](https://isocpp.org/wiki/faq/cpp11-language#auto)
* [inherited constructors](https://isocpp.org/wiki/faq/cpp11-language-classes#inherited-ctors)
* [decltype](https://isocpp.org/wiki/faq/cpp11-language#decltype)

这些大的改进使得C++11更想一个新语言

而对于STL而言，11的STL比98的STL表现更好。

### What were the general design goals of the C++11 effort?
C++的宗旨是：

* a better C
* 支持data abstraction
* 支持面向对象编程
* 支持泛型编程

整个C++11的努力是强化了：

* 使得C++成为一个更好的针对system programming和library building的语言
* 使得C++更容易教与学。即增加了统一性、stronger guarantees和更便于新手使用
* 还有就是C++演进过程中严格遵守向后兼容性。虽然已对某一特性有更新的语法支持，
但是委员会依然会保留旧的特性，例如:
[static_assert](https://isocpp.org/wiki/faq/cpp11-language-misc#static-assert)
[null_ptr](https://isocpp.org/wiki/faq/cpp11-language#nullptr)
[constexpr](https://isocpp.org/wiki/faq/cpp11-language#constexpr)

### What specific design goals guided the committee?
略
无非介绍了一下C++的设计宗旨，包括兼容性、更好的库扩展、增加类型安全、更好的硬件表现等。

有一定还是值得提一下，即C++面对的不同领域：

#### Machine model and concurrency
提供对现代的并行硬件更好的支持，包括：
**threads**，**future**，**thread-local-storage**和**atomics**

#### Generic programming
提供对泛型编程更好的编程，包括**auto**，**template alias**等

#### Systems programming
增加了更贴近底层硬件的支持和效率（例如底层的嵌入式系统编程），
例如**constexpr**，**std::array**和**generalized PODs**(参见我的wiki关于C++:POD)

#### Library building
增加了一些特性，以解决abstraction mechanisms的限制的、低效的及不合常规的地方，
例如：**inline namespace**, **inherited constructors**以及**rvalue reference**

### Where can I find the committee papers for C++11 features?
参见：[the committee papers archive](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/)
但是好像太细了，讨论各种可行的不可行的特性，没有时间不值得看。

### Where can I find academic and technical papers about C++11?
就不一一列举的，详情参见：
<https://isocpp.org/wiki/faq/cpp11#cpp11-technical-papers>

### Where else can I read about C++11?
参见：
<https://isocpp.org/wiki/faq/cpp11#cpp11-other-reading>
这里还是有很多不错的材料，值得推荐

### else
略

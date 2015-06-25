---
layout: post
title: "C++填坑系列：tip1 视C++为一个语言联邦"
description: "C++填坑，主要来自effective c++"
keywords: c++, cpp, effective
category: c++
tags: [c++]
---

## 前言

大部分内容来自《Effective C++》以及《More Effective C++》，但结合自己的认识和例子。无奈C++确实太多坑了。

## tip1 视C++为一个语言联邦

C++是一个多重范式的编程语言，主要可以视为以下四类：

### 1. C style

### 2. 面向对象C++

### 3. Template C++

### 4. STL

有些高效编程的守则在不同的类型情况下有所区别，例如：

#### pass-by-value 还是 pass-by-reference ?

例如传统C style中，`pass-by-value`比`pass-by-reference`更高效。

然而对于Object-Oriented C++以及Template C++而言，`pass-by-reference-to-const`往往更好。这是由于构造函数、析构函数存在的原因，copy要消耗一定的资源。Template C++连对象类型都不知道，更是如此。

然而，对于STL而言，由于迭代器和函数对象都是在C指针上塑造的，所以`pass-by-value`更适用。

---
layout: post
title: "C++填坑系列：tip5 了解C++默默编写并调用哪些函数"
description: "C++填坑，主要来自effective c++"
keywords: c++, cpp, effective
category: c++
tags: [c++]
---

### 概述

C++的default构造函数、析构函数，copy构造函数以及copy asignment操作符比较特殊，

可以不用自己定义它们编译器自动为你加上去，如果加进去的话，默认是public、inline、non-virtual的。

当然，编译器在你使用了该函数的情况下，才会加进去（通常情况下肯定会使用啦）。

其次，对于copy asignment操作符，编译器单纯的将每个non-static成员变量copy到目标函数。故不能作用于const类的，
因此，定义copy构造函数以及copy asignment操作符为私有类的，以及其derived类（derived类的copy函数会调用base类的copy函数）都不行。

### 总结

#### 默认public、inline、non-virtual

#### default 构造函数

如果已经声明了一个构造函数，编译器不再添加default构造函数

#### copy构造函数

1. 如没定义copy操作符，编译器将为你生成。
2. copy构造函数将调用成员的“copy函数”，如果成员是类例如string 就是copy构造函数，如果成员是int等内置类型就是copy每个bits完成初始化

#### copy assignment操作符

相对copy构造函数，copy assignment要考虑到const的情况，当copy assignment合法且有意义编译器才会生成。

例如，如果class内含const成员或者reference成员，那么赋值就不合法了，
因此编译器拒绝为其生成copy assignment操作符，除非自己定义。

如果base classes中copy assignment操作符是private的，
那么derived class因为无法调用base class的copy assignment操作符，因此编译器也不会默认生成。

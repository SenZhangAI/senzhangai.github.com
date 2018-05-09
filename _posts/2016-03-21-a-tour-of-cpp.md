---
layout: post
title: "《a Tour of Cpp》笔记"
description: "阅读笔记"
keywords: C++
category: C++
tags: [C++]
---

## 前言
《a Tour of C++》是Bjarne Struoustrup的一本类似C++导读的书，书中简述了C++的诸多编程风格，值得一读。

本文将其中重要的部分整理成笔记。很久没做笔记了，倒不是偷懒，只是在拉进度，觉得整理笔记有些耗时。但反过来，如果重新查看书中的内容，发现更难找，更耗时间。

但本文并不是系统性地整理内容，如果有些我觉得自己很熟悉了就忽略掉。

## 1. Basic

### 类型转化
对于计算符，例如乘法，`int * double`，类型转化的规则是取最高精度
对于赋值，可能存在**narrowing conversions**，这完全是为了兼容C，而且不安全，
如果要规避这一问题，可以用`{}-list form`来初始化，例如：

```cpp
int i = 7.2   //i becomes 7
int i2{7.2}   //error, floating-point to integer conversion
```

### Scope and Lifetime
变量声明的作用域分为以下几种：

1. Local scope 声明于function或者lambda内，作用域从声明处到`}`，函参视为local name
2. Class scope 定义于某个class内，在任何function, lambda,enum class之外，作用域在整个class`{`及`}`内
3. Namespace scope 定义于namespace，在任何function，lambda，enum class之外，作用域到声明处到`}`

如果都不在以上的范围内，则称为global name，或称其在global namespace中。

对于namespace内的变量，destruction发生在整个程序结束。

对于new对象（我认为也包括static对象）生命期独立于scope

### Constants
C++语法规定了有些地方必须用常量，比如array bounds，case labels，template value arguments以及常量声明

### Pointer,Arrays,References
只有唯一的`nullptr`shared by all pointer type。

旧风格常用NULL或者0来表示空指针，这不是类型安全的，因为容易跟int混淆，
最好用nullptr，`int a = nullptr`将造成类型不匹配错误，而NULL不会。

### Advise
keep common and local names **short**, and uncommon，nonlocal names longer

Perfer `{}-initializer` in declareations with a named type(因为避免narrowing conversion)

Perfer the `=` syntax for the initialization in declarations using **auto**(参考Effective Modern C++ Item 2，auto的统一初始化需要初始化list中各项是同一种类型才能推导T)

## User-Defined Types
C++的抽象机制主要包括classes and enumerations

这里classes也包括struct

而enumerations指的是enum class，例如`enum Class Color{}`,枚举类，
而不是`enum Old_Color{}`，旧风格枚举非类型安全，可以赋值给int，或者被int赋值，
而枚举类**强类型安全**，除了默认的比较、赋值等操作外，还跟类一样也可以定义其他函数。

### class
关于class我觉得甚至有必要单独写一篇博文，这里只简述本书的内容，

class的重要概念之一是接口与实现，接口定义于public，实现定义于private
C++中对于varying amounts of information的处理常用handler，handler是fixed-size

### Unions
编译器不会也无法判断程序员想要的确切类型，所以这部分还需要程序员自己负责。
比如说用一个`enum Type`来指示union类型的trick(也称为tagged union)，而这样也增加了额外的空间。

如上原因，union也是易错的，所以通常封装起来。

### Enumeration
如上所述enum class与plain enum(兼容C)的区别，
补充一下：enum class需要`::`,plain 不需要

### Advice
Avoid "naked" unions, wrap them in a class together with a type field

Perfer class enums over plain enums

## Modularity

### Separate Compilation
分离编译 -> 半独立的代码片段 -> 最小化编译时间 + 强制的逻辑分段隔离（这样出错概率更小） + user只知道声明不知道实现

### namespace
Namespace主要用于组织大型的程序组件，例如libraries

```cpp
void MyNameSpace::func() {
    //注意，此block也在MyNameSpace命名空间中
}
```

### Error Handling
Error Handling是一个复杂而重要的部分

C++提供多种Error handling机制，例如：

1. type system
2. Exception，run-time错误检测与处理的隔离
3. class等各种模块化、抽象机制的隔离，例如list 4
4. class的不变式(invariants)
5. assert

#### Exception
out_of_rangey异常定义于STL中的`<stdexcept>`

noexcept的函数如果抛出异常，则系统将执行standard-library function: `terminate()`

#### Invariants
由constructor来建立类的不变式

#### Static Assertions
static_assert可用于任何constant expressions，因为static嘛，编译器必须能确定
static_assert 对于泛型编程的类型检查很有用

#### Advice
Use namespace to express logical structure

Develop an error-handling strategy early in a design

Use purpose designed User-Defined types as exception(not built-in types)这样才能更直观地表达错误原因

能在编译器检查的错误就不要放到运行期，例如(using static_assert)

## Classes


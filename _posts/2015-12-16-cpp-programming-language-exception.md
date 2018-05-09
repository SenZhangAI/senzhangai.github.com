---
layout: post
title: "Exception in C++ Programming language"
description: "《C++ Programming Language》Exception章节"
keywords: c++, cpp, Exception, Error handling
category: C++
tags: [c++, exception]
---

## 前言
本想整理在Exception FAQ 日志中的，但是内容实在有些多，
故单独分成一篇。

## Error Handling
本节所述错误处理策略主要包含两个方面：

1. exception-safety guarantees, 着眼于从rum-time错误中恢复
2. RAII, 用ctor，dtor管理资源

以上两个方面都基于**不变式**，所以也涉及到**enforcement of assertions**

### Exceptions
本节介绍了什么是Exception以及其优点

An exception is an **object** *thrown* to represent the occurrence of an error.
It can be of **any type that can be copied**.

通常用类或者struct作为异常较为合适，因为可实现**层次结构**，以及可**携带错误信息**。

### Traditional Error Handing
介绍传统错误处理的方式，以及其局限性。这在Exception FAQ中详细说明，不展开。

### Muddling Through蒙混过关
对于异常而言，如果漏掉了catch通常会终止程序，这样系统也变得脆弱，不像传统处理方式可以蒙混过关。
对于开发而言，暴露问题当然最好，但发行给用户时，还是小心处理，
最好catch all避免程序终止，并提供必要措施。
对于程序开发，提供错误信息，对于用户尽量不把问题抛出来，能解决问题最好。

### Alternative Views of Exceptions
有些错误函数本地内就能处理，就没必要抛出异常。抛出异常的是本地无法处理的错误。

### Asychronous Events
C++中的异常指的是同步的异常，对于异步异常，例如按下终止键，则是系统中断信号，机制不一样。

### Exceptions That Are Not Errors
千万别把异常机制当做正常的分支处理程序，这样的奇淫巧计并没有什么用，带来代码混乱和低效。

### When You Can't Use Exceptions
在Exception FAQ中也有这部分内容。

以下情况最好不用异常：

* A time-critical component of an embedded system
    因为对于实时要求，而异常从throw传播到catch缺乏时间检测工具，因此不行。
* 如果是处理一个老旧的大型程序，资源管理还是ad-hoc(点对点)方式。例如用野指针new-delete，
    而不是采用系统性地设计，例如resource handles。
    如果遇到这种情况，那么，只好入乡随俗，采用“传统”的方式，因为约束条件实在太多。
    处理这个情况给两个建议：

1. **模仿RAII**
类似如下代码：

```cpp
void f(int n) {
    my_vector<int> x(n);
    if( x.invalid() ) {
        // ...deal with error ...
    }
    // ...
}
```

即，在**每个**class的constructor中，用一个invalid()函数来验证不变式，
出错则返回错误码，例如返回0则为正确，其他为错误。
当ctor中请求资源失败的情况下，**首先确保无资源泄露**，然后再用invalid返回错误码。

2. **模仿函数抛出异常**
这一通过返回一对值（原因之前已经描述）例如`pair<Value, Exception>`。
Value是实际返回值，Error_code则表示“throw”的异常。例如：

```cpp
void g(int n) {
    auto v = make_vector(n); //return a pair
    if ( v.Exception ) {
        // ... deal with error ...
    }
    auto val = v.first;
    // ...
}
```

#### Hierarchical Error Handling
TODO

#### Exceptions and Efficiency
TODO

#### Exception Guarantees
TODO

#### Resource Mananger
TODO



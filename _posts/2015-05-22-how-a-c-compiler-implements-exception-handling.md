---
layout: post
title: "How a C++ compiler implements exception handling[译文]"
description: "探索异常处理的机制，翻译文献"
keywords: "C++, compiler, exception handling, 异常处理"
category: "Programming"
tags: [C++, 异常处理]
---

## 扩展链接

[原文网址](http://www.codeproject.com/Articles/2126/How-a-C-compiler-implements-exception-handling)

[知乎：如何用C语言实现异常/状况处理机制](http://www.zhihu.com/question/20597909/answer/32371076)

[译文A Crash Course on the Depths of Win32 Structured Exception Handling](http://www.cnblogs.com/awpatp/archive/2010/06/15/1758763.html)

## 正文

### 概述

深入讨论了VC++实现异常处理的机制，包括VC++异常处理的源码（[Download](http://www.codeproject.com/KB/cpp/Exceptionhandler/Exceptionhandler_src.zip)）。

### Introduction

C++优于传统语言的一个特性是异常处理（exception handling）。对于传统技术中的不足和容易出现错误，C++的异常处理提供很好的代替。正常代码和异常处理代码之间良好的分离使得程序更为整洁，更易于维护。

本文讨论编译器如何实现异常处理，同时呈现了异常处理机制和它的语法。本文还包含了我实现的异常处理源码，要使用此异常处理的库，需要首先替换掉VC++默认的异常处理库。代码如下：

```cpp
install_my_handler();
```

执行上述代码后，任何异常相关的过程，从抛出异常（throwing an exception）到stack upwind都由本文的库处理，而不是VC++的默认库。

C++标准仅指明异常处理的特性而不没有限定其实现。这意为着每个供应商都能自由地选择自己认为合适的实现机制。我将描述VC++如何实现这一特性，然而这也是一份很好的学习资料，即便您用的是其他编译器或操作系统[^1]。

VC++的异常处理库依赖于Windows操作系统底层中的Structued Exception Handling(SEH)[^1]。

### SEH-Overview

此次讨论中，我将考虑那些显式抛出的异常以及在某些条件下发生的异常，例如除以零或者空指针的访问。当异常发生时，生成中断（interrupt）并将控制权移交给操作系统。之后，操作系统调用异常句柄（exception handler）检查从异常发生处开始的函数调用序列（the function call sequence）[^1]，并执行stack upwinding以及控制权转移。我们能写出我们自己的异常句柄并注册到操作系统中，它（异常句柄）将在异常发生时被调用。

Windows系统定义了一个专门的异常注册数据结构，**EXCEPTION_REGISTRATION**:

```c
struct EXCEPTION_REGISTRATION{
    EXCEPTION_REGISTRATION *prev;
    DWORD handler;
};
```

为了注册你自己的异常句柄，需要创建该数据结构并将其存储与FS寄存器

### Notes and Reference

[^1]: 本文主要实现于VC++ 6.0，同时也测试了VC++ 5.0以及VC++ 7.0beta

[^1]: 参见A Crash Course on the Depths of Win32 Structured Exception Handling[译文](http://www.cnblogs.com/awpatp/archive/2010/06/15/1758763.html)[原文](http://www.microsoft.com/msj/0197/Exception/Exception.aspx)

[^1]: 我觉得可以理解为函数调用堆栈

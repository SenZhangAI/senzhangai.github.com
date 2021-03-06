---
layout: post
title: "CSAPP要点总结 第3章 程序的机器级表示 part5 汇编数组与异质的数据结构"
description: "学习《深入理解计算机系统》第3章 程序的机器级表示 part5 汇编数组与异质的数据结构"
keywords: CSAPP, 操作系统, 计算机系统, 汇编, 优化, 机器码, 数据, 结构
category: CSAPP
tags: [汇编, computer system ]
---

## 数组
### 数组与指针运算
这部分内容较为简单，所以不打算详细写出来。

大致而言，就是注意是：

1. 地址还是地址指向的数组，以及取地址`leal`
2. 数据类型的长度，例如如果是int的4字节数组E[i],则是类似`(%edx,%ecx,4)`这类操作数，其中`%edx`是基址，`%ecx`是i。

### 嵌套的数组
即多维数组。
对于例如 `int A[5][3]`,
等价于：

```c
typedef int row3_t[3];
row3_t A[5];
```

看成一个5行3列的二维数组。C语言是行优先顺序，有些语言例如Fortran是列优先。

### 定长数组
C语言可以优化定长多维数组的代码。
简单来说，其基本原理是根据数组实际计算的规律，例如每次都是变动j，i不动 或者每次都是变动i， j不动之类的规律，来加入一个中间变量。
这样避免了每次都要从头计算数组地址。

这些优化编译器会自动完成。

### 变长数组
历史上，老版本的C基本上只支持定长数组。如果需要变长数组，不得不用malloc或者calloc，且不得不显示编码，用行索引将多维数组映射到一维数组。

到了ISO C99则引入了允许数组的维度是表达式，在数组分配时才计算出来。即变长数组。

## 异质的数据结构



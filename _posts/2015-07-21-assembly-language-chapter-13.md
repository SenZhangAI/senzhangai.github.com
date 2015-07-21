---
layout: post
title: "汇编语言 第13章 int指令"
description: "汇编语言笔记要点"
keywords: assembly, 汇编, 机器码, 8086, 寄存器, BIOS
category: 汇编
tags: [汇编, 底层, int指令, BIOS]
---

## int指令
由上一章节知，int中断执行以下过程：

1. 取中断类型码n
2. pushf，IF = 0，TF = 0
3. push CS, push IP
4. (IP) = (n*4)，(CS) = (n*4 +2)
5. 转到n号中断程序处理中断。

以后，我们可将中断处理程序简称为**中断例程**。
例程意思是例行事务性的子程序。

[百度百科]:

> 例程的作用类似于函数，但含义更为丰富一些。例程是某个系统对外提供的功能接口或服务的集合。
> 比如操作系统的API、服务等就是例程；
> Delphi或C++Builder提供的标准函数和库函数等也是例程。
>我们编写一个DLL的时候，里面的输出函数就是这个DLL的例程。

## BIOS和DOS所提供的中断例程
在系统板的ROM存放着一套程序，成为BIOS，BIOS主要包含以下几个部分：

1. 硬件系统的检测和初始化
2. 外部中断和内部中断的中断例程
3. 用于对硬件设备进行I/O操作的中断例程
4. 其他和硬件系统相关的中断例程

操作系统DOS也提供了中断例程，从DOS操作系统的角度，DOS操作系统提供的中断例程就是向程序员提供的编程资源。

BIOS和DOS的中断例程中包含许多子程序，可供程序员使用，用int指令调用。

## BIOS和DOS中断例程的安装过程
之前的讲解的int中断可以调用0000:03FF中断向量表所指向的的中断例程。
那么为了加载我们自己的中断程序，
就是选择一块地方存放我们的子程序，
对于8086通常0000:0200到0000:02FF是空的（这是中断向量表0000:03FF的预留的一部分空间），
所以将代码片段存放在该处。
然后修改0000:03FF中断向量表中某个中断码n对应的IP、CS。

那BIOS和DOS是如何安装中断例程呢？对于8086步骤如下：

1. 开机后，CPU一加电，初始化(CS)=0FFFFH，(IP)=0，自动从FFFF:0单元开始执行程序。FFFF:0处有一跳转指令，CPU执行该指令跳到BIOS的硬件系统检测和初始化程序。
2. 初始化程序将建立BIOS所支持的中断向量，即将BIOS中断例程入口地址登记到中断向量表。(与我们自己写的中断程序不同点在于，不用copy中断程序代码段到某地址，因为BIOS是固化在ROM上，一直在内存中，只需处理中断向量表)
3. 硬件检测与初始化完成后，调用int 19h进入操作系统的引导。自此，计算机交由操作系统控制。
4. DOS启动后，除完成其他工作外，还将它提供的中断例程装入内存，并建立相应的中断向量（跟我们加载中断程序的方法一样）。

从以上步骤可知，BIOS主要解决几个问题：

1. 设定CPU启动后作为第一个执行程序
2. 硬件检测与初始化、注册中断程序
3. 交控制权给操作系统

## DOS中断例程示例——程序返回

```
mov ax, 4c00h
int 21h
```

这段代码都出现在程序的结束段，实际上这是DOS提供的中断程序。
具体含义是：

int 21h即DOS提供的程序返回的中断程序，
而该程序执行时会查看ah、al中的内容（相当于函数的参数）

```
mov ah, 4ch ;该参数指示调用21h号中断例程的4ch号子程序（程序返回）
mov al, 00h ;该参数指示程序返回值，即返回0
int 21h     ;调用DOS提供的中断，该中断程序查看ah,al中的内容作为程序参数
```

实际上，BIOS、DOS的中断程序大多以某些寄存器，如ah、al、dh、dl、dx等等特定寄存器中的内容作为程序的参数来调用不同功能的子程序，如返回、显示某字符串等等。



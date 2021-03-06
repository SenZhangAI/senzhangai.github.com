---
layout: post
title: "汇编语言 第4、5章 [BX]和LOOP"
description: "汇编语言笔记要点"
keywords: assembly, 汇编, 机器码, 8086
category: 汇编
tags: [汇编, 底层]
---

## 第4章概述
第4章主要介绍一个文本文件编译执行的过程。
太简单，所以只简要概述：

Edit → main.asm →编译（masm）→ main.obj → 连接（link）→ main.exe → 加载(shell) → 内存中的程序 → 运行（CPU）

执行的过程是：
首先操作系统的shell加载程序到内存，然后将访问权限移交给程序，程序运行结束后返回，权限返还给shell。

### DOS中的程序段前缀PSP
对于DOS而言，生成的.exe文件在初始地址0H到10H之间创建一个成为程序段前缀（PSP）的数据区域。
DOS利用PSP来和被加载的程序之间进行通信。

打个比方说，在载入某可执行二进制文件try.exe后
在debug中，假设初始DS = 0B2D，则 0B2D:0 ~ (0B2D+10H:0 - 1) 这100H(256字节)大小的空间内存放PSP信息，
因此程序真正的入口比DS大10H，即CS = 0B3D。

如果非.exe载入文件方式，而是直接在debug中编辑，则无psp信息，则此时初始化的CS=DS

补充：以上知识点可能过于老旧，毕竟是用在8086上的文件格式，下面补充一下Linux的标准可执行文件格式ELF：
参见： [ELF-百度百科](http://baike.baidu.com/subview/1090277/10973487.htm#viewPageContent) [ELF-wikipedia](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)

>在计算机科学中，是一种用于二进制文件、可执行文件、目标代码、共享库和核心转储的标准文件格式。
>是UNIX系统实验室（USL）作为应用程序二进制接口（Application Binary Interface，ABI）而开发和发布的，也是Linux的主要可执行文件格式。
>1999年，被86open项目选为x86架构上的类Unix操作系统的二进制文件标准格式，用来取代COFF。因其可扩展性与灵活性，也可应用在其它处理器、计算机系统架构的操作系统上。

## [BX]
即对于数据区域DS:[addr]而言，用寄存器BX中的值代表addr

常用的指令是inc bx,
相当于"bx++"，指向下一个数据字节。
注意对于一个字长为两个字节的单位，要inc两次再mov

可见，[bx]在循环处理内存中的数据时很有用。

此外，还有[BX]还有另一个用途:
如果我们在debug中编辑并运行

    mov ax, [0]

没有问题，ax将被ds:[0]赋值，[]代表了内存地址的意思。然而，如果是放在文件中编辑，并编译为.exe文件，那么将会编译为 mov  ax 0H。
显然这不是我们想要的（奇怪的语法）。

解决这个问题的办法有两种，显式说明ds:[0],即：

    mov ax, ds:[0]

要么，用寄存器

    mov ax, ds:[bx]

那么为什么要显示说明ds:[0]呢？
猜测原因是[0]不仅可以用在ds:[0]这样的场合，还有其他比如：
cs:[bx]、ss:[0]、es[1]等用途。而非ds专用，所以需要指明。


## Loop指令
首先给cx设置值
判断cx是否为0，不为0则跳转到loop标记处，cx--。
当cx等于0跳出循环，继续往下执行。

例如计算2^12,可以：

    assume cs:code
    code segment
        mov ax, 2
        mov cx, 11
    s:  add ax, ax
        loop s
        mov ax, 4c00H
        int 21H
    code ends
    end

## 安全的内存空间
DOS并没有提供对数据的保护机制，因此DOS系统也是不安全的，可以随意修改其内容。但这样会带来很多安全问题，或者死机。
通常约定了0:200 ~ 0:2ff内是安全的，其他程序不会占用，因此试验中可以用这段空间。

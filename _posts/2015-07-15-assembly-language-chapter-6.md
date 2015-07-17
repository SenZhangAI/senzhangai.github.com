---
layout: post
title: "汇编语言 第6、7章 包含多个段的程序"
description: "汇编语言笔记要点"
keywords: assembly, 汇编, 机器码, 8086
category: 汇编
tags: [汇编, 底层]
---

## 第6章概述 包含多个段的程序
本章主要讲了如何组织汇编代码，
主要是用伪码标记指明**代码段**、**数据段**、**栈段**以及**程序入口**。

就本质而言，
IP将被设置在标记了**程序入口**的地方，所以才从该处执行。
而ds和ss则需要汇编代码中指定，以如下代码为例：

    assume cs:code, ds:data, ss:stack ;指定标记，这样确认了数据大小，也就确认了位置
                                      ;cs指向code ds指向data ss指向stack
    data segment
        dw 0123h, 0456h, 0789h, 0abch, 0defh, 0cbah, 0987h
    data ends
    stack segment
        dw 0, 0, 0, 0, 0, 0, 0, 0  ;dw 即 define word
    stack ends
    code segment
    start:  mov ax, stack     ;指定程序入口，有冒号
                              ;可以看到只要是指定寄存器位置的伪码都有冒号
                              ;如 assume中也是
                              ;但是这里还是得通过赋值给ss等寄存器指定
            mov ss, ax        ;指定了栈数据区域
            mov sp, 16        ;指定栈大学
            mov ax, data
            mov ds, ax        ;指定了数据区
            push ds:[0]
            push ds:[1]
            mov ax, 4c00h
            int 21h
    code ends
    end start ;此处指定了start标记

此代码编译后放到内存中的数据是按照代码中的顺序连在一起的

## 第7章 更灵活的内存定位方法
### 有趣的ASCII大小写转换
说句题外话，发现了一个有意思的ASCII码大小写转换的方法，
A 0100 0001 a 0110 0001
B 0100 0010 b 0110 0010
可以看出其实大写小写只有一位不同，那么通过位运算就很好解决了：
转换成大写  x & 1101 1111
转换成小写  x | 0010 0000

### [bx+idata]
例如[bx+200]就是在bx偏移量的基础上再加上200的偏移量
也可以写出idata[bx]的形式

[bx+idata]为高级语言实现数组提供便利机制

### SI和DI
8086的si和di寄存器与bx功能类似，打不能分成两个寄存器

### [bx+si+idata]和[bx+di+idata]
有了si、di寄存器的支持，就有了更灵活的
[bx+si]、[bx+di]甚至[bx+si+idata]和[bx+di+idata]的支持。
这在需要做二维处理时很有用！

不同寻址方式的处理本质而言，是对处理数据的结构的理解，
清楚哪些地方会变，哪些地方是固定的偏移量之类。

### 内外循环
因为loop循环都保存在cx中，
那么如果内外循环将出错。

解决办法是push cx 、 内循环、 pop cx
之所以用到栈，或者说内存地址是因为寄存器数量毕竟有限。
如果寄存器很充足的情况下，用 mov dx cx、内循环、 mov cx dx也行。


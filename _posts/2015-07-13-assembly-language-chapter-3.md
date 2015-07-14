---
layout: post
title: "汇编语言 第3章 寄存器(内存访问)"
description: "汇编语言笔记要点"
keywords: assembly, 汇编, 机器码, 8086
category: 汇编
tags: [汇编, 底层]
---

## 内存中字的存储
从前面的试验也可以顺便看出（试验平台是win7 普通intel PC机）
普通PC机，对于多字节（字）的存放顺序是小端。

## DS和[address]
为了告诉CPU。需要访问内存的哪个地方需要DS和[address]
DS(Data Segment)是内存的段地址
[address]则是偏移地址,address是整数，后面章节可知address也可是寄存器
（win7 模拟8086中不能执行[address]的命令额）

与CS一样，8086并不支持DS直接改变其值（硬件设计问题），而是通过通用寄存器
例如：

    mov bx, 1000H
    mov ds, bx

8086有16根数据线，可以一次传送16位数据。

## 栈
需要两个地址，一个是栈地址，一个是栈顶指针。
SS(Stack Segment)寄存器的数据是栈的段地址。
SP(Stack Pointer)寄存器的数据是栈顶指针，指向栈顶元素

同样的8086并不能直接给SS赋值，不要中转，例如：

    mov ax, 1000H
    mov ss, ax

SP可以直接赋值。

### 执行步骤
push ax命令执行以下两步：
(1) SP = SP - 2
(2) 将ax数据入栈

pop ax命令执行以下两步：
(1) 弹出栈顶元素，赋值给ax
(1) SP = SP + 2

### 越界问题
空栈pop或者满栈push都会发生越界，
不幸的是8086并没有提供特殊的flag寄存器检测越界。

### push/pop 对象
push/pop 寄存器/段寄存器/内存单元

### 总结
可见对于CPU而言，同样是二进制的一个段，
要使其作为代码，就设置CS:IP
要使其作为数据，就设置DS:[address]
要使其作为栈，就设置SS:SP

此外，第75页的试验2.（2）很有意思，
如果没猜错的话，8086寄存器的地址在该处（2000:0）附近，
此处略

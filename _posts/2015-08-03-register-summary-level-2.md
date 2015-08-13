---
layout: post
title: "[转载]寄存器总结 level 2"
description: "转载一文，进一步总结了各个常用寄存器，扩展了32位平坦寻址，加深对寄存器进一步理解"
keywords: 寄存器, 汇编, 标志寄存器
category: 汇编
tags: [寄存器, 转载]
---

## 前言
level 1： [汇编语言 第11章 标志寄存器]({% post_url 2015-07-19-assembly-language-chapter-11 %})

此文为level 2，因此文写得太好了！解开 level 1 中的很多疑惑，故转载于此。
来自于百度文库，估计来源于教材。因作者不详，无法与作者取得联系，如侵权，即删除

本文主要为理解FS段寄存器，所以相关内容放在前面，后面才是总结各个寄存器

## 平坦寻址
因为Win32中的地址为**平坦模式**，**ds,ss,cs等各段的段基地址都指向同一个地方**，所以平常用的逻辑地址都是默认以这些段为基址的，不管是数据段还是代码段，只要他们的偏移相等，那么他们就是寻址一样的物理内存，所以我们就**只需指明偏移就能得到统一的寻址目标**，不管这个目标是在代码段还是数据段或者堆栈段之中。WriteProcessMemory用的就是这样的“逻辑地址”，像"0xXXXXXXXX"，其实，这样的逻辑地址实际上是这样的形式：ds:XXXXXXXXX，只因为ds=ss=cs，所以可以用ds:XXXXXXXXX来寻址cs:XXXXXXXX或者ss:XXXXXXXX，也就是只需要指定偏移就足够寻址代码段或者数据段了。

## FS段寄存器
win32中，fs却和各个段寄存器的值不一样，所以要想用ds来寻址fs指向的内存，那就得转换偏移了，因为fs:XXXXXXXX和ds:XXXXXXXX指的并不是同样的内存；而如果用WriteProcessMemory寻址某个地址如 0xXXXXXXXX的话，它默认的寻址却是ds:XXXXXXXX。

为什么FS寄存器要这样设置呢？FS实际上跟ES、GS一样，是附加的寄存器，但有其特殊用途，其指向当前活动线程的TEB结构（线程结构）

FS寄存器指向当前活动线程的TEB结构（线程结构）
注意：以下结构可以看出适用于32位系统，64位系统相关结构应该有所不同，但估计大致内容一致。
偏移 说明
000 指向SEH链指针
004 线程堆栈顶部
008 线程堆栈底部
00C SubSystemTib
010 FiberData
014 ArbitraryUserPointer
018 FS段寄存器在内存中的镜像地址
020 进程PID
024 线程ID
02C 指向线程局部存储指针
030 PEB结构地址（进程结构）
034 上个错误号
举例：

``` nasm
POP DWORD PTR FS:[004]
```

这个句指令的意思就是将堆栈顶部的4个字节的字符弹栈出去！

## 寄存器总结
32位计算机的寄存器一般分为以下几类

* 4个数据寄存器(EAX、EBX、ECX和EDX)
* 2个变址和指针寄存器(ESI和EDI) 2个指针寄存器(ESP和EBP)
* 6个段寄存器(ES、CS、SS、DS、FS和GS)
* 1个指令指针寄存器(EIP) 1个标志寄存器(EFlags)

### 数据寄存器
AX称为**累加器(Accumulator)**，**用累加器进行的操作可能需要更少时间**。累加器可用于乘、除、输入/输出等操作，它们的使用频率很高；
BX称为**基地址寄存器(Base Register)**。它可作为存储器指针来使用；ds:[bx]
CX称为**计数寄存器(Count Register)**。在循环和字符串操作时，要用它来**控制循环次数**；在位操作中，当移多位时，要用CL来指明**移位的位数**；
DX称为**数据寄存器(Data Register)**。在进行乘、除运算时，它可作为默认的操作数参与运算，也可用于存放I/O的端口地址。

在16位CPU中，AX、BX、CX和DX不能作为基址和变址寄存器来存放存储单元的地址，但在32位CPU中，其32位寄存器EAX、EBX、ECX和EDX不仅可传送数据、暂存数据保存算术逻辑运算结果，而且也可作为指针寄存器，所以，这些**32位寄存器更具有通用性**。

### 变址和指针寄存器(ESI和EDI)  
寄存器ESI、EDI、SI和DI统称为**变址寄存器(Index Register)**，主要用于存放存储单元在段内的偏移量 ， 通过它们可实现多种存储器操作数的寻址方式，为以不同的地址形式访问存储单元提供方便。作为通用寄存器，也可存储算术逻辑运算的操作数和运算结果 。 它们可作一般的存储器指针使用。在**字符串操作**指令的执行过程中，对它们有特定的要求，且具有特殊的功能

### 指针寄存器(ESP和EBP)  
32位CPU有2个32位通用寄存器EBP和ESP。其低16位对应先前CPU中的BP和SP，低16位数据的存取不影响高16位的数据。

寄存器EBP、ESP、BP和SP称为指针寄存器(Pointer Register)，主要用于存放堆栈内存储单元的偏移量，用它们可实现多种存储器操作数的寻址方式，为以不同的地址形式访问存储单元提供方便。 作为通用寄存器，也可存储算术逻辑运算的操作数和运算结果。
它们主要用于访问堆栈内的存储单元，并且规定：

**BP为基指针(Base Pointer)寄存器，用它可直接存取堆栈中的数据**；
**SP为堆栈指针(Stack Pointer)寄存器，用它只可访问栈顶** 。 

### 段寄存器(ES、CS、SS、DS、FS和GS)
CS——代码段 寄存器(Code Segment Register)，其值为代码段的段值；
DS——数据段 寄存器(Data Segment Register)，其值为数据段的段值；
SS——堆栈段 寄存器(Stack Segment Register)，其值为堆栈段的段值；
ES——附加段寄存器(Extra Segment Register)，其值为附加数据段的段值；

FS——附加段寄存器(Extra Segment Register)，其值为附加数据段的段值；
GS——附加段寄存器(Extra Segment Register)，其值为附加数据段的段值。

**在16位CPU系统中，它只有4个段寄存器.在32位微机系统中，它有6个段寄存器**.
32位CPU有两个不同的工作方式：实模式和保护模式。在每种方式下，段寄存器的作用是不同的。有关规定简单描述如下：
**实模式**： **前4个段寄存器CS、DS、ES和SS与先前CPU中的所对应的段寄存器的含义完全一致**，内存单元的逻辑地址仍为“段值：偏移量”的形式。为访问某内存段内的数据，必须使用该段寄存器和存储单元的偏移量。
**保护模式**： 在此方式下，**情况要复杂得多**，装入段寄存器的**不再是段值**，而是称为**“选择子”(Selector)**的某个值 。

### 指令指针寄存器(EIP)
32位CPU把指令指针扩展到32位，并记作EIP，EIP的低16位与先前CPU中的IP作用相同。
指令指针EIP、IP(Instruction Pointer)是存放下次将要执行的指令在代码段的偏移量。在具有预取指令功能的系统中，下次要执行的指令通常已被预取到指令队列中，除非发生转移情况。所以，在理解它们的功能时，不考虑存在指令队列的情况。
在**实模式下**，由于**每个段的最大范围为64K**，所以，**EIP中的高16位肯定都为0**，此时，相当于只用其低16位的IP来反映程序中指令的执行次序。

## 标志位寄存器

### 一、运算结果标志位
#### 1、进位标志CF(Carry Flag)
进位标志CF主要用来反映运算是否产生进位或借位。如果运算结果的最高位产生了一个进位或借位，那么，其值为1，否则其值为0。
使用该标志位的情况有：多字(字节)数的加减运算，无符号数的大小比较运算，移位操作，字(字节)之间移位，专门改变CF值的指令等。

#### 2、奇偶标志PF(Parity Flag)
奇偶标志PF用于反映运算结果中“1”的个数的奇偶性。如果“1”的个数为偶数，则PF的值为1，否则其值为0。
利用PF可进行奇偶校验检查，或产生奇偶校验位。在数据传送过程中，为了提供传送的可靠性，如果采用奇偶校验的方法，就可使用该标志位。

#### 3、辅助进位标志AF(Auxiliary Carry Flag)
在发生下列情况时，辅助进位标志AF的值被置为1，否则其值为0：
**(1)、在字操作时，发生低字节向高字节进位或借位时**；
**(2)、在字节操作时，发生低4位向高4位进位或借位时**。
对以上6个运算结果标志位，在一般编程情况下，标志位CF、ZF、SF和OF的使用频率较高，而标志位PF和AF的使用频率较低。

#### 4、零标志ZF(Zero Flag)
零标志ZF用来反映运算结果是否为0。如果运算结果为0，则其值为1，否则其值为0。在判断运算结果是否为0时，可使用此标志位。

#### 5、符号标志SF(Sign Flag)
符号标志SF用来反映运算结果的符号位，它与运算结果的最高位相同。在微机系统中，有符号数采用补码表示法，所以，SF也就反映运算结果的正负号。运算结果为正数时，SF的值为0，否则其值为1。

#### 6、溢出标志OF(Overflow Flag)
溢出标志OF用于反映有符号数加减运算所得结果是否溢出。如果运算结果超过当前运算位数所能表示的范围，则称为溢出，OF的值被置为1，否则，OF的值被清为0。
“溢出”和“进位”是两个不同含义的概念，不要混淆。如果不太清楚的话，请查阅**《计算机组成原理》**课程中的有关章节。

### 二、状态控制标志位
状态控制标志位是用来控制CPU操作的，它们要通过专门的指令才能使之发生改变。

#### 1、追踪标志TF(Trap Flag)
当追踪标志TF被置为1时，CPU进入单步执行方式，即每执行一条指令，产生一个单步中断请求。这种方式主要用于程序的调试。
指令系统中没有专门的指令来改变标志位TF的值，但程序员可用其它办法来改变其值。

#### 2、中断允许标志IF(Interrupt-enable Flag)
中断允许标志IF是用来决定CPU是否响应CPU外部的可屏蔽中断发出的中断请求。但不管该标志为何值，CPU都必须响应CPU外部的不可屏蔽中断所发出的中断请求，以及CPU内部产生的中断请求。具体规定如下：
(1)、当IF=1时，CPU可以响应CPU外部的可屏蔽中断发出的中断请求；
(2)、当IF=0时，CPU不响应CPU外部的可屏蔽中断发出的中断请求。
CPU的指令系统中也有专门的指令来改变标志位IF的值。

#### 3、方向标志DF(Direction Flag)
方向标志DF用来决定在串操作指令执行时有关指针寄存器发生调整的方向。具体规定在第5.2.11节——字符串操作指令——中给出。在微机的指令系统中，还提供了专门的指令来改变标志位DF的值。

### 三、32位标志寄存器增加的标志位
#### 1、I/O特权标志IOPL(I/O Privilege Level)
I/O特权标志用两位二进制位来表示，也称为I/O特权级字段。该字段指定了要求执行I/O指令的特权级。如果当前的特权级别在数值上小于等于IOPL的值，那么，该I/O指令可执行，否则将发生一个保护异常。

#### 2、嵌套任务标志NT(Nested Task)
嵌套任务标志NT用来控制中断返回指令IRET的执行。具体规定如下：
(1)、当NT=0，用堆栈中保存的值恢复EFLAGS、CS和EIP，执行常规的中断返回操作；
(2)、当NT=1，通过任务转换实现中断返回。

#### 3、重启动标志RF(Restart Flag)
重启动标志RF用来控制是否接受调试故障。规定：RF=0时，表示“接受”调试故障，否则拒绝之。在成功执行完一条指令后，处理机把RF置为0，当接受到一个非调试故障时，处理机就把它置为1。

#### 4、虚拟8086方式标志VM(Virtual 8086 Mode)
如果该标志的值为1，则表示处理机处于虚拟的8086方式下的工作状态，否则，处理机处于一般保护方式下的工作状态。

可见32位系统将原来的9个标志位（条件码）扩充到13个。
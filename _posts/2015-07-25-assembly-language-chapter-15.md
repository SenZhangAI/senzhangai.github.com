---
layout: post
title: "汇编语言 第15章 外中断"
description: "汇编语言笔记要点"
keywords: assembly, 汇编, 机器码, 8086, 寄存器, 外中断, 键盘虚拟码
category: 汇编
tags: [汇编, 底层, 外中断]
---

## 前言
由14章内容我们知道，CPU与外设交互是通过**端口**。
CPU发送信息给外设需要通过接口芯片（接口内部的寄存器）；
外设给CPU发送信息也是通过接口芯片（接口内部的寄存器）。
而外设的寄存器被抽象为统一的地址，**端口**。

以上说明了底层通信机制，
那现在还有有几个实现上的问题需要考虑：

1. 外设的输入随时发生，CPU如果得知？ 答：通过外中断（对不起，不管CPU先生您现在在忙什么，现在有外设输入，先中断手头的任务）
2. CPU从何处得到外设的输入？ 答：当然是端口咯。
3. CPU获得什么消息？

## 可屏蔽/不可屏蔽外中断
外中断也分为两种类型：（a） 可屏蔽外中断 （b） 不可屏蔽外中断

### 不可屏蔽外中断
可CPU必须现在、立刻、马上处理的外中断，比如，我猜测强行关机或者Ctrl + Alt + Del之类的应该是不可屏蔽中断吧，毕竟出现死循环的情况下，需要强行介入啊。

#### 设置不可屏蔽外中断的方法
简单来说，就是**中断码为2**的外中断，为不可屏蔽外中断。

### 可屏蔽外中断
即Mr. CPU可能现在在忙于一个重要的事情（例如另一个中断之类的）,如果这时候被打断可能会出现死循环，死锁之类的。
因此不希望被非紧急情况下的中断给打扰了，先处理完手头上的事情再马上处理中断。
这些可以先等一等的中断就是可屏蔽外中断。实际上绝大多数中断都是可屏蔽外中断。

那么问题来了，如何知道CPU现在是否可以被立即打断呢？
答案就是标志寄存器中的IF（Interrupt-enable Flag）位。

如果IF位被设置为1，CPU就是在声明，我现在可以被打断，来吧！
如果IF为被设置为0，CPU就是在声明，不好意思，我现在忙于重要的事情，现在不便于处理可屏蔽的外中断。

回忆一下内中断的处理过程：

1. 取中断类型码n
2. pushf，**IF = 0**，TF = 0
3. push CS, push IP
4. (IP) = (n*4)，(CS) = (n*4 +2)
5. 转到n号中断程序处理中断。

其中有一项`IF = 0`，就是在说，我现在在处理中断任务，不宜跳出处理其他中断。
而`TF = 0`则是说，如果debug单步执行，本中断指令完整执行，而不单步。

结合第11章各标志寄存位，
以及第12章了解到的TF（Trap Flag）标记寄存位用来控制单步循环。
现在又学会了IF寄存位的作用。
那还有AF未知。

#### 设置IF的指令

```
sti ;set interrupt 设置IF = 1 
cli ;clean interrupt 设置IF = 0
```

## 以键盘处理过程为例

键盘上有一个芯片对每一个按键进行扫描，当按下一个按键时，产生一个扫描码（称为通码）；放开一个按键时，产生另一个扫描码（称为断码）。

《汇编语言》中关于扫描码的内容有些过时，估计仅针对8086之类的老机器，故不详述。
现在很多硬件设计的扫描码不同，但都映射为统一标准的**键盘虚拟码**，详情参见[此文](http://blog.csdn.net/whatday/article/details/7054643)

### 8086中按键引发的中断消息
键盘与8086的通信通过60h端口，当有按键事件发生时，产生int 9的中断。

整个过程为：

1. 键盘产生扫描码
2. 扫描码送入60h端口
3. 引发9号中断，int 9
4. CPU执行9号中断中的例程

还有一个问题，比如shift+a，ctrl + s之类还有控制键或者切换键insert之类的，系统如何知道呢？
8086是这样实现的：

1. 对于字符键，例如a、b、q之类可屏显的按键，将扫描码和字符对应ASCII码一起送入机器。
2. 对于控制键，有特定扫描码，其效果是改变**键盘状态字节**的状态。

例如，对于8086，键盘状态字节存储在0040:17内，0~7bit分别代表右shift，左shift、Ctrl、Alt、Scrolllock、Numlock、Capslock、Insert的状态。

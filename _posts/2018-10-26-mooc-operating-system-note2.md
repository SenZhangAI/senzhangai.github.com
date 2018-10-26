---
layout: post
title: "操作系统课程笔记"
description: "operating system note"
keywords:
category:
tags: [os]
---


## 地址空间与地址生成

逻辑地址 -> 物理地址 是通过MMU查表得到，操作系统所起的作用是建立MMU这张映射表。

## 内存碎片

有程序与程序之间的**外碎片**和程序内部无法利用的**内碎片**，
操作系统关注如何减少外碎片和内碎片。

分配空间策略：

* 首次适配
* 最佳适配
* 最差适配

程序需要的空间是无法预知的，所以这几种分配内存的策略都不是银弹。 目标是尽可能多的有大的地址空间，

除了分配策略，还考虑其他解决地址空间的方式。

方法一是碎片整理，需要考虑：整理的时机？拷贝的开销？

方法二 swapping ，内存与硬盘swap，需要考虑：换哪个程序？什么时候换入换出？开销？ 虚拟内存管理考虑这个问题。

## 非连续内存

为什么采用非连续内存分配？

答：连续内存空间分配容易产生内碎片、外碎片，

非连续内存分配优点：
1. 更好的内存利用和管理
2. 允许共享代码和数据（共享库等）
3. 支持动态加载和动态链接

问题： 管理开销比较大

如何建立虚拟地址和物理地址之间的转换呢？有软件方案和硬件方案，为了加速，采用硬件方案，硬件方案有两种：
 * 分段（例如程序各个段（数据段，代码段...）的特点（有些只读，有些可读写）来管理与共享）
 * 分页（现代操作系统分段管理方式比较少，更多的是分页的管理方式）与分段不同点在于页的尺寸是固定的，而各个段的尺寸可以不一样

## 分页

逻辑地址和物理地址的分页大小要一致，例如如果逻辑地址是4k每页，物理地址也要4k每页。

frame物理页，page逻辑页

逻辑页和物理页的映射关系：
* 页表
* MMU/TLB
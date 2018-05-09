---
layout: post
title: "C++并发编程实战 part3 共享数据"
description: "《C++ Concurrency in Action》阅读笔记"
keywords: C++, concurrency, C++11, Thread, Mutex
category: C++
tags: [C++]
---

## 前言
关于共享数据的安全我觉得是并发编程的重中之重，需要深入学习。

这部分主要介绍：
1. 如何在threads之间共享数据
2. 听过互斥锁mutexes保护数据
3. 其他的保护共享数据的方式

## Problems with sharing data between threads
如果threads之间的共享数据都是read-only的，则没有任何问题。
因为此thread不会影响彼thread。

但，往往会遇到修改数据的操作，这是造成潜在错误的元凶。

这里特别说明C++中的一个概念：
**不变式**(invariants) —— 对于一个特殊的结构（例如类）而言始终为真的声明（statements）
以非编程语言作为比方，例如：对于英文（类），一定是由26个阿拉伯字母组成（不变式）。
Bjarne Stoustrup曾说类就是为了维护其不变式（我理解为类的封装特性），
参见：[关于C++中的不变式](http://my.oschina.net/xtfggef/blog/56558?fromerr=fgRQvkhj)

那么，对于并发编程尤其注意是否能维持不变式，
打个比方，对于vector，`vec.size()`返回元素数量，
那么如果某thead增加或删除元素后，来没来得及修改其size，以维持不变式，而此时另一个thread访问了size，
显然这就破坏了不变式。

### 3.1.1 Race conditions
并发代码中引起bug的一种常见情况

例子：例如电影院买票，很多人在不同售票处买（并发），而有可能会抢同一个座位（race condition），
而且人越多、座位越少时越严重。

定义：In concurrency, a race condition is anything where the outcome depends on the relative ordering of execution of operations on two or more threads;

多数情况下race condition是良性的，例如两个线程分别给queue添加元素，不变式依然维持。
但当race conditions**破坏不变式**时就有问题了。
所以通常说的race conditions指有问题的竞争条件，良性的race condition不考虑。

data race: C++标准定义了**data race**的概率，为race conditions中的一种特殊情况，指并发修改同一个object。
data race容易导致**undefined behavior**。

race condition的常见原因：常常出现在某个步骤需要一串操作，操作分隔的数据片段的时候，
例如删除链表节点，就要操作下游链表节点和上游链表节点。


难以复现：race conditions产生的bug是概率性时间，很难复现。

并发编程很大一部分复杂度就来自于race conditions。

### 3.1.2 Avoiding problematic race conditions
有几种方式解决race condition：

最简单的方式是wrap data structure with a protection mechanism，确保只有一个thread可以在不变式失效的中间状态下执行修改。
C++ STL提供了几种机制，本文后面详述。

另一种方式是modify the design of your data structure and its invariants
使得修改操作是一系列不可分的步骤，以保护不变式。
这也通常称为无锁编程(lock-free programming)且容易出问题，
如果达到这层级别需要对内存模型、thread等了若指掌，十分复杂。
内存模型将在part5详述、无锁编程将在part7详述。

第三种方式是将data structure的update handler视作transaction(协议，比方说TCP)。
那么将一系列数据修改、读取操作打包成一个transaction log并用一个单步发送出去，
如果因为其他线程的原因提交失败，那么transaction再次发送。这称为software transactional memory(STM)
这是一个现在较活跃的领域，但C++并不直接支持STM。


## 3.2 Protecting shared data with mutexes
mutexes是C++中最常见的数据保护方式，
通过lock、unlock保证一次只有一个thread操作需要保护的数据。其他thread想要进入则需等待。

mutexes同时也需要注意几个问题：

* protect the right data (3.2.2)
* avoid race conditions inherent in your interface(3.2.3)
* deadlock(3.2.4)
* protecting either too much or too little data(3.2.8)

### 3.2.1 Using mutexes in C++


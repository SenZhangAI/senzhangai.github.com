---
layout: post
title: "C++并发编程 part1"
description: "《C++ Concurrency in Action》阅读笔记"
keywords: C++, concurrency, C++11, Thread, Mutex
category: C++
tags: [C++]
---

## 前言
这是关于《C++ Concurrency in Action》的学习笔记，
该书作者是Boost::Thread库的主要贡献者。
书中主要介绍了C++11对并发编程的支持。
强烈建议学习。此外，该书的中文翻译很烂。

## Hello word of concurrency in C++
引言：C++11新标准对多线程的支持使得编写多线程C++不再需要依赖于平台的特性扩展。
因此更有利于可移植性。

## 1.1 What is concurrency？
简单来说，同一时间做多个事情就是并发

### 1.1.1 Concurrency in computer systems
最早期的单核计算机，通过任务切换(task swiching)实现并发。
看上去就像同一时间可以运行不同程序那样。
实际上，通过上下文切换(context swich)让不同程序分期占用CPU，
只不过运算较快，看上去就像并发一样。(需要注意的是切换过程中也是需要时间的)
当然这也能实现加速，因为有的程序可能到了某个计算步后需要等待，
这时候切换其他任务则避免CPU闲置。

而多核或者多线程则真正意义上实现了硬件并发(hardware concurrency)

### 1.1.2 Approaches to concurrency
两种方法，multiple processes或者multiple threads

#### multiple processes
缺点：multiple processes缺点是复杂而且慢。
缺点-原因：复杂在于系统隔离了不同进程，不能共享数据等，
只能通过signals, sockets, files, pipes等方式。
这也是慢的一个原因，另一个原因是系统启动进程、切换进程也很耗时。

优点：容易保证并发安全; 适用于异构集群(例如MPI就是通过网络通信)

#### multiple threads
特点-数据共享：所有同一process的线程都共享地址空间，绝大多数数据可直接访问。
相比较而且，不同processes虽说也可实现一定的地址空间共享，但较复杂。
所以threads并发的overhead比process并发的overhead小
(当然，threads并发也较不安全，需要mutex之类的保护)

C++支持线程并发而不是多进程并发：
需要说明的是C++并不在语法上支持process级并发，process级并发依赖于平台APIs，
C++中的并发指的是threads并发。

## 1.2 Why use concurrency？
### 1.2.1 using concurrency for separation of concerns
举例说明: 一个音乐播放器除了管理、播放音乐文件外，还要随时等待用户输入，比如暂停，开始etc
这实际上应该是一个程序的两个子模块之间的交互，如果不用并发，
可行的方式是比如一个程序接收用户指令后通过中断信号来控制另一个播放程序，很麻烦。

这种分离任务式的并发，考虑的是逻辑子模块的数目，而不是CPU核数。

### 1.2.2 using concurrency for performance
两种方式，
一种是task parallelism以及data parallelism
另一种是用同一程序处理不同数据，例如开好几个fluent算不同算例。

### 1.2.3 When not to use concurrency
不用并发的终极理由是其带来的收益比不上付出。

毕竟多线程编程比单线程麻烦一点，
且并不是thread越多就越快。

通常考虑将并发应用于performance-critical的地方。

## 1.3 Concurrency and multithreading in C++
### 1.3.1 History of multithreading in C++
早期C++标准并没有考虑到并行，
这导致后来各编译器供应商提供自己的语法扩展，例如POSIX C标准和Windows API
且很少compiler供应商提供一个正规的multithreading-aware memory model。

以上都是C的API，为了更好地给C++程序员提供面向对象的多线程支持。
比如MFC、Boost、ACE等应用框架也封装了底层API，提供应用抽象层。
尤其RAII惯用法，使得比如mutex等较容易实现。

### 1.3.2 Concurrency support in the new standard
这么重要的地方，新的C++标准当然不能放过，
1. C++11不只是提供brand-new thread-aware memory model
2. STL也提供各种管理thread的类。
3. 还有protecting shared data
4. synchronizing operations between threads
5. low-level atomic operations

对于原子操作的直接支持，使得C++可以用语法写出高效代码，而不需针对平台的汇编。

### 1.3.3 Efficiency in the C++ Thread Library
简单来说，STL的设计上是非常注重效率的，
更建议用STL而不是重新造轮子。
如果STL慢，极有可能是用法上的问题，而不是STL本身的问题。

### 1.3.4 Platform-specific facilities
有些情况是STL没有提供，而必须用针对特定平台实现的。
那么这时候可以用STL专门为此保留的`native_handle()`。
这样也可以兼容到STL，避免为此而重新造轮子。

### 1.4 一个简单的helloworld程序
略，示例代码如下：

```cpp
#include <iostream>
#include <thread>

void hello() {
    std::cout << "Hello World\n";
}

int main() {
    std::thread t(hello); //生成新的线程t后main继续向后执行
    t.join(); //告诉main函数等待t执行完
    return 0;
}
```

需要说明的是，单线程程序始于main，而并发程序始于构建`std::thread`对象之时。
而多线程中主线程（此处为main）并不会等待子线程结束，而是生成thread对象后立马往后执行。
此时，`join()`函数的作用就是告诉main函数等待，否则可能子函数还没执行，main就结束了。

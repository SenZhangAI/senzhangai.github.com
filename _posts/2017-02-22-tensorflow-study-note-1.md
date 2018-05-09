---
layout: post
title: "tensorflow 代码构架学习笔记 1"
description: ""
keywords:
category: Programming
tags: [tensorflow]
---

## 前言
本文是阅读tensorflow源码的过程中，记录tensorflow的设计思路。内容并非关于如何用tensorflow，又或者是其他跟人工智能相关的专题。

本文研究的版本为tensorflow原始的版本，为2015-11-07-08:27的提交版本，哈希值前7位为**f41959c**

## 底层设计
代码设计第一步是平台无关，通常放在例如`flatform`这样的文件夹下，

tensorflow放在`core/platform`中

### 1. 定义整型
放在`integral_types.h`头文件中，tensorflow用的是比较常见的命名规则，比如uint64,int8之类。

tensorflow定义的方式比较简单，比如`typedef int int32`，这就必须保证int是32位的了，所以编译器或者平台会有这方面的要求，不过我觉得这种不含糊的定义方式才舒服。免得写一大堆。
按照C标准那种int没给明确位数简直让人难受。

然后定义各个整型的极大值，极小值，为什么放在`integral_types.h`而要放在`port.h`中这个我没理解。

### 2. 定义mutex
默认的实现很简单，就是用了stl，包括`<chrono>`，`<condition_variable>`，`mutex`。google应该也模仿stl的接口实现了自己的库，

用宏来条件选择include那个版本，google版本还是stl版本

### 3. 确定大端-小端
这里简单地定义一个bool常量`kLittleEndian = true`，TODO，留待以后实现。

### 4. 定义其他某些跟平台有关的函数
这里暂时没给出实现，但是注释到时很齐全，主要包括一下内容：

```cpp
string Hostname(); //程序运行的主机名
int NumScheduableCPUs(); //估计可被调配的CPU个数，通常在运行中不变，但也有可能有cluster(集群)管理软件动态调整
void AdjustFilenameForLogging(string* filename); //有些平台对于文件名有特殊要求，所以用此函数调整文件名
void* aligned_malloc(size_t size, int minimum_alignment);//对齐申请-释放动态内存
void aligned_free(void* aligned_memory)

enum PrefetchHint {
PREFETCH_HINT_T0 = 3,  // More temporal locality
PREFETCH_HINT_T1 = 2,
PREFETCH_HINT_T2 = 1,  // Less temporal locality
PREFETCH_HINT_NTA = 0  // No temporal locality
};
template <PrefetchHint hint>
void prefetch(const void* x);

// Snappy compression/decompression support
bool Snappy_Compress(const char* input, size_t length, string* output);

bool Snappy_GetUncompressedLength(const char* input, size_t length,
size_t* result);
bool Snappy_Uncompress(const char* input, size_t length, char* output);
```

### 确定是否支持C++11
这很简单，略。

### 一些编译器的attributes
这个跟编译器有关，详见代码，略。

### 小工具
以下的宏很有意思，用一个宏就搞定了禁止复制和拷贝行为，很有用！

只不过感觉此宏应该在另外的地方定义才对。

```cpp
// A macro to disallow the copy constructor and operator= functions
// This is usually placed in the private: declarations for a class.
#define TF_DISALLOW_COPY_AND_ASSIGN(TypeName) \
TypeName(const TypeName&) = delete;         \
void operator=(const TypeName&) = delete
```

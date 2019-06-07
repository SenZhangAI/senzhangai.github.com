---
layout: post
title: "redis中的数据结构"
description: "data structure in redis"
keywords: redis
category: Programming
tags: [redis, c]
---

## 数据结构

### 字符串

首先思考传统C字符串有哪些问题？

1. 查询字符串长度低效，为O(n)
2. 缓冲区溢出容易造成安全问题
3. 以`\0`作为字符串的结束符，那么字符串只能是ASCII码，限制了应用范围。

Redis的解决思路类似于动态数组，结构体中加入`len`以及`alloc`两个成员变量。

基于动态数组的方案，也实现了**空间预分配**和**惰性空间释放**两种优化策略。

**空间预分配**就是多分配一些空间，动态数组的常规操作，当修改后的字符串长于原字符串时，有一定可能性不需要重新分配空间。

**惰性空间释放**也是动态数组的常规操作，即空间减少时不立即换内存。

针对问题3，不再以`\0`作为结尾判断条件，这样可用于利于视频，音频等多种格式而不受影响，是二进制安全的(binary-safe)

那么再思考一个问题，保存实际数据的buffer，是否需要兼容C字符串风格以`\0`结束？

Redis的做法是兼容C字符串，虽然多了一个字节，但好处是可以直接利用C的`<string.h>`函数库

### 链表

C没有内置链表，需要自己实现，链表在Redis中使用广泛，例如`链表键`,`发布与订阅`,`慢查询`,`监视器`等。

Redis中链表节点属于常规操作了，是一个双端链表

```c
typedef struct listNode {
    struct listNode* prev;
    struct listNode* next;
    void* value;
} listNode;
```

而对于整个链表的操作，是由一个list持有各个listNode节点:

```c
typedef struct list {
    listNode* head;
    listNode* tail;
    void* (*dup)(void* ptr);
    void (free)(void * ptr);
    int (*match)(void* ptr, void* key);
    unsigned long len;
} list;
```

链表节点用`void*`保存不同数据类型，那么就可以针对不同类型用特定的拷贝，释放函数，
所以list中包含了三个函数指针。

此外Redis还实现了一个`listIter`的数据结构，用于辅助遍历：

```c
typedef struct listIter {
    listNode* next;
    int direction;
}
```

### 字典
又称为符号表(symbol table)、关联数组(associative array)或map。

C中也没有提供标准的map，需要自己实现。
Redis的数据库是使用字典实现，对数据库的增删查改都是对字典的操作。

Redis字典采用hash表实现，常规的Hash表应该是桶+链表来实现。

首先看看字典最基础的元素：

```c
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

`dictEntry`就包含了key, v以及链表下一个节点next

因为key和v都有`void* val`这种多态类型，
也需要指定该类型对应的特性函数，例如指定该类型hash值怎么计算而来(如下hashFunction函数指针)，如何销毁，如何比较，如何复制。

值得注意的是没有`dictEntry`中没有一个枚举Tag来说明`union v`中到底是何种类型，这个先大胆猜测一下，应该Hash表的`hashth`这一个上层的数据结构中表示，
由或许以`hash`作为模板，特化为内置的`intHash`，`doubleHash`的不同数据结构。

```c
typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;
```

在看看`dictht`数据结构，`dictEntry** table`是表指针，为什么是指针的指针呢？
因为我们桶+链表实现的字典基础元素不是`dictEntry`而是`dictEntry*`，

```c
/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask; // 用于计算索引值，sizemask 恒等于 size - 1, size一定是2^n
    unsigned long used;
} dictht;
```

在看看最顶层的`dict`数据结构
```c
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

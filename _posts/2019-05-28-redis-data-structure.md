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
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

链表节点用`void*`保存不同数据类型，那么就可以针对不同类型用特定的拷贝，释放函数，
所以list中包含了三个函数指针。

此外Redis还实现了一个`listIter`的数据结构，用于辅助遍历：

```c
typedef struct listIter {
    listNode *next;
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

值得注意的是`dictEntry`中没有一个枚举Tag来说明`union v`中到底是何种类型，这个先大胆猜测一下，应该Hash表的`hashth`这一个上层的数据结构中用类似多态函数表的方式来特化不同dict，而从顶层dict的角度没必要知道其具体类型，看代码发现是这种方式，

另一种可能是以`hash`作为模板，特化为内置的`intHash`，`doubleHash`的不同数据结构，最后看代码，发现Redis源码采用第一种方式，这是因为如果特化为具体类型以后，需要针对每种类型写特定的函数(函数签名决定了`doSomething(intHash* h)`函数不能传入`doubleHash*`类型，所以得写多个函数)。

如下及实现不同类型的多态函数：

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
因为我们桶+链表实现的字典基础元素不是`dictEntry`而是`dictEntry *`，

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

再看看最顶层的`dict`数据结构:

```c
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

其中`dictType *type`用于支持不同类型的多态函数，`void *privdata`是不同类型多态函数的特定参数。

#### 分析

dict有两个重要的内容

1. 尽可能做到perfect hash，即尽可能少出现哈希冲突(hash collision)，这就跟`hashFunction`有关。

2. **扩容**，如果当前的hash桶太小，哈希冲突的概率增大时需要扩容。

扩容需要考虑以下几个问题:

* hash扩容的时机选择，hash在怎么时候扩容较优，

* hash扩容过程中如何保证访问正确性，直觉上感觉应将扩容期尽可能缩短。

#### Hash算法

旧版Redis采用[MurmurHash2](https://github.com/aappleby/smhasher)算法,该算法优点是即使输入的key有规律，依然有较好的随机分布性，且计算速度快。

当前最新版Redis采用`SipHash`算法。

#### 扩容Rehash算法

Redis采用的方式是用两个hash表`dictht ht[2]`，`ht[0]`专门用于常规的访问，
`ht[1]`专门用在扩容的时候，扩容结束时交换`ht[0]`和`ht[1]`。

这样就能保证扩容时只需要管`ht[1]`，对于`ht[0]`数据索引位置不需要更改，即可正确访问到，
扩容期访问数据我估计是先访问`ht[0]`，没找到再找`ht[1]`。

实际如果是添加`key-value`，扩容期间添加操作一定添加到`ht[1]`中，而涉及到查找操作，则两个`ht`都要找

`ht[0]`和`ht[1]`的桶尺寸不一样，所以索引mask不一样，需要区分开。
正好关于桶尺寸的参数在`dictht`数据结构中(`dictht->size`和`dictht->sizemask`)

这里有个问题，扩容结束后`ht[0]`与`ht[1]`交换的那一瞬间，应该有一个互斥锁才对，但`dict`数据结构中并没有一个`lock`私有变量，待进一步分析(再看一遍代码发现是用`rehashidx`记录rehash进度，实现的互斥)。

第二个子问题，扩容时`ht[1]`数据执行次数越少越好，越快进入常规状态越好。我之所以这样考虑是因为看代码扩容期间`ht[0] ht[1]`都要查找，觉得效率不高，但实际代码并非为了快速进入常规状态，
而是经常执行`_dictRehashStep(dict* d)`函数，每次只执行一个单元格，小步前进。而实际上如果数据在`ht[1]`中，`ht[0]`数据为`NULL`，耗时很少，对性能影响不大。

第三个子问题，如果在rehash的时候插入新数据，应该插入`ht[1]`(猜对了)。

第二、三子问题跟rehash的算法非常相关，可以说rehash算法才是这个dict的精髓

##### rehash算法
实际的rehash算法如上所述，通过`ht[0] ht[1]` 以及用于判断rehash进度的`rehashidx`很好地保证了dick表rehash过程中的稳定。

---
layout: post
title: "C++填坑系列：tip7 为多态基类声明virtual析构函数"
description: "C++填坑，主要来自effective c++"
keywords: c++, cpp, effective
category: c++
tags: [c++]
---

### 问题的产生

1 derived 类的内存顺序大概是:

     [ base member ] [ base vptr ] [ derived class member ] [ derived vptr ]

2 对于**多态**，实现方法主要是：

```cpp
BaseClass* pbase = new DerivedClass();

pbase -> DerivedClass.doSomething();

delete pbase;
```

如果delete调用了baseClass的non-virtual析构函数，
根据析构函数的判断，数据长度应该是`sizeof(baseClass)`，那么heap中的数据现在是：

                               ↓ derived class 部分未被释放掉 ↓
     [ 被delete了的部分 ] [ derived class member ] [ derived vptr ]

**因此出现了内存泄漏**

所以，要让编译器明白，此处用到了**多态**，即`base class pointer`指向了`derived class`，析构就调用`derived class各自的析构函数`吧，因此base class 用`virtual 析构`。

### virtual函数与多态的关系

对于多态基类有一个特点，就是一定会有virtual函数，不然怎么实现多态？
因此，也可以认为含有virtual成员函数的基类需要使用virtual析构函数。
当然本质而言，是因为多态下基类指针指向了子类的原因。

### virtual函数带来的类体积、可移植性的问题

此外需要说明的是virtual函数会带来对象体积的增加，对于C++的C style部分需要注意，因为加上了虚表指针vptr的类无法与C style数据结构对应。因此在于C交互的时候需要注意，否则不再有可移植性。

### 当心

例如string类其实不含虚析构的，因此千万不要定义一个类继承自string。
需要的时候查看下lib的源代码判断下即可。

其他不带vitual析构的包括STL的vector、list、set、tr1::unordered_map等。

### pure virtual析构函数

所谓pure virtual函数是指包含该函数的类不能拥有实体instance，即**抽象类**。

因抽象类总试图作为base class，如果没有其他函数适合作为virtual函数的话，
可以将析构函数作为纯虚函数。此外，该`pure virtual dtor`需要提供定义。

#### 示例

```cpp
//参考自 Effective cpp
class AbstractVirtual {
public:
  virtual ~AbstractVirtual() = 0;
};

AbstractVirtual::~AbstractVirtual() { } 
//需要提供定义。因为derived class会调用base class的析构函数，不定义将出现linkage error
```




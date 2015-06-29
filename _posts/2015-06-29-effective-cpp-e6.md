---
layout: post
title: "C++填坑系列：tip6 如何显式拒绝编译器自动生成的函数"
description: "C++填坑，主要来自effective c++"
keywords: c++, cpp, effective
category: c++
tags: [c++]
---

### 概述

方法很简单，分两步：

1.声明并放在private中

2.不去定义它，那么即便member函数或者friend函数调用也将会出现linkage error。

### 复习private,public,protected

参考自 <http://blog.chinaunix.net/uid-722885-id-124903.html>

第一:private,public,protected的访问范围:
 
private: 只能由该类中的函数、其友元函数访问,不能被任何其他访问，该类的对象也不能访问. 
protected: 可以被该类中的函数、子类的函数、以及其友元函数访问,但不能被该类的对象访问 
public: 可以被该类中的函数、子类的函数、其友元函数访问,也可以由该类的对象访问
注：友元函数包括两种：设为友元的全局函数，设为友元类中的成员函数

第二:类的继承后方法属性变化:

使用private继承,父类的所有方法在子类中变为private; 
使用protected继承,父类的protected和public方法在子类中变为protected,private方法不变; 
使用public继承,父类中的方法属性不发生改变;
 
 
水平访问：声明了某类的一个对象，访问其成员函数和数据成员；一般只有public区（不含protected区!）的成员函数和数据成员，可以被水平访问。
垂直访问：一个类从某个基类派生，派生类访问基类的成员函数和数据成员；一般只有public和protected区的成员函数和数据成员，可以被垂直访问。
再次提到：可以提供访问行为的主语为“函数”。类体内的访问没有访问限制一说，即private函数可以访问public/protected/private成员函数或数据成员，同理，protected函数，public函数也可以任意访问该类体中定义的成员public继承下，基类中的public和protected成员继承为该子类的public和protected成员（成员函数或数据成员），然后访问仍然按类内的无限制访问。

对于类域范围内，成员函数访问无所谓访问限制。对于继承情况下的基类private成员，在派生类中仍然继承了下来，只不过它不能直接访问,即使在类里也不行,更不用说是类对象了。

### 示例 GCC iostream

```cpp
//文件： ios_base.h
  class ios_base
  {
  public:
  // 省略

  // _GLIBCXX_RESOLVE_LIB_DEFECTS
  // 50.  Copy constructor and assignment operator of ios_base
  private:
    ios_base(const ios_base&);  // 阻止copy 构造函数

    ios_base&
    operator=(const ios_base&); // 阻止copy assignment操作符
  };
```

```cpp
//文件： basic_ios.h
template<typename _CharT, typename _Traits>
    class basic_ios : public ios_base  //继承自ios_base
    { //省略 };
```

```cpp
//文件： ostream
template<typename _CharT, typename _Traits>
    class basic_ostream : virtual public basic_ios<_CharT, _Traits>
    { //省略 };

template <typename _CharT, typename _Traits>
    class basic_ostream<_CharT, _Traits>::sentry 
    //class basic_ostream中的类sentry
    { //省略 };
// ...
```

```cpp
//文件： istream
template<typename _CharT, typename _Traits>
    class basic_istream : virtual public basic_ios<_CharT, _Traits>
    { //省略 };
```

### 示例 Uncopyable class

``` cpp
// 代码来自Effective C++
class Uncopyable {
protected:    //允许derived class调用构造和析构
  Uncopyable() {}
  ~Uncopyable() {}
private:
  Uncopyable(const Uncopyable&);  //阻止copying
  Uncopyable operator= (const Uncopyable&);
};

class HomeforSale : private Uncopyable { 
                   //这种继承实现的方式有可能带来多重继承的问题
    // ...
};
```


### RAII

RAII（resource acquisition is initialization，资源获取即初始化）是一个C++的技巧，或者说是属于C++的设计模式，跟本主题有一定的关系，所以稍微补充一下。

所谓资源获取即初始化，我觉得可以理解为**通过类初始化来获取资源**，因为类生命周期结束后即会通过析构函数自动释放掉资源，而无需考虑资源释放的问题，这就很爽了。

例如智能指针就是将指针封装成类的RAII技巧。

既然是资源，那就具有唯一性，**大多数情况下不能被复制**，所以就需要如上所述的uncopyable 技巧，要么自己定义，要么也可以直接用

关于RAII这里不详细展开了，
更多参见： [【C++设计技巧】C++中的RAII机制](http://www.cnblogs.com/gnuhpc/archive/2012/12/04/2802307.html)

           <http://www.jellythink.com/archives/101>

           <http://blog.csdn.net/hunter8777/article/details/6327704>

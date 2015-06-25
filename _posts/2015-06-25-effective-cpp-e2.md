---
layout: post
title: "C++填坑系列：tip2 尽量用const，enum，inline代替 #define"
description: "C++填坑，主要来自effective c++"
keywords: c++, cpp, effective
category: c++
tags: [c++]
---

## tip2 尽量用const，enum，inline代替 #define

### 优点：

1. 出错信息是Pi而不是宏定义的3.1415926，这是由于宏定义的标记会被预处理器移走，无法出现在记号表（symbol table）中。
2. 对于浮点常量，可能比使用宏导致较小的代码量。
3. 最大的好处其实不在于类型检测，而是宏没有自己的namespace，而const有。
4. #define宏由于没有类型，对于需要函数重载的情况则不适用。同时const也是类型安全的。
5. 编译效率更快。例如如下情况，编译器每次遇到AREA都要求值，而Area只求值一次。

```cpp
#define AREA (12*15)

const float Area = 12*15; //编译更快
```

对于宏而言，优势不在于定义常量，而是在于定义一些快捷的语法，以及不同编译选项。

### const 常数

#### 例子：

```cpp
#define PI 3.1415926
```

改为：

```cpp
const double Pi = 3.1415926
```



#### 补充 关于extern 与static

定义常量第一想法是用`extern`，然而，**如果不需要强调共享数据的话，不用extern更好**。
而是把该常量直接放在.h文件中。

这是因为，对于const常量，编译器优化通常可以把它优化掉（跟inline一样），除非你调用了该常量的地址，例如`cout << &Pi`。

而对于`extern`编译器不得不为其分配地址，并取址调用。然而，如果const对象是复杂的类，用extern更好一点。

如果const不加任何其他修饰，则默认为static。

参见 <http://blog.csdn.net/wxl1986622/article/details/8214903>

### 字符串常量

对于字符串常量，例如：

```cpp
#define AUTHOR_NAME_STR "Sen"
//或者如下定义
#define _STR(s) #s
#define STR(s) STR(s) //该函数主要用在STR(__LINE__)等场合
#define AUTHOR_NAME Sen
//使用方法 cout << STR(AUTHOR_NAME)

const char* const authorName_cstr = "Sen";
//或者
const std::string authorName_str("Sen");//更好

```

### enum hack

即用`enum{max = 10};`代替`const int max_int = 10;`, 这样有以下好处：

enum强制不允许取地址（有点像#define），这样不论编译器是否优秀都能足够优化。

enum hack是模板元编程的基础技术。

### 宏用inline替换

```cpp

#define MAX(a, b) ( (a) > (b) ? (a) : (b) ) //各种带括号
//且面对MAX(++m, n)这种会出问题
//如m > n, m被累加2次
//如m < n, m被累加1次
//归根结底这不是真正的函数

//解决办法，用如下inline template代替
template<typename T>
inline T Max_Temp(const T& a, const T& b) {
    return (a > b ? a : b);
};
```

### 小结

1. 对于单纯常量，最好用const或者enum代替#define
2. 对于宏macro，最好用inline tempelate代替

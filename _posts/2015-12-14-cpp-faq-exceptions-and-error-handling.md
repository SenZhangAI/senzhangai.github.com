---
layout: post
title: "C++ Faq: Exceptions and Error Handling"
description: "来自isocpp官网的FAQ专题"
keywords: c++, cpp, Exceptions, Error Handling
category: c++
tags: [c++, exception]
---


## 前言
本章内容主要翻译自：

<https://isocpp.org/wiki/faq/exceptions>

这对于异常讲解地比较全面，以此为契机更系统地了解C++异常

## Exceptions and Error Handling

### Why use Exceptions
或者为何不用"good old errno(错误码) and if-statements"?

1. 基本原因是因为错误码的设计方式会跟正常的代码混在一起。
试想如果要写库(library)，库的作者知道怎么抛出错误，却无法判断使用者怎么处理错误，
而库使用者知道自己想要如何处理错误，但却不知道作者抛出了错误。
2. 有些情况只能用exception解决，例如构造函数中发生的错误：

#### 构造函数中发生错误的例子
这是RAII基本的不要解决的问题，
一个构造函数的职责是构建class的**不变式**（为成员函数创造环境）
而这通常涉及到资源的获取，资源例如：memory, locks, files, sockets etc.
而获取资源过程中经常可能会遇到请求失败的现象，例如：

```cpp
vector<double> v(100000);   //needs to allocate memory
ofstream os("myfile");      //needs to open a file
```

如果不使用异常，首先我们得需要一个变量来表示请求资源失败的bad state，
那么错误处理类似于如下方式：

```cpp
vector<double> v(100000);   //needs to allocate memory
if(v.bad()){
    /* handle error */
}
ofstream os("myfile");      //needs to open a file
if(os.bad()){
    /* handle error */
}
```

这样vector还得多保存一个错误的状态变量和相应的成员函数。
而对于没有设置错误码的class就无能为力了、
且与常见代码混在一起。

以上可以看出构造函数中的错误很难不用异常解决，
那么对于Plain Old Function呢？
如果不用异常，解决办法是return an error code or
set a non-local variable(e.g. errno)

设置全局变量(set a non-local variable)通常不是一个好主意，例如在检测错误码前被其他函数改变了值，
除非立即检测错误码。
更不用谈多线程情况下访问这个全局变量的情况了。。。

如果是return an error code，那么返回值必须是一个非正常值，
例如，对于`abs()`函数，显然返回`-1`等负数就不是正常值，可以表示错误，
但对于函数正常的返回值已经占满整个int空间的情况呢？解决办法是返回一对数pair而不是int之类。
这个其中一个表示错误信息，一个表示正常返回值。
但这样就很麻烦了。
而且传统错误处理不仅带来麻烦的问题，**而且还不安全**。
试想使用者万一忘了检测并处理错误呢？
那么代码会按照“正确”的代码继续执行。

#### 通常反对使用异常的观点

1. exceptions are expensive!
实际上现代C++的实现已经异常的负担降低到只比no error handling高3%
而error-return-test方式也有开销。
实际上，如果代码不抛出异常，几乎没有多余代价。

2. JSF++中 Stroustrup本人就禁用异常
JSF++对实时性和安全性要求极高的应用（飞机控制系统），实际上JSF++也期望往后用上异常

3. 在new 构造函数时抛出异常将会导致内存泄露
这是很古老的一个编译器错误，早已经修正

### How do I use exceptions?
主要参考《The C++ Programming Language》关于异常的章节，以及Appendix E。
这里只给出两个小例子：

#### 例子1 构造函数获取资源时抛出异常的方式

```cpp
class VectorInSpecialMemory {
    int size;
    int* elem;
public:
    VectorInSpecialMemory(int s)
        : size(s)
        , elem(AllocateInSpecialMemory(s))  //注意请求内存可能会失败
    {
        if (elem == nullptr)
            throw std::bad_alloc();         //申请内存失败则抛出异常
    }
    // etc.
};
```

#### 例子2 函数中的RAII惯用法
对于通常不安全的做法是：

```cpp
void old_func(const char* s) {
    FILE *f = fopen(s, "r");
    // use f
    fclose(f); //close the file
}
```

如果在`use f`中抛出异常，file则没有被关闭，正确做法是RAII：

```cpp
void func(string s) {
    File_handle f(s, "r");  //RAII
    // use f

    // call File_handle destruction to close the file
}
```

因为异常发生后，发生点后的代码不会继续执行，但生成的临时变量生命期结束，
会调用其析构函数，那么在析构函数中`fclose(f)`就是异常安全的。

### What shouldn't I use exceptions for?
首先要明确的是C++异常的设计是用来支持**error handling**的。

有以下建议：

1. Use Throw only to signal an error
2. Use catch only to specify error handling actions when you know you can handles an error(或者也可用于转发更详细的Exception信息，re-throw)
3. **Do not** use throw to indicate a coding error in usage of a function.
不要用Exception处理程序员编程上犯的错误，而是用assert或者其他机制send process to a debugger or to crash the process and collect the crash dump for the developer to debug.
4. **Do not** use throw if you discover unexcepted violation of an invariant of your component.
如果发现不变量不成立，不要用Exception，而是assert或者其他机制终止程序。Exception 无法解决这个错误，也许还会进一步污染其他用户数据。

在其他编程语言中，Exception有其他用法，但这些不是C++的惯用法(idiomatic), 且C++也故意没有对其提供良好实现。

特别的，Exception别用在control flow，虽然跟return很像，但会带来低效与混淆。同样，也别用于getting out of a loop

### What are some ways try/catch/throw can improve software quality?
从软件工程的角度，try-catch-throw如何提供软件质量？

以传统的return error做法作为对比，
传统做法错误处理会多一个或者多个if-statements，而这会带来：

1. Degrade quality： 条件分支是错误发生的重灾区，比其他地方出错概率大，约10倍。减少条件分支能带来更robust的代码。
2. Slow down time-to-market： 白盒测试一个重要的地方是分支点，增加的条件分支带来更多测试点。推向市场的时间就延长了，
减少测试的话，错误出现在user那边更是灾难。
3. Increase development cost：正如上所述，更多的测试，更多的Bug finding，bug fixing使得开发流程更复杂。

当然也要考虑实际情况，如果团队对try-catch-throw不熟悉，不能硬上，可以在小项目中练练手。

### More about exceptions and return-codes
I'm still not convinced: a 4-line code snippet shows that return-codes aren't any worse than Exceptions; why should I therefore use exceptions on an application that is orders of magnitude larger?
我仍然无法信服，我的一个4行代码算例显示return-codes并不比Exceptions差...

对于一个toy program当然看不出来，让我们来到real-world program：

the code that detect a problem must typically **propagate** error information back to a different function that will handle the problem. This propagation often needs to go through dozens of functions.
错误传播到错误处理程序常常需要经过多个中间调用的程序:

```cpp
    void f1()
    {
      try {
        // ...
        f2();
        // ...
      } catch (some_exception& e) {
        // ...code that handles the error...
      }
    }
    void f2() { ...; f3(); ...; }
    void f3() { ...; f4(); ...; }
    void f4() { ...; f5(); ...; }
    void f5() { ...; f6(); ...; }
    void f6() { ...; f7(); ...; }
    void f7() { ...; f8(); ...; }
    void f8() { ...; f9(); ...; }
    void f9() { ...; f10(); ...; }
    void f10()
    {
      // ...
      if ( /*...some error condition...*/ )
        throw some_exception();
      // ...
    }
```

用return-codes试试？

而try-catch-throw这样upwind的类似“回到过去”的能力使得问题变得so-easy。
而且这样return-codes还得设计出复杂的错误代码来表示本错误不是本函数发生而是传播过来的错误。

### How do exceptions simplify my function return type and parameter types?
如果用return-codes，假如出错信息类型不只一个而是多个，那么处理错误将变得复杂且混在一起。

一个简单的例子：

```cpp
class Number {
public:
    friend Number operator+(const Number& x, const Number& y);
    friend Number operator-(const Number& x, const Number& y);
    friend Number operator*(const Number& x, const Number& y);
    friend Number operator/(const Number& x, const Number& y);
    // ...
};

void f(Number x, Number y) {
    // ...
    Number sum  = x + y;
    Number diff = x - y;
    Number prod = x * y;
    Number quot = x / y;
    // ...
}
```

以上函数有可能会出现异常，包括overflow, divide-by-zero, underflow等。

如果使用异常处理就会较为简单，比如：

```cpp
void f(Number x, Number y) {
    try {
        // ...
        Number sum  = x + y;
        Number diff = x - y;
        Number prod = x * y;
        Number quot = x / y;
        // ...
    }
    catch (Number::Overflow& exception) {
        // ... code that handles overflow ...
    }
    catch (Number::Underflow& exception) {
        // ... code that handles underflow ...
    }
    catch (Number::DivideByZero& exception) {
        // ... code that handles divide-by-zero ...
    }
}
```

这样只需要声明几个不同的异常，并且在出错的地方throw即可。
如果是用return-codes的方式，又该怎样处理呢？

```cpp
class Number {
public:
    enum ReturnCode {
        Success,
        Overflow,
        Underflow,
        DivideByZero
    };
    Number add(const Number& y, ReturnCode& rc) const;
    Number sub(const Number& y, ReturnCode& rc) const;
    Number mul(const Number& y, ReturnCode& rc) const;
    Number div(const Number& y, ReturnCode& rc) const;
    // ...
};
```

1. 设计一个数据结构来表征正确返回及出错信息，如上用了内部匿名枚举表征。
2. 还可以注意上述函数必须多一个引用参数处理异常(`ReturnCode& rc`), 函数得再嵌套一层。

*注意* 设计异常处理的数据结构时千万别放在namespace-scope, global or static变量中去，
因为这不是线程安全的。

不只类设计变得复杂，原先的函数将变成：

```cpp
int f(Number x, Number y) {
    // ...
    Number::ReturnCode rc;
    Number sum = x.add(y, rc);
    if (rc == Number::Overflow) {
        // ...code that handles overflow...
        return -1;
    } else if (rc == Number::Underflow) {
        // ...code that handles underflow...
        return -1;
    } else if (rc == Number::DivideByZero) {
        // ...code that handles divide-by-zero...
        return -1;
    }
    Number diff = x.sub(y, rc);
    if (rc == Number::Overflow) {
        // ...code that handles overflow...
        return -1;
    } else if (rc == Number::Underflow) {
        // ...code that handles underflow...
        return -1;
    } else if (rc == Number::DivideByZero) {
        // ...code that handles divide-by-zero...
        return -1;
    }
    Number prod = x.mul(y, rc);
    if (rc == Number::Overflow) {
        // ...code that handles overflow...
        return -1;
    } else if (rc == Number::Underflow) {
        // ...code that handles underflow...
        return -1;
    } else if (rc == Number::DivideByZero) {
        // ...code that handles divide-by-zero...
        return -1;
    }
    Number quot = x.div(y, rc);
    if (rc == Number::Overflow) {
        // ...code that handles overflow...
        return -1;
    } else if (rc == Number::Underflow) {
        // ...code that handles underflow...
        return -1;
    } else if (rc == Number::DivideByZero) {
        // ...code that handles divide-by-zero...
        return -1;
    }
    // ...
}
```

需要说明的是比如加法并不会返回一个divide-by-zero异常，但这里只是示意性的，有些情况还是会返回一些同类型的异常。

以上函数可以看出来，一方面复杂度增加了，参数也多了一个引用类型，rc表征异常。
同时，每次调用一个函数后都得**立刻**检测并处理异常，错误代码与正确代码混在一起。
在考虑一下如果未来又多了一个类型的异常信息，可不是简单地修改几处代码的问题了。。。

总之，对于return-codes处理异常，当因为需要多余参数表征异常时，return或者函参就会变得复杂。

### What does it mean that exceptions separate the "good path"(or "happy path") from the "bad path"
这里主要说明Exception的另一大好处是能够清晰地在代码中区分“good path”和“bad path”，
这一点正如上节中看到，如果用return-codes方法，因为需要在函数返回后立即处理，看上去很凌乱，
再考虑不同函数抛出同类型异常的情况。。。

### Exception Handling is easy and simple,did I get it right?
Exception handling很好，但并不简单。
首先他需要严格的使用规则，必须学习它。
其次它不是万能药，有些场合，例如考虑到团队素质等因素，是否使用值得商榷。
再次它也不是一招鲜吃遍天，有些场合可能return-codes更合适。
但它确实值得学习。

### Exception handling seems to make my life more difficult,that must mean exception handling itself is bad
Exception handling似乎是我的代码更加麻烦，这是否意味了它不好？

大错特错，
Exception handling是一个工具，如果会用它将威力无穷，但如果抱有错误的认识，则反而会使情况变得糟糕。

如果你发现代码中的try blocks混乱而糟糕，那么很有可能你对异常处理理解错误，
以下列出对于异常的常见误解：
*注意* 不能照本宣科地理解以下内容，知识不是死的，异常也要看场合随机应变

#### 1. The return-codes mindset
常见的return-codes的习惯是return以后马上处理错误，
这样一个函数一个try-catch-throw，使得代码中充斥着大量的try-block。
这就是误把throw当成了一种return，而把catch当成return后的处理函数了。

#### 2. The Java mindset
这一部分前，首先可以先了解一下Java的try-catch-finally机制
参见： [Java解惑之try catch finally](http://javcoder.iteye.com/blog/1131003)
或者其他资料。

简单来说，finally的作用有点像C++中跳出循环后自动执行析构的机制，只不过C++这里有RAII，自动逆序析构资源。
而Java这边则要在finally中手动执行，出错风险增加，不过也更灵活，而且可以在finally中直接return或者throw退出。
还有一点是Java中为了安全逆序释放资源会有嵌套try语句，很难看。
参见： [Java的try-finally] 中最后的代码。

废话说完，以下为正文：

Java中，非内存资源(例如socket、file之类)的回收是依靠**显式**的try-finally blocks
(相对而言C++则是通过RAII、RRID将资源释放操作放在析构函数中，当对象生命期结束时自动释放)
如果将Java的这一思想放在C++中，将会有很多嵌套try-block
(这是因为Java要想安全地逆序释放资源就得这样做，参见[Java的try-finally])
而实际上C++并不需要这样的嵌套try-block，也能安全地逆序释放资源。
这样的嵌套try-block还带来了其他问题，
一个是“good path”和“bad path”交替出现，不干净，容易弄混。
还有一点是RAII将资源释放放在析构中，也就是资源拥有者负责释放，这样资源拥有者更独立，
有利于code reviews和unit-testing。

#### 3. Organizing the exception classes around the physical thrower rather than the logical reason for the throw
以一个banking APP为例，假设其有好几个子系统。
比如出现金额不足的异常，那么正确的思路应该是抛出一个insufficient_funds_exception，这是logical reason。
错误的做法是Foo抛出Foo_exception，Bar抛出Bar_exception，这是physical thrower。
错误的做法将会带来更多的try-catch blocks，而且也会让逻辑变得复杂，例如Foo抛出异常可能要转发给Bar来进一步判断。
In general, exception classes should represent the problem, not the chunk of code that noticed the problem.

#### 4. Using the bits/data within an exception object to differentiate different categories of errors
意思是说不同的Exception应该是设置不同的**type**，而不是在同一个type中设置不同的data来区分，
避免多余的if判断以及避免增加逻辑复杂度。
还是以banking APP为例。
假设要抛出的异常包括 错误的账号，试图清算坏账，基金不足等等。
如果放在同一个Exception Class中，则需要额外地变量区分这些异常，并作出判断。

#### 5. Designing exception classes on a subsystem by subsystem basis
有些人会受到bad old day的常用做法的影响，
古老地做法是将return-codes中codes赋予一个特殊的意义，打个比方说 0代表success，1代表溢出等等。
这样的return结构有一个特点，就是这些变量是跟这个函数耦合在一起的，是一个局部变量，
每个函数都会有自己的magic number。
如果将这样的想法带入到Exception中，将带来无尽的痛苦。
因为这样会带来很多逐层的try-throw-re-throw，而实际上都是一种异常。

Exception的设计应该是system-wide mindset, Exception class不应局限在子系统中，
Exception classes cross subsystem boundaries
—— they are part of the intellectual glue the holds the architecture together（支撑系统构架的粘合剂）

#### 6. Use of ra(as opposed to smart) pointers
这只是诸多non-RAII代码中的一个例子，但因为这个错误太普遍了，所以单独拿出来。
使用raw指针的话，因为要考虑delete，所以会多一个try-catch-re-throw的间接层。

#### 7. Confuseing logical errors with runtime situations
以函数`f(Foo *p)`为例：
通常规定了指针p不能说nullptr，
但对于出现nullptr可能有以下情形：

1. 因为用户使用过程中(runtime)的误操作
2. 该函数的caller，在coding中犯的错误

不同的情况要区别对待，
对于runtime错误，应该抛出一个异常，因为这个不是编码阶段的错误，不能通过code-review排除；
对于coding中犯得失误，用assert，log等等方式告知此bug，
这不是软件本身的设计问题，不要为此改变本身的软件设计，而是让caller调用者知道(此前置条件)。

### I have too many try blocks, what can I do about it?
举例说明：

```cpp
void myCode() {
    try {
        foo();
    }
    catch (FooException& e) {
        // ...
    }

    try {
        bar();
    }
    catch (BarException& e) {
        // ...
    }

    try {
        baz();
    }
    catch (BazException& e) {
        // ...
    }
}
```

这就是典型的受return-codes思想的影响，这并没有发挥Exception的优势。
每当你打算加一个try-block的时候就问一问自己："Why am I using a try block here?"
列举以下可能的原因：

1. 为了handle the exception.
我的catch处理该异常并继续执行而不再传播异常或者错误，
这样，从外层caller的角度，并没看到异常地发生。

这样的思路不错。

2. 为了处理一些异常的中间步骤，然后再re-throw该异常。

如遇到这种情况，可以尝试将这些中间步骤放在dtor中，
例如假设异常处理中间步骤要close a file。那么，可以将这个步骤放在File object的dtor中，也就是RAII。

3. 为了repackage the exception
例如我catch了XyzException，然后提取其中的内容，判断，并throw a PqrException

入遇到这种情况，可以考虑是否能重构一个结构更加优良的Exception objects层次结构，
这样的层次结构不再需要，或者极少需要catch/repackage/re-throw。

4. 其他，以上情况是最常见的，其他看情况

以上内容的核心思路是为自己Why，当明确了自己这样处理的真实目的的情况下，
或许能找到一个更好的方式achieve your goal.

### Can I throw an exception from a constructor? From a destructor?
对于ctor，可以。you SHOULD throw an exception from a constructor
whenever you cannot properly initialize(construct)an object.
恩，是的，C++甚至鼓励ctor出错的情况下抛出异常，详见下一节。

对于dtor，最好别抛出异常，这是C++的基本规则，是异常实现机制决定的
（异常的unwind阶段会调用临时变量的dtor，这时如果dtor又抛出异常。。。）

更多细节参见 [Appendix E:Standard-library Exception Safety](http://stroustrup.com/3rd_safe0.html)

还有要注意的是有些系统是禁用异常的，例如一些实时性要求较高的系统。
比如航空用的JSF

### How can I handle a constructor that fails?
Throw an exception.

首先ctor是没有return的，所以return-codes的方式行不通。
而且抛出异常后内存将被安全释放，而不会出现内存泄露问题，例如：

```cpp
void f() {
    X x;                //if X::X()throws, the memory for x will not leak
    Y* p = new Y();     //if Y::Y()throws, the memory for *p will not leak
}
```

关于ctor阻止内存泄露更多内容参考[how to prevent memory leak if the constructor allocates memory](https://isocpp.org/wiki/faq/exceptions#selfcleaning-members)

还需要补充说明的是使用placement new而不是普通的new的情况下需要注意。

#### How about when I can't use exception

现在假设由于某种情况，你不能使用异常，那么如何处理呢？
次好的方式是用一个状态标志变量将此对象设置为僵尸状态("zombie" state)。
并且每个成员函数都得验证前置条件是否成立，例如`if( !isvalid() ) return`
且每一次构建对象（包括较大的对象以及对象数组）都得用if检测状态变量。
总之很麻烦，但相对其他方法而言还好一些。
所以最好的方法还是throw an exception。

### How can I handles a destructor that fails?
Write a message to a log-file or Terminate the process.
但千万别抛出异常。

如上已述，对于底层的异常处理机制的stack unwinding阶段，所有throw到catch之间阶段的临时对象都得调用dtor释放。
这样才可能有效地、安全地“回到过去”。
那么如果此时dtor又抛出异常。。。C++语法建议此处终止程序。

所以建议never在dtor中抛出异常。
如果抛出异常，要么立即catch，避免扩散到dtor之外，但如果一个错误能在函数内解决，就不需要异常了呀。
或者是有办法区分自己主动抛出的异常，以及unwind阶段的异常分开处理，但这样情况就更为复杂了。

### How should I handle resources if my constructors may throw exceptions?
**Every data member inside your object should clean up its own mess**
也就是说各家自扫门前雪，每个data member解决自己的问题，
这是因为，当ctor抛出异常时，由于object的析构函数dtor不会运行，
那么，扫尾工作就交给各个data member了，例如释放内容，关闭文件等。

比如涉及到内存分配问题，最好不用raw指针，而用smart pointer，例如:

```cpp
#include <memory>

class Fred {
public:
    typedef std::unique_ptr<Fred> Ptr
    // ...
};
```

```cpp
#include "Fred.h"

void f(Fred::Ptr p);
void g() {
    Fred::ptr p2( new Fred() );
    // ...
}
```

### How do I change the string-length of an array of char to prevent memory leaks even if/when someone throws an exceptions?
最重要的一条就是**不要用原始的**`char*` ，取而代之用string之类的class代替。
因为**arrays are evil**。

举个例子：

```cpp
void doSomethingFrom2Source(const char* s1, const char* s2) {
    char* copy = new char[strlen(s1) + 1];      // make a copy
    strcpy(copy, s1);
    // use a try block to prevent memory leaks when exception happend
    // note: we need the try block because we used a "dumb" char* above
    // 这里注意了，char*是用new声明的，new中有throw，所以可以加try-block
    // 为什么不把copy初始化也包含进来呢？还有，为什么需要copy，而不直接用s1?
    try {
        // ... do sth. with copy ...
        char* copy2 = new char[strlen(copy) + strlen(s2) + 1];
        strcpy(copy2, copy);
        strcpy(copy2 + strlen(copy), s2);
        delete[] copy;
        copy = copy2;       //delete后如果不再使用最好赋值NULL,这里是再次使用
        // ... do sth. with copy ...
    }
    catch(...) {
        delete[] copy;      // when get an exception, prevent memory leak
        throw;              // re-throw the current exception
    }

    delete[] copy;          // 不仅catch内要delete，又得在正常代码退出前重写一遍
                            // 还是RAII好哇
}
```

以上代码繁琐冗长且容易出错，如果是用string类之类的，则简单很多。
这也是基于string类优秀的内存管理。

```cpp
#include<string>

void doSomethingFrom2Source(const string& s1, const string& s2) {
    std::string copy = s1;
    // ... do sth. with copy ...
    copy += s2;             //之所以不需要try，是因为RAII内存安全
                            //无需像char* 那样delete指针再转发出去
    // ... do sth. with copy ...
}
```

### What should I throw?
C++不像其他语言，他很包容，甚至可以throw任何你想throw的(不能copy的除外)，
然而这也带来问题，到底throw什么较为合适呢？

最好throw objects而不是build-ins
objects最好是继承自std::exception class
这样，用户使用会很方便，因为可以通过`catch std::exception&`捕获大部分想要捕获的异常，
进一步地，你也可能会为它提供显性的信息，所以并不直接继承自`std::exception`
而是例如继承自`runtime_error`（runtime_error继承自std::exception）

最常见的做法是如下例子：

```cpp
#include <stdexcept>

class MyException : public std::runtime_error {
public:
    MyException() :
            std::runtime_error("MyException")
            { }

};

void f() {
    // ...
    throw MyException();
}
```

### What should I catch?
C++依然对此提供包容的不同做法，所以你能catch：

1. value (尽量避免)
2. reference (最好)
3. pointer (某些特殊情况用pointer好于ref.)

这就像函数参数一样，这些都可以作为catch的“函参”，规则是一致的。

然而，通常情况下，**最好catch by reference**。
尽量避免catch value以免带来额外的copy开销，且[copy行为还说不定跟原值不一致](https://isocpp.org/wiki/faq/virtual-functions#virtual-ctors)。

某些特殊情况下pointer好于ref.

### But MFC seems to encourage the use of catch-by-pointer?
这是因为MFC早于C++对Exception语法的支持，为了向后兼容所以用的指针，
所以，视情况而定。
如果你依赖于MFC框架，入乡随俗，按照pointer用法。
如果你构建自己的exception构架，只是用了一部分MFC，最好用reference。

关于catch by pointer的一个重要问题是：不清楚到底谁负责delete，例如：

```cpp
MyException x;

void f() {
    MyException y;

    try {
        switch ( (rand() >> 8) % 3 ) {
            case 0: throw new MyException;
            case 1: throw &x;
            case 2: throw &y;
        }
    }
    catch (MyException* p) {
        // Should we delete p here or not???
    }
}
```

对于以上例子，有以下几个问题：

1. 假设同时存在一个本地new的对象，和一个全局的对象，那么是否要delete很难判断。
2. 那么我统一一下，全部用new，全部delete行了吧？这样堆占用很大，吃内存
3. 那么我统一一下，全部不用new（也就不用delete）可否？
这样的话，就没有本地的exception了（因为出了作用域就被释放了，BTW static可以吧？）
no no no,static不是线程安全的。

所以相比而言，catch-by-ref更容易些。

### What does throw;(without any object)mean? Where would I use it?
`throw;`主要意思是**re-throw the current exception**
甚至你可以修改现有的exception然后再转发，
这一惯用法(idiom)也可用来实现简单的**stack-trace**

另一个re-throwing惯用法是**exception dispatcher(调度器、分配器)**。
这个用法有点像多态，如下所示：

```cpp
void handleException() {
    try {
        throw;
    }
    catch (MyException& e) {
        // ... do sth. ...
    }
    catch (YourException& e) {
        // ... do sth. ...
    }
}

void f() {
    try {
        // ... sth. that might throw ...
    }
    catch(...) {
        // 用一个handleException统一调度处理n多不同exceptions
        handleException();
    }
}
```

### How do I throw polymorphically?
如果想实现多态，如上的调度器已经实现了，
而有些人会误以为按照如下思路可以实现多态：

```cpp
class MyExceptionBase { };

class MyExceptionDerived : public MyExceptionBase { };

void f(MyExceptionBase& e) {
    // ...
    throw e;
}

void g() {
    MyExceptionDerived e;
    try {
        f(e);
    }
    catch (MyExceptionDerived& e) {
        // never achieve here
    }
    catch (...) {
        // it caught here
    }
}
```

如上代码发现并没有通过继承的方式实现多态，
这是为什么呢？
答案是因为函数f中的`throw e;`实际上是**static** type of expression e。
即MyExceptionBase类型，而不是动态类型MyExceptionDerived。
thrown object被copied而不是[makign a "virtual copy"](https://isocpp.org/wiki/faq/virtual-functions#virtual-ctors)

如果要用多态，需要用如下方式：

```cpp
class MyExceptionBase {
public:
    virtual void raise();
};

void MyExceptionBase::raise() {
    throw *this;
}

class MyExceptionDerived : public MyExceptionBase {
public:
    virtual void raise();
};

void MyExceptionDerived::raise() {
    throw *this;
}

void f(MyExceptionBase& e) {
    // ...
    e.raise(); //用虚函数包装，原来如此
}

void g() {
    MyExceptionDerived e;
    try {
        f(e);
    }
    catch (MyExceptionDerived& e) {
        // achieve here
    }
    catch(...) {
        // ...
    }
}
```

### When I throw this object, how many times will it be copied?
看情况，可能为“零”。

如果不throw，当然重来就不会调用exception的ctor，没有任何负担。

而需要注意的是throw的对象必须是可copy的，
因此要提供pulicly accessible copy-constructor.
即使一次也没有copy，编译器也是会检测exception类是否有copy-ctor且可访问。

### Why doesn't C++ provide a "finally" construct?
因为C++提供更好的机制——RAII

例如如下代码：

```cpp
class File_handle {
    FILE* p;
public:
    File_handle(const char* n, const char* a) {
        p = fopen(n, a);
        if (p == 0) {
            throw Open_error(errno);
        }
    }

    File_handle(FILE* pp) {
        p = pp;
        if (p==0) {
            throw Open_error(errno);
        }
    }

    ~File_handle() { fclose(p); }

    operator FILE*() { return p; }

    // ...
};

void f(const char* fn) {
    File_handle f(fn, "rw");
    // use file through f
} // automatically destroy f here(exception-safe)
```

### Why can't I resume after catching an exception?
换句话说，为什么C++不提供一个原语使得可以返回到exception点继续执行呢？

这是由于，如果要想正确地返回到exception点，throw的作者以及catch的作者必须非常熟悉彼此的代码。
这就使得throw和catch耦合在一起了，就没法实现库实现和库调用的分离。

在Stroustrup的书《The Design and Evolution of C++》中，他阐述了关于这一命题的探讨。

[Java的try-finally]:http://blog.sina.com.cn/s/blog_639dde240100rjjw.html

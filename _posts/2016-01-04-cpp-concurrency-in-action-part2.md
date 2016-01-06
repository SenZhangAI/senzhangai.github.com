---
layout: post
title: "C++并发编程 part2 thread基本用法"
description: "《C++ Concurrency in Action》阅读笔记"
keywords: C++, concurrency, C++11, Thread, Mutex
category: C++
tags: [C++]
---

## 前言
本章讲述thread的基本用法

## Managing threads
总体而言，C++设计理论是让thread管理变得简单。
所有与之相关的管理都通过`std::thread`对象关联thread资源。

## 2.1 basic thread management
每个程序都至少有一个thread，即`main()`作为起始thread，
而每个新加入的thread都是在已有的thread某个执行点上生成分支。

### 2.1.1 Launching a thread
生成一个thread对象即可：

``` cpp
std::thread my_thread(do_some_job);
```

其中`do_some_job()`是any callable type，例如函数、仿函数
或者函数指针、lambda表达式。

#### 当thread构造函数为仿函数的注意事项
首先要注意该仿函数是copy到新的thread中，
所以需要确保copy行为对于原始资源和新生成资源要一样。

其次，由于仿函数也是类，可以构造，
所以有时候也想用一个内部匿名的构造函数生成一个临时对象作为thread的函参
注意如下语法，
本打算将一个仿函数通过内部匿名temp构造生成一个对象作为thread构造的函参：

```cpp
std::thread my_thread( Func_temp() );
```

但实际对编译器而言却是另一个语意：
**编译器会认为是一个函数声明，而不是一个thread对象定义**，
函数为my_thread，返回std::thread类型，
函参为一个函数指针，该指针参数为void，返回Func_temp类型。

为了消除歧义，可用如下方式：
1. 加括号

```cpp
std::thread my_thread( (Func_temp()) );
```

2. 用uniform initialization syntax(统一初始化语法)

```cpp
std::thread my_thread{ Func_temp() };
```

3. 某些情况下可以改用lambda表达式，同样实现匿名temp

```cpp
std::thread my_thread( []() {

// do_some_job

} );
```

#### 必须显式说明父thread是否等待子thread，
最常见的形式是等待用`my_thread.join()`，不等待用`my_threa.detach()`，
如果不明确（显式）说明是否等待，则整个程序会终止(thread的dtor调用terminate())。

对于join()或者detach()执行的时机，理论上只要thread对象没有被销毁任何时候都可以。
(注意thread对象被销毁的情形，例如局部对象退出函数作用域，生命期结束。)

**对于detach**
实践过程中通常detach再生成thread对象后就立即执行了，
同时当心线程中的资源别被主线程提前释放了，
尤其遇到子线程通过reference或者pointer调用父线程中的资源时(虽然copy了引用或指针)。
所以注意**线程访问local variables**的情况。解决方法比如copy数据而不是引用。

**对于join**
而join则通常在生命期结束前执行（通过RAII管理再合适不过了），
实际执行时机也根据实际情况。

### 2.1.2 Waiting for a thread to complete
用join，
如此一来父线程得等待子线程，可能会耗时更久。
join是简单粗暴的方式。如要更细粒度地精确控制，考虑用`condition variables`或者`future`等。

### 2.1.3 Waiting in exceptional circumstances
对于异常可以认为是另一个退出点，
join必须而退出前执行（即thread生命期结束前），
所以要保证每个退出点join，那么最好的方式就是RAII了，(或者对于detach则没有此问题)
RAII很经典，所以这里给出例子：
(注意thread对于move的支持)

```cpp
class thread_guard
{
    //域守护类，这里引用thread资源（后续可以看到可以不用引用，用move更好！）
    std::thread& t;
public:
    explicit thread_guard(std::thread& t_):
    t(t_)    {}

    ~thread_guard()
    {
        //由于是引用资源，不能保证封装性(不变式)，
        //可能thread已经被join了，所以先检查是否可join
        if(t.joinable())
        {
            t.join();
        }
    }
    //thread不可被复制
    thread_guard(thread_guard const&)=delete;
    thread_guard& operator=(thread_guard const&)=delete;
};

struct func;

void f()
{
    int some_local_state=0;
    func my_func(some_local_state);
    std::thread t(my_func);
    thread_guard g(t);
    do_something_in_current_thread();
}
```

上述采用引用的方式不如下述采用move的方式好，
move后对资源的封装性更好。

```cpp
class scoped_thread
{
    std::thread t;
public:
    explicit scoped_thread(std::thread t_):
    t(std::move(t_)) //使用move将资源所有权交给守护类，这样封装性更好
    {
        //前置条件
        if(!t.joinable())
            throw std::logic_error(“No thread”);
    }
    ~scoped_thread()
    {
        //保证封装性(维护不变式)的基础上，可确保dtor不必再检查是否join
        t.join();
    }
    scoped_thread(scoped_thread const&)=delete;
    scoped_thread& operator=(scoped_thread const&)=delete;
};
struct func;
void f()
{
    int some_local_state;
    scoped_thread t(std::thread(func(some_local_state)));
    do_something_in_current_thread();
}
```

### 2.1.4 Running threads in the background
一旦某个thread执行detach之后，控制权与所有权将转移，将不能再控制它或者引用它。
detach常用于后台执行。
注意确保detach的thread关联的资源在退出时回收资源。
detached threads也据UNIX的概念称其为daemon threads(守护线程)，没有显式的user interface
这类threads通常执行时间较长，例如后台监视文件系统、清除无用的对象catch或优化数据结构等。

跟join一样，detach也只能在joinable为true时可以执行。

以一个打开新文件/标签页为例，
因为其与原有文件无关了，是一个独立的程序，所以应该用detach，如下：

```cpp
void edit_document(std::string const& filename)
{
    open_document_and_display_gui(filename);
    while(!done_editing())
    {
        user_command cmd=get_user_input();
        if(cmd.type==open_new_document)
        {
            std::string const new_name=get_filename_from_user();
            std::thread t(edit_document,new_name);
            t.detach();
        }
        else
        {
            process_user_input(cmd);
        }
    }
}
```

## 2.2 Passing arguments to a thread function
默认情况下，参数是**copied** into internal storage。
即使参数是引用，也是copy行为，例如：

```cpp
void f(int i, std::string& const s);
//"Hello" 字符串字面值首先被copy到线程作为实参
std::thread t(f, 3, "Hello");
```

### 形参为string&，实参为char*时可能出现的问题
**string literal**是C++中的概念，实际上就是字符串常量，
为兼容C语言，C++将所有字符串字面值后都有编译器末尾添上空字符。
实参为字符串字面值而实参为string的情况，有一个`char const*`转`std::string`的过程(未考证)。

当形参为string&，而实参为`char const*`（字符串字面值或者字符串数组），
因为存在一个`char const*`转`std::string`的过程，而如果转化过程中detach了，
主程序无需等待，而如果此时原字符串又release了，就可能出现未定义行为。
解决办法是实参先显式转化为string，例如：

```cpp
void f(int i,std::string const& s);
void oops(int some_param)
{
    char buffer[1024];
    sprintf(buffer, "%i", some_param);

    //std::thread t(f, 3, buffer);  //可能出现dangling pointer
    //实参先显式转化为string，避免buffer被提前释放了
    std::thread t( f, 3, std::string(buffer) );
    t.detach();
}
```

按照书上的说法，我猜或许所有存在这类隐式转化的行为都要注意。

### 修改默认copy为传引用
有时候我们需要的是引用，而不是copy，例如操作很大的数据结构，
或者分支thread也是为了操作同一片数据区域时：

```cpp
void update_data_for_widget(widget_id w,widget_data& data);
void oops_again(widget_id w)
{
    widget_data data;
    //std::thread t(update_data_for_widget,w,data); //看上去传引用实际copy，不能操作原数据
    //解决方法是wrap with std::ref
    std::thread t( update_data_for_widget, w, std::ref(data) );
    display_status();
    t.join();
    process_widget_data(data);
}
```

### 传类成员函数
对于类成员函数，可视为普通函数 + this指针，所以可以这样用：

```cpp
class X
{
public:
    void do_lengthy_work();
};

X my_x;

//相当于do_lengthy_work(this);
std::thread t(&X::do_lengthy_work,&my_x);
```

当然了，这种用法应该是故意按照C++的this指针原理设计的
（据文中所述std::bind有类似用法），
但已经测试，普通的类成员函数不能这样调用this。

### 参数的move而非copy
有些参数不能被copy只能被move，
例如`std::unique_ptr`

move常见于两种，一种作为参数，一种作为返回，
如果被move的对象是临时变量则自动实现move语义，
如果不是(例如 named variables)，则需要显式调用`std::move(obj)`

`std::thread`和`std::unique_ptr`有些相似，
都是关联唯一一个资源(thread关联线程)
所有权可以被move，但不能被copy

## 2.3 Transferring ownership of a thread
例如如下情况下可能有需求要转移所有权：
(1) 想要一个thread在后台执行，但不想在该函数内等待，
跳出该函数而不结束thread生命期，thread所有权交给call function：

```cpp
std::thread f() {
    // ...
    // 跳出函数，thread生命期没有结束，所有权交给calling function
    return std:: thread(some_function);
}
```

(2) 或者相反，将thread所有权交给某个called function

```cpp
void f(std::thread t);

void g() {
    std::thread t(some_function);
    //因为t是named variables，需要显式move
    f( std::move(t) );

    //如果是temp thread，则自动隐式move
    f( std::thread(some_other_function) );
}
```

需要C++的资源拥有类型都有类似特征，只能被move，不能被copy
例如`std::ifstream`, `std::unique_ptr`当然也包括`std::thread`。

例如：

```cpp
std::thread t1(some_function);
std::thread t2 = std::move(t1); //move语义
//如下也是move语义，是临时变量的隐式move，并不是copy赋值语义
t1 = std::thread(some_other_function);
t1 = std::move(t2); //注意由于t1非空，所以此move语句将导致执行terminate()
```

### move用于RAII
如上已述，略

### 在container中move
例如如下thread作为vector元素时，pushback操作中隐式move了temp thread：

```cpp
vector<std::thread> threads;

threads.pushback( std::thread(some_function) );

//mem_fn为调用各元素的成员函数
foreach(threads.begin(), threads.end(), std::mem_fn(&std::thread::join))
```

通过vector等容器管理threads是更为合适的管理多线程的方式，而不是一个个构造，一个个join。

## 2.4 Choosing the number of threads at runtime

即根据runtime实际情况选择合适的线程数，

例子参见：
[C++并发实战5：并行化的std::accumulate](http://blog.csdn.net/liuxuejiang158blog/article/details/17118649)

需要注意的是，线程的结果不能return，所以常常会放在参数中，用引用的方式来回传结果，例如`func(T& result);`
另一种回传结果的方式是`future`

## 2.5 Identifying threads
C++函数中，thread将线程id存储于`std::thread::id`类结构中。

通过`get_id()`获取id值
如果thread没有关联到任何线程，则id通过默认构造函数生成一个特殊的称之为"not any thread"对象，
如果关联了线程，通过`std::this_thread::get_id()`获取id值。

`std::thread::id`支持各种常见的比较操作，大于、小于、等于（表示同一个thread），等等。
STL也提供`std::hash<std::thread::id>`这样使用键值的方式。

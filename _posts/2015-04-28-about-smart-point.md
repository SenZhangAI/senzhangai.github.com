---
layout: post
title: "关于C++的智能指针"
description: "关于智能指针的实现原理，意义以及使用场合"
keywords: "C++，智能指针"
category: programming
tags: [C++, 智能指针]
---

## 主要参考

1. [如何理解智能指针？](http://www.zhihu.com/question/20368881)

2. [C++中方法的参数和返回值、类成员变量什么时候该用原始指针什么时候该用智能指针？](http://www.zhihu.com/question/22821303)

### 什么是智能指针

智能指针是普通指针的一层封装。

>从较浅的层面看，智能指针是利用了一种叫做RAII（资源获取即初始化）的技术对普通的指针进行封装，
这使得**智能指针实质是一个对象，行为表现的却像一个指针。**

### 智能指针有哪些

[Boost::Smart Pointers](http://www.boost.org/doc/libs/1_50_0/libs/smart_ptr/smart_ptr.htm)

>1) scoped_ptr:
这是比较简单的一种智能指针，正如其名字所述，scoped_ptr所指向的对象在作用域之外会自动得到析构，一个例子是：[Boost::Smart Pointers](http://www.boost.org/doc/libs/1_50_0/libs/smart_ptr/smart_ptr.htm)

>此外，scoped_ptr是non-copyable的，也就是说你不能去尝试复制一个scoped_ptr的内容到另外一个scoped_ptr中，这也是为了防止错误的多次析构同一个指针所指向的对象。

>2) shared_ptr:
很多人理解的智能指针其实是shared_ptr这个范畴。

>正如@陈良乔 和@郑斌 同学的答案所提到的，shared_ptr中所实现的本质是引用计数(reference counting)，也就是说shared_ptr是支持复制的，复制一个shared_ptr的本质是对这个智能指针的引用次数加1，而当这个智能指针的引用次数降低到0的时候，该对象自动被析构，这一点两位同学给的答案都非常精彩，不再赘述。

>需要特别指出的是，如果shared_ptr所表征的引用关系中出现一个环，那么环上所述对象的引用次数都肯定不可能减为0那么也就不会被删除，为了解决这个问题引入了weak_ptr。

>3) weak_ptr:
对weak_ptr起的作用，很多人有自己不同的理解，我理解的weak_ptr和shared_ptr的最大区别在于weak_ptr在指向一个对象的时候不会增加其引用计数，因此你可以用weak_ptr去指向一个对象并且在weak_ptr仍然指向这个对象的时候析构它，此时你再访问weak_ptr的时候，weak_ptr其实返回的会是一个空的shared_ptr。

>实际上，通常shared_ptr内部实现的时候维护的就不是一个引用计数，而是两个引用计数，一个表示strong reference，也就是用shared_ptr进行复制的时候进行的计数，一个是weak reference，也就是用weak_ptr进行复制的时候的计数。weak_ptr本身并不会增加strong reference的值，而strong reference降低到0，对象被自动析构。

>为什么要采取weak_ptr来解决刚才所述的环状引用的问题呢？需要注意的是环状引用的本质矛盾是不能通过任何程序设计语言的方式来打破的，为了解决环状引用，第一步首先得打破环，也就是得告诉C++，这个环上哪一个引用是最弱的，是可以被打破的，因此在一个环上只要把原来的某一个shared_ptr改成weak_ptr，实质上这个环就可以被打破了，原有的环状引用带来的无法析构的问题也就随之得到了解决。

>4) intrusive_ptr:
简单的说，intrusive_ptr和shared_ptr的区别在于intrusive_ptr要求其所指向的对象本身实现一个引用计数机制，也就是说当对象本身包含一个reference counter的时候，可以使用intrusive_ptr。

>在实际使用中我几乎从来没有见到过intrusive_ptr...

### 为什么需要智能指针

就一句话：**而如果是类的话，不论异常或者未delete，因为跳出该语句块即会调用类的析构，因此是安全的，这就是用类封装指针的原因**

>简而言之，是为了防止`delete`失败等错误，以及确保异常安全。
智能指针还有一重更加深刻的含义，就是把@陈硕所说的value语义转化为reference语义。
C++和Java有一处最大的区别在于语义不同。

##### 关于防止delete失败等错误

因为普通指针会遇到如下问题：

```cpp
Class A* p = new A();
p->DoSomeThing();
delete p;
```

如果忘了`delete`的话，会造成一个悬挂指针(dangling pointer)，带来内存泄漏（空间浪费）。泄漏就是“内存池子里的水漏掉，越来越少了”，很形象啊。

即便没有忘记`delete`如果`DoSomeThing()`在运行时抛出异常，那么`delete p;`极有可能不会执行。



#####关于“将value语义转化为reference语义”

以下是JAVA代码：

```java
Animal a = new Animal(); //JAVA
Animal b = a;
```

以下是C++代码：

```cpp
Animal a = new Animal(); //C++
Animal b = a;
```

对于JAVA而言，`a` 和 `b` 都指向同一个对象，只是`reference`引用。而C++中，`a` 和 `b` 是两个对象，用了`copy`。

但有些情况需要用到`reference` 而不是 `copy`， 例如：

>在编写OOP程序时，value语义带来太多的困扰，例如TCP连接中我封装一个`accept`函数接收请求，那么应该是这样的：

>``` cpp
Socket accept();
```

>这就带来一个问题，采用对象做返回值，这里面有一个对象的复制的过程，但是Socket因为某些原因，我让他继承了`boost::noncopyable`，总之就是Socket失去了复制和赋值的能力，那么该怎么办？
我们首先想到指针，在accept内部new生成一个对象，然后返回指针。但是问题更多，这个对象何时析构？ 过早析构，程序发生错误，不进行析构，又造成了内存泄露。
这里的解决方案就是智能指针，而且是引用计数型的智能指针。

>```cpp
typedef boost::shared<Socket> SocketPtr;
SocketPtr accept();
```

>这样外部就可以用智能指针去接收，那么何时析构？当然是引用计数为0，也就是我不再需要这个Socket的时候析构。
这样，我们利用了`SockerPtr`，实现了跟Java类似的Reference语义。

#### 如何实现智能指针

[C++ primer智能指针（HasPtr）实现](http://blog.csdn.net/randyjiawenjie/article/details/6723367)

[C++中智能指针的设计和使用](http://blog.csdn.net/hackbuteer1/article/details/7561235)


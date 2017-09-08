---
layout: post
title: "a little java, a few patterns(3)"
description: "一本设计模式的好书，第五章及前五章小结"
keywords: design pattern
category: DesignPattern
tags: [design pattern]
---

## Chapter 5 Objects Are People, Too

### 重新组织数据结构

之前的数据结构像一个链表，例如算总价，把串接在一起的水果列表递归一遍即可，
为了构成链表，每个具体类中都包含抽象类用来链接**下一个**对象。
然而可进一步优化数据结构。

```java
abstract class Link {}

class LinkEnd extends Link {}

class LinkNode extends Link {
    Object content;
    Link l;
    LinkNode(Object _o, Link _l) {
        content = _o;
        l = _l;
    }
}
```

这样具体类就与串接链表结构分离而无需内含链接下一个节点的抽象类，好比珍珠与项链的关系，珍珠作为独立个体。
另一个好处是之前每个具体类都要实现一些重复性的方法，而现在，几乎只需要极少数链表串接类中实现这些迭代操作。

### 进一步抽象操作

之前实现了比如`removeA()`函数删除A，或`substitleB(A)`函数将所有B替换为A，

但这还不够通用，例如我如果想要删除B再写`removeB()`，等等。。。太麻烦。

这里更进一步，
本章的主题就是利用Object实现更进一步的抽象。

关键在于进行**比较判断**。

比如，可以实现通用的函数`remove(Object o)`， 移除某个实物，

当包含的某个东西与o相等时（用 equals函数，函数自定义，instanceof 或比较值等）
移除o，移除o作为一个通用的动作，具体的移除动作再找个地方定义一下。

那么就可以比方说`Some.remove(new ClassA())`，或者`Some.remove(new Integer(3))`等等，移除相应内容

例如：

``` java
class Anchovy extends Something {
    @override
    public boolean equals(Object o) {
        return (o instanceof Anchovy);
    }
}

//这里o为Object类型，其实际类型在运行期才动态给定
//equals函数是Object类声明的，默认返回false，因此要override
// 这个例子来自书上，但通常不要这样做！
// == 判断对象地址是否相同，
// instanceof 判断是否是某个class的实例
// 而通常equals则自己定义，用来判断两个对象的**值**是否相同
// 这里之所以这样做是因为需要用equals作接口，instanceof无法动态地用于抽象class的判断，否则语法错误
if (new Anchovy().equals(o)) {
    // ...
}
```

```java
class SubstV {
    Pie forBot(Object n, Object o) { return new Bot();}
    Pie forTop(Object t, Pie r, Object n, Object o) {
        if ( o.equals(t))
            return new Top(n,r.subst(n,o));
        else
            return new Top(t,r.subst(n,o));
    }
}
```

## 前五章回顾与小结

之前的visitor class是不是太boring了一点？
第六章将进一步抽象，改进成最终的visitor设计模式。

仔细想想貌似函数式编程很最重要的一点是操作对象的抽象，
比如`(myfunc a b)`，根本不关心a和b的具体是什么，
只要函数体中的操作能对a、b都有效即可，
这就有点vistor模式，多分派的特点了。

不过，进一步学习第六章前，首先回顾一下之前的各个章节内容：

### 第一章 Modern Toy
第一章介绍类型扩展，因为编程中，仅用内置的类型很局限，
需要对内置类型进一步扩展成更强大的类型，或功能。

那么如何考虑构造类型呢？

关键在于抽象，一个类型是一个集合，例如boolean类型就是{true, false}的集合，
一个集合有某些共同的特性，而各个元素之间又有所区别。

用abstract class表示这个类型，用class继承表示其中的元素或者说变量。

其实我不完全理解这个思路，在编程中思考抽象类和类的时候会这样思考？

或者很可能作者是为了告诉大家如何按照函数式编程的思路思考抽象类和类。
所以提出一个思考抽象类和类的方式。

我猜测这应该是将面向对象与函数式编程融合的最基本思考模式。
为什么，不将类看成集合，将实例看成元素呢？
答： 因为函数式编程中，函数关心更抽象的，而非具体的，
例如`(myfunc a b)`中思考a的时候，不会关心a,b具体是什么，而是抽象，
面向抽象类，面向接口编程。

### 第二章 Methods to Our Madness

第二章主要介绍了用虚函数多态才实现每个子类的不同行为，

比较有意思的是可以串接类来递归获取结果，看上去有些类似于装饰者模式。

```java
float totalPrice =
    new Banana(
        new Orange(
            new Apple(
                new Banana(
                    new Base())))).Price();
```

另外需要补充的是对于子类共同的域或者方法，可以放在父类里面。

### 第三章 What's New

有意思的是第二章的递归计算不仅可实现普通的计算，
甚至通过new，可以实现替换，插入，删除特定类的操作，

例如可设计函数将如上所有Banana换成Apple，或者所有Banana上添加Orange，
或者删除所有Apple等。

可进一步了解 interpreter and composite设计模式

删除所有Apple类简单，但要想更灵活的替换或者添加，需要用参数，找到Banana这个具体类，所以需要instanceof函数的帮助，这在第四章给出思路。

### 第四章 Come to Our Carousel

第三章遇到的问题是，要实现这种串接函数，或者其他接口，所有的类都要写实现，
如果所有类都分散在不同文件， 例如，想查看`RemoveApple()`函数，则很难在一个类文件、一个类文件中查找，而且这样造成理解困难，
所以，要想办法将实际方法集中起来，办法是集中到一个类里面处理，
我们给这样的访问其他类来执行某个过程的类取个名字，叫访问者类(Visitor Class)。

注意这里的访问者类不是访问者模式，或者说是不够完整的访问者模式，目前好处还只能说是将操作集中到一起，便于理解。
因为这里还是存在具体的方法，
而访问者模式更进一步将方法也抽象了，这样就做到的**数据**与**针对数据的具体操作**的解耦。

### 第五章 Object Are People, Too

见此文

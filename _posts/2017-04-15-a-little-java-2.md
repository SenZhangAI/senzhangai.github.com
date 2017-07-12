---
layout: post
title: "a little java, a few patterns(2)"
description: "一本设计模式的好书，第四章，也讨论了点Java访问者模式"
keywords: design pattern, visitor, 访问者
category: DesignPattern,
tags: [design pattern]
---

## Chapter 4 Come to Our Carousel

如上的类组织方式的问题在于：

当抽象方法越来越多，而方法分散在各个类中，
很难理清逻辑。

解决方法，用visitor class 将这些方法集中到一起。

### visitor class是访问者模式么？

对于设计模式而言，访问者模式有种说法是可以在单分派语言中模拟双分派技术。

而此书中的visitor class貌似不是这个意思。
但从某种意义上来说也算是访问另一个类了。

-------------

** 更新 **

答案在第六章揭晓，第六章才是完全版的visitor模式！

-------------

对于java的访问者模式，示例代码如下：

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

//@see http://www.cnblogs.com/haore147/p/3888182.html
//@see http://blog.csdn.net/zhengzhb/article/details/7489639

public class visitor_pattern {
    public static void main(String[] args) {
        List<Element> list = ObjectStruture.getList();
        List<IVisitor> visitors = VisitorStruture.getList();
        for (int i = 0; i < list.size(); i++) {
            list.get(i).accept(visitors.get(i));
        }
    }
}

class ObjectStruture {
    public static List<Element> getList() {
        List<Element> list = new ArrayList<Element>();
        List<Element> visitors = new ArrayList<Element>();
        Random ran = new Random();
        for (int i = 0; i < 10; i++) {
            int a = ran.nextInt(100);
            if (a > 50) {
                list.add(new ConcreteElement1());
            } else {
                list.add(new ConcreteElement2());
            }
        }
        return list;
    }
}

class VisitorStruture {
    public static List<IVisitor> getList() {
        List<IVisitor> list = new ArrayList<IVisitor>();
        Random ran = new Random();
        for (int i = 0; i < 10; i++) {
            int a = ran.nextInt(100);
            if (a > 50) {
                list.add(new Visitor1());
            } else {
                list.add(new Visitor2());
            }
        }
        return list;
    }
}

abstract class Element {
    public abstract void accept(IVisitor visitor);

    public abstract void doSomething();
}

interface IVisitor {
    public void visit(ConcreteElement1 el1);

    public void visit(ConcreteElement2 el2);
}

class ConcreteElement1 extends Element {
    public void doSomething() {
        System.out.println("这是元素1");
    }

    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
}

class ConcreteElement2 extends Element {
    public void doSomething() {
        System.out.println("这是元素2");
    }

    public void accept(IVisitor visitor) {
        visitor.visit(this);
    }
}

class Visitor1 implements IVisitor {
    public void visit(ConcreteElement1 el1) {
        System.out.print("Visitor1: ");
        el1.doSomething();
    }
    public void visit(ConcreteElement2 el2) {
        System.out.print("Visitor1: ");
        el2.doSomething();
    }
}

class Visitor2 implements IVisitor {
    public void visit(ConcreteElement1 el1) {
        System.out.print("Visitor2: ");
        el1.doSomething();
    }
    public void visit(ConcreteElement2 el2) {
        System.out.print("Visitor2: ");
        el2.doSomething();
    }
}
```

结果：

```
Visitor2: 这是元素2
Visitor1: 这是元素2
Visitor2: 这是元素1
Visitor1: 这是元素1
Visitor2: 这是元素2
Visitor2: 这是元素2
Visitor1: 这是元素1
Visitor1: 这是元素2
Visitor2: 这是元素1
Visitor2: 这是元素2
```

可以看到实现了多分派，Visitor1访问元素1，2，Visitor2也可访问1，2，并作不同处理

之前写过一个htmlparser也是需要双分派，貌似c++还需要dynamic_cast而java不需要，
直接获得了其实际类型。

<https://github.com/SenZhangAI/htmlParser/blob/master/src/htmlRander.cpp>

C++之所以需要dynamic_cast，
是因为这里双分派必须有一个知道其类型，然后通过重载还实现分派，
java应该也是通过类似dynamic_cast等方式，目的在于获得确切类型。

不过到底C++能不能不用cast也有待进一步考证。

另外，对于visitor模式，不局限于双分派的场景，很多时候不需要严格的双分派策略，
例如渲染，直接用`RanderHTML()`、`RanderPlainText()`等函数，就可以很好的处理问题。

还有，访问者是接口，而被访问的元素是抽象类，因为是抽象类，
所以可以将类的共同成员方法或者域汇集到抽象类中，
而访问者只提供操作，所以用接口最合适。

#### Java 访问者模式小结
优势：

* 双分派
* 方便地新增针对该类的操作

### 代码示例

```java
public class visitor_class {
    public static void main(String[] args) {
        new TypeB(
            new TypeB(
                new TypeC(
                    new TypeA(
                        new BaseClass())))).doSomeThing();
    }
}

abstract class AbstractClass {
    AbstractClass son;
    DoSthVisitor v = new DoSthVisitor(); //注意这里初始化
    abstract int doSomeThing();
}

class BaseClass extends AbstractClass {
    int doSomeThing() { return 0; }
}

class TypeA extends AbstractClass {
    TypeA(AbstractClass _son) { super.son = _son; }
    int doSomeThing() { return v.forClassA(1); }
}

class TypeB extends AbstractClass {
    TypeB(AbstractClass _son) { super.son = _son; }
    int doSomeThing() { return 100 + v.forClassB(son); }
}

class TypeC extends AbstractClass {
    TypeC(AbstractClass _son) { super.son = _son; }
    int doSomeThing() { return v.forClassC(); }
}

//其优势是将所有子类做的事情集中到一起了，否则分散各处不好找
class DoSthVisitor {
    //每个函数可以有不同的参数类型，这点很棒，只要返回类型一致
    int forClassA(int a) {
        System.out.println("do A");
        return a;
    }
    // 子节点迭代
    int forClassB(AbstractClass son) {
        System.out.println("do B");
        return son.doSomeThing();
    }
    int forClassC() {
        System.out.println("do C");
        return 0;
    }
}
```

如果不是因为Java必须用class，应该更方便把？直接把函数独立出来不就不需要visitor_class了么？



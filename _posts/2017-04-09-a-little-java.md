---
layout: post
title: "a little java, a few patterns(1)"
description: "一本设计模式的好书,1-3章"
keywords: design pattern
category: DesignPattern
tags: [design pattern]
---

## Foreword

本书作者(Friedman)将函数式程序设计（input-output driven）
很自然地引入到著名的Object-oriented设计模式中，并很好地融合了两者。

我觉得最好带着一部分函数式编程的思路阅读本书。

读完本书若进一步深入，参考最后一节的参考书。

## Chapter 1 Modern Toy

``` java
abstract class Seasoning {}

class Salt extends Seasoning {}
class Pepper extends Seasoning {}
```

其中`Seasoning`调味品是一个类型Type，
`Salt`以及`Pepper`是它的变量variants。

即：

**abstract class** introduces a datatype,
**class** introduces a variants.

### Tips
```
When specifying a collection of data,
use abstract classes for datatypes and extended classes for variants
```

类型是一个collection，但许多类型之间会有重合的部分，
例如几乎所有类型都是`Object`的子类。
即那些`new`生成的是`Object`，所以int类型不是，需要包装成例如`Integer(5)`。

也有完全不重合的，例如int和boolean。

## Chapter 2 Methods to Our Madness

```java
abstract class Shape {
    abstract double area();
}
class Rect extends Shape {
// field and constructor...
    double area() {
        return x * y;
    }
}
class Circle extends Shape {
// field and constructor...
    double area() {
        return PI * r * r;
    }
}
```

abstract methods 总属于 abstract class，
用于约定所有继承该抽象类的concrete class必须实现该抽象方法。

``` java
abstract class Fruit {
    abstract float Price();
}

class Base extends Fruit {
    float Price() { return 0; }
}

class Banana extends Fruit {
    Fruit f;
    Banana(Fruit _f) { f = _f; }
    float Price() { return f.Price() + 1; }
}

class Orange extends Fruit {
    Fruit f;
    Orange(Fruit _f) { f = _f; }
    float Price() { return f.Price() + 2; }
}

class Apple extends Fruit {
    Fruit f;
    Apple(Fruit _f) { f = _f; }
    float Price() { return f.Price() + 3; }
}

float totalPrice =
    new Banana(
        new Orange(
            new Apple(
                new Banana(
                    new Base())))).Price();
```

用以上不断传递给下一层的设计模式应该是装饰者模式吧？

文中的例子是判断烤串上是不是只有洋葱`abstract boolean onlyOrion()`，
模式类似。

这看上去有点像Scheme中的`cdr`，不断递归求解下一层的内容。

```scheme
(define total-price(x)
    (+ price(x) total-price(cdr(x))))
```

补充：
当子类中有共同的域或者方法时，放到抽象类里，
例如这里的`x`,`y`以及`boolean closerTo(Point p)`方法。

```java
abstract class Point {
    int x;
    int y;
    Point(int _x, int _y) {
        x = _x;
        y = _y;
    }
    boolean closerTo(Point p) {
        return this.distance() < p.distance();
    }
    abstract int distance();
}

class CartesianPt extends Point {
    CartesianPt(int _x, int _y) { super(_x, _y); }

    int distance() { return (int) Math.sqrt(x * x + y * y); }
}

class StreetPt extends Point {
    StreetPt(int _x, int _y) { super(_x, _y); }

    int distance() { return x + y; }
}
```

## Chapter 3 What's New?

有意思的地方在于，可以用如上的`new`重新组装对象，
不仅可以如上remove对象，还可以在某对象前后添加对象。

### 代码示例

``` java
class Base extends Fruit {
    Fruit removeAllApple() {
        return new Base();
    }
}

class Banana extends Fruit {
    Fruit f;
    Banana(Fruit _f) {f = _f;}
    Fruit removeAllApple() {
        return new Banana(f.removeAllApple());
    }
}

class Orange extends Fruit {
    Fruit f;
    Orange(Fruit _f) {f = _f;}
    Fruit removeAllApple() {
        return new Orange(f.removeAllApple());
    }
}

class Apple extends Fruit {
    Fruit f;
    Apple(Fruit _f) {f = _f;}
    Fruit removeAllApple() {
        return f.removeAllApple();
    }
}
```

以上可看到abstract的强大，然而， 当抽象方法越来越多时，却使得类变得更重。

下一章重新组织代码。

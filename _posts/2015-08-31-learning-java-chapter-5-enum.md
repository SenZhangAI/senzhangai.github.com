---
layout: post
title: "Java学习 第5章 小节 枚举类"
description: "《Java核心技术1 基础知识》《Core Java for the impatient》笔记"
keywords: Java, 语法, 笔记, 枚举类
category: Java
tags: [Java]
---

## 枚举类
Java的enum其实很像特殊的class，实际上enum声明定义的类型就是一个类。
这些类（enum中的元素）都是类库中Enum类的子类(java.lang.Enum<E>)。它们继承了这个Enum中的许多有用的方法。

所以想了解enum可以直接查看源码，在rt.jar目录中的java.lang.Enum中，具体直接查看源码即可，

#### enum声明本质是为每个枚举值生成java.lang.Enum实例对象
Enum中有两个域：
private final String类型的**name**，保持枚举名
private final int类型的**ordinal**，可理解为ID

对于如下enum声明：

```java
public enum Size{ SMALL, MEDIUM, LARGE, EXTRA_LARGE};
```

其实可以理解为new了4个Enum实例（以下为示意性，实际代码并非如此）：

```java
new Enum<Size>("SMALL", 0);
new Enum<Size>("MEDIUM", 1);
new Enum<Size>("LARGE", 2);
new Enum<Size>("EXTRA_LARGE", 3);
//    protected Enum(String name, int ordinal) {
//        this.name = name;
//        this.ordinal = ordinal;
//    }
```

#### enum类(注意是enum类)的声明可看成构造了私有实例的特殊类
之前说明了，enum与class关键字一样也是声明一个类，只是这个类有点特殊
参见如下声明enum的类的例子：
可知enum类有以下特点：

1. 构造器是private的，且在声明中生成了枚举的实例，如下的`{RED,BLUE,BLACK,YELLOW,GREEN}`
2. 继承父类Enum的方法和域（name和ordinal），也可有自己的方法和域。
3. 在1中所述的枚举实例（枚举值）是public static final的，但构造器的语法不用这样写，例如不用写成`public static final Color RED(255,0,0)`
4. 如下的`Color.RED`就是一个Color类型的静态常量

Java Enum类型的语法结构尽管和java类的语法不一样，应该说差别比较大。但是经过编译器编译之后产生的是一个class文件。该class文件经过反编译可以看到实际上是生成了一个类，该类继承了`java.lang.Enum<E>`。

```java
//参考自：http://www.cnblogs.com/frankliiu-java/archive/2010/12/07/1898721.html
enum Color {
    // 注意这里！ 在声明中就构造了实例，也注意语法，本质构造器，但语法特殊
    RED(255,0,0),BLUE(0,0,255),BLACK(0,0,0),YELLOW(255,255,0),GREEN(0,255,0);  
    //构造枚举值，比如RED(255,0,0)  
    private Color(int rv,int gv,int bv){  
        this.redValue=rv;  
        this.greenValue=gv;  
        this.blueValue=bv;  
    }
    public String toString(){  //覆盖了父类Enum的toString()  
    return super.toString()+"("+redValue+","+greenValueblueValue+")";  
    }  

    private int redValue;  //自定义数据域，private为了封装。  
    private int greenValue;  
    private int blueValue; 
}
```

#### String转enum类型用valueOf方法
enum类型转string用toString方法或者获取名字的name方法，具体看源码
String转enum类型用valueOf静态方法，例如：`Size s =Enum.valueOf(Size.Class, "SMALL")`;

#### 遍历枚举值用values()方法
例如：

```java
for (Size s : Size.values()) {
    //do sth
}
```

values方法是编译器自动加入的扩展方法，返回包含所有枚举值的数组，查看源码查不到，参见：[values方法](http://stackoverflow.com/questions/13659217/values-method-of-enum)

#### 注意比较两个枚举值一用quals方法，还是用==
因为Enum.equal()方法比较两地址，所以比较枚举值的时候要特别注意，
网上看了相关内容，有说法枚举用在switch中会调用equals方法比较，比较直涌==，比较地址用equals。但这个结论没有考证。

### Java枚举小结——特殊类
总体而言，Java的枚举本质是类，因此扩展能力好（扩展方法、域），功能更强大！而C++等语言中枚举只是int类型的标签。

enum是特殊类，声明中就通过私有构造器给定了枚举常量。
可用默认的Enum类中的方法，也可以自己像构造class那样构造继承自Enum的enum类。

因为Java中枚举的实现更像是类，所以也是**类型安全**的，比如不可能将COLOR枚举值赋值给Size枚举变量。

有篇文章总结地不错：[Java enum的用法详解](http://www.cnblogs.com/happyPawpaw/archive/2013/04/09/3009553.html)

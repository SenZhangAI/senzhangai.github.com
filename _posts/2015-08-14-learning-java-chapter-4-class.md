---
layout: post
title: "Java学习 第4章 对象与类"
description: "《Java核心技术1 基础知识》《Core Java for the impatient》笔记"
keywords: Java, 语法, 笔记
category: Java
tags: [Java]
---

## 面向对象程序设计概述
面向对象与面向过程到底有何不同呢？
Pascal语言设计者Niklaus Wirth曾在1975年将其著作名为《Algorithms + Data Structures = Programs》。即程序有两个重要部分，算法 + 数据结构。

面向过程把算法放在第一位，数据放在第二位。

而面向对象是把数据放在第一位，其次是算法。

我个人认为，其实翻译成汇编本质也是数据以及处理数据的算法。但是面向对象的思想，构架却和面向过程不一样。

面向对象将算法与数据合理的划分在一起，这样整个构架分得很清晰，便于查错与扩展。
所以面向过程因为简单直观适用于小型程序；而面向对象则合理规划数据与算法，适用于大型程序。
例如以下场合：
如果用面向过程处理大型程序，面对大量全局变量，很容易出错，也不知道哪个算法处理哪类数据。
此外，对于多人合作的程序，面向对象的价值也体现着各自独立，只需要规划好类之间的相应接口就是数据安全的。

所以我认为面向对象应该理解一种软件构架设计风格。

### 类
是构造对象的模板。Java所有代码都位于某个类中。

### 对象
注意对象的行为（成员方法）、状态(数据成员)与标识(ID)

### 如何设计类
传统面向过程设计先从main()开始，而面向对象设计则无所谓。
**首先从设计类开始**，然后往类中添加方法。

类设计的一个简单思路是分析问题过程中的**名词**（属性）和**动词**（方法）。
当然，还有其他思路有待学习。

### 类之间的关系

类之间的关系常见的有：

*　依赖（"uses-a"）
*　聚合（"has-a"）
*　继承（"is-a"）

应该尽可能地将相互依赖的类减少，即让类之间耦合度最小。
这是因为如果A依赖于B，B的变动将牵扯到A。

## 使用预定义类
Java中，并不是所有类都具有面向对象的特性，例如Math类，但不能没有类。
我个人猜测像math类更像C++中的namespace

### 对象与对象变量
首先是构造器，即C++中的构造函数。

有些对象只需要用一次，例如取时间`new Date()`，获得即时时间就可以扔了。
但通常情况下，对象是应用多次的，这时用变量名引用对象。

个人认为Java中的对象变量与C++中的对象指针变量更像。

既然本质是对象指针，可以只声明不赋值，但更好的方式是显式地将对象变量设置为null

```java
Date deadline;
deadline = null;

if( deadline != null ) //这样就避免了因对象变量未引用就使用造成的运行错误
    //do something
```

#### C++注释-Java与C++的不同点
很多人都会错误认为Java对象变量就是C++的引用，但C++没有空引用，且引用不能被赋值。

可将Java对象变量看出C++的**对象指针**(看来我理解对了)。

**Java对象都是存储在堆中**，这点似乎与C++不一样。当一个对象包含另一个对象时，实际上是包含对象的指针，指向另一个堆中的对象。

C++中，由于指针的随意性，常常容易导致错误。而Java则限定了对象变量，如果没指向一个真正的对象就使用它则会产生一个运行时错误，这总比产生一个随机结果要好。同时Java也不必担心内存管理问题。

C++中的拷贝很简单，如果Java需要一个对象的完整拷贝，则需要用clone方法。

### 以GregorianCalendar类说明面向对象的设计
对于时间，有很多种表示，例如阳历，阴历，甚至火星历。所以把“时间”与“时间的表示”分开是一个比较好的类设计。

### 更改器方法与访问器方法
更改器方法就比如set函数，改变对象状态；
访问器方法就比如get函数，仅访问，不改变对象状态，

在C++中访问器就是const对象函数。但在Java中，并没有区分更改器与访问器的语法区别。

## 用户自定义类
Java通常习惯上按照如下的风格设计类：

```
class ClassName {
    field1;
    field2;
    ...
    constructor1;
    constructor2;
    ...
    method1;
    method2;
}
```

例如：

```java
class Myclass {
    private String name;

    public Myclass(String n) {
        name = n; //or this.name = n
    }

    public String getName() {
        return name;
    }
}
```

注意一个源文件中只能有一个public类，且该类与源文件文件名相匹配。

当然，一个源文件中可以有任意多个非公有类。

注意Java中的构造器与C++中的构造函数略有区别，
不同点在于Java的对象都是在堆中构造的，
因此，必须使用`new`操作符，而C++还可以在栈中构造，不一定用`new`，所以：

```cpp
   Myclass myclass("para1", "para2", "para3");
// C++, not Java
// Java is like this：
// Myclass myclass = new Myclass("para1", "para2", "para3");
```

### 多个源文件的使用
只需要直接用`javac ClassIncludeMain.java`命令行编译，对于该文件用到的类，Java编译器会根据类名自动查找并编译之，这也就是为什么Java要求源文件必须只有一个public类，且与文件名同名。

如果用IDE，甚至不需要查看哪个类含有main，IDE会自动查找到。

而且，如果某个源文件更新过后，Java编译器会自动重新编译之，相当于内置了make的功能。

### 隐式参数与显式参数
隐式参数就是`this`，可以省略，也可以写上。
显示参数就是方法的括号中设置的参数。

### C++注释-Java与C++的不同
如果是C++的编程风格，通常有如下场景：
在`myclass.h`中声明类，并在`myclass.cpp`中定义类：

```cpp
#include"myclass.h"
String Myclass::getName() {
    return name;
}
```

C++中如果在头文件中定义类，则视为inline函数，每个编译文件都拥有一个副本，因此执行文件也会变大，但是从汇编的角度应该减少了跳转过程，理论上效率会有提升

头文件与cpp文件分开真的是一个繁琐的事情。
Java干脆来个简单的，不用头问题，直接定义，是否内联是Java虚拟机自动完成的任务。**即时编译器会监视调用那些经常被调用的、简洁的、没有被重载的以及可优化的方法**。

### 类的封装
封装的有点不用说，通常数据私有化，用public方法来访问和更改数据。

封装要保证**一个对象的数据只能用它所设定的public方法改变，不能被任何其他方式改变**。
但是有一个很特别的不易发现的破坏封装性的地方需要注意，就是`get引用()`时：

```java
class MyDate {
    private Date date;

    public Date getDate() {
        return date;
    }
}
```

看上去没有什么问题，但是可以绕过`setDate()`改变其值，比如：

```java
//Core Java P113
MyDate haha = New MyDate();

Date d = haha.getDate(); // 这样d和haha.date都指向了同一个Date对象

dooble tenYearsInMilliSeconds = 10 * 365.25 * 24 * 60 * 60 * 1000;
d.setTime(d.getTime() - (long) tenYearsInMilliSeconds) //减10年
//这样不通过haha也可以改变haha.date的值了，封装性被破坏
```

date也是可变的，所以不能用`final`。
对于这样的可变引用数据，解决的办法是`clone()`:

```java
class MyDate {
    private Date date;

    public Date getDate() {
        return date.clone(); //这样对象内的私有Date对象就不会被引用
    }
}
```

### 私有方法
私有方法的用途主要是配合公有方法，比如作为一个中间步骤，又或者这些私有方法的调用顺序有严格要求，如果public了，其他人可以调用的话，用错了顺序可能乱套了。

### final实例域
例如雇员的名字，每个雇员对象在初始化时是肯定有的，且设置后不能改变，因此加上`final`修饰符。类似的比如ID也应该是final的，但是ID更严格，因为不能有重复的。

```java
    class Employee {
        private final String name; 
    }
```

除外需要特别注意的是：final修饰name，那么name不可变，但并不意味着name引用的对象也不可变（当然String本身是不可变的，这里例子不恰当），用C++的说法，就是是一个`Type * const variable`
而不是`const Type * const variable`或者`const Type *variable`

比如：

```
class MyDate {
    private final Date mydate;
}

```

并不意味着mydate引用的Date对象不能变，而是指mydate只能指向那一个Date对象，不能改指向其他Date对象。

### 静态域与静态常量
跟C++一样都是指的类中共享的一个数据，这个数据无论多少个对象，都只有一份且被共享，也称为类域，用`static`修饰。
例如设置ID用的nextID就适合用静态域表示，因为要确保每个ID不一样，所以可以在每构建一个对象时，用:

```java
    void setId() {
        id = nextId;//nextId适合用static修饰
        nextId++;
    }
```

还有些常用的常量作为静态常量，例如`Math.PI`就是这样
`public double final static PI = 3.1415926...`，
`System.out`也是静态常量。
这样的常量常作为public常量，供其它地方使用。

### 静态方法
可以将静态方法理解为没有this参数的方法，即不对对象实施任何操作。

应用场景比如：

如果要访问上面提到的NextId，由于是静态变量，因此其不属于任何一个对象，
只属于Employee.NextId,通常访问它用静态方法：

```
public static int getNextId() {
    return nextId;  // 等同于 Employee.nextId 而不是this.nextId
}
```

其实上述也可以不用静态方法，但较麻烦。

又比如Math中的函数，例如`Math.pow(x, a)`
首先，并不需要`Math newsample = new Math()`这样的方式构建对象，所以`pow`应该作为静态方法。其次可以看到，参数x和a都不是Math中的域，所以可以肯定其为静态方法。

总结而言，以下两种情况用静态方法：

1. 该方法不需要访问对象状态，不需要this(例如上述pow(x, a))
2. 该方法需要访问类的静态域(例如上述getNextId())

#### 静态方法的调用形式与非静态有区别
例如，如果MyClass里有静态方法`static int func(MyClass a, MyClass.b)`
调用格式是`MyClass.func(a, b);`
而如果是非静态方法，调用格式是：
`a.func(b)`

#### C++注释-Java与C++的区别
对于静态域和静态方法，Java的意义与C++相同。只是格式上不同，例如Java的`Math.PI`在C++中是`Math::PI`。

static的历史：
起初C用static指代退出了一个块语句后依然存在的局部变量。
而来C又赋予第二个意义，表示不能被其他文件访问的全集变量和函数，即强调私有特性
后来C++赋予的第三个意义又不一样，指那些**属于类但不属于该类对象的变量和函数**，Java也是这样的意义。

### 工厂方法
略

### main方法
如上所述，静态方法不需要一个对象实例就可以调用。

main是静态方法，main方法不对任何对象进行操作。
更深入地说，在启动程序时，还没有任何对象。

**每一个类都可以有一个main方法，这是常用的对类进行单元测试的技巧**。

例如，假设一个Employee.java文件，需要对里面的Employee类进行测试，
可以直接在其中创建一个main方法。然后`java Employee`即可，
毕竟Java主要看用到什么类再链接编译相应的类文件，所以多个main没问题。
IDE也可以自动检测那些类中含有main方法，让你主函数文件编译入口。

如上所述每个类文件都只能有一个同名的public类。

## 方法参数
通常关于参数调用主要有**值传递call by value**和**引用传递call by reference**。（C++还有call by pointer，但此处不展开，更多内容参见[引用与指针的区别](http://www.cnblogs.com/tracylee/archive/2012/12/04/2801519.html)）

与C++不同的地方时，Java不像C++那样由程序员决定采用按何种方式传参。

Java默认的传递方式为：

* 基本数据类型（数字，布尔）按**值传递**
* 对象，按**引用传递（误**（实际是值传递，传地址）

但是，实际上，一定要理解对象参数也是值传递的，传递的是地址的值，而且要理解的是，每次函数调用，都是copy值

例如：

```java

void swap(MyObject x, MyObject y) {
    // ...
}

swap( InstanceX, InstanceY  ); // 并不会交换两对象
```

以上swap是无效的，这是因为函数传递进实参后，是copy 给 x 以及 copy给 y。
copy的是地址值，也就是说x指向InstanceX对象，y指向InstanceY 对象。
这时swap只是交换了x和y，将y指向InstanceX对象，x指向InstanceY 对象。

所以，**Java中所谓对象引用进行的是值传递**。

总结如下：

* 一个方法不能修改基本数据类型
* 一个方法可以修改对象的状态
* 一个方法不能让对象参数引用到另一个参数

## 对象构造
### 重载
跟C++一样，略

### 默认域初始化
也就是说没有给定对象的域的情况下，域将取默认值，数值型取0，boolean取false，对象引用取null。

然而，**一定要初始化对象的域，为了可读性**。

域/局部变量的区别：
域在类中，有默认初始化值
局部变量在方法中，必须初始化

### 默认构造器
同C++

如果没有为类添加构造器，则系统自动添加默认构造器，规则是将域将取默认值，
即 数值型取0，boolean取false，对象引用取null。

如果为类添加了哪怕一个构造器，则系统将不会自动添加默认构造器。

### 显式域初始化
就是类似C++中的初始化列表功能，提供在构造函数前初始化数据。
但是语法形式却不同，Java的初始化语法形式是：

```java
class Mycalss {
    // ...
    private String name = ""; // initialization in Java
    // C++ 则是用初始化列表
}
```

可以看出C++将初始化列表语法放在构造函数里，即便初始化列表发生在构造函数前。
这样处理的一个理由是，C++的Object内可以有Object（subobject），
而Java中Object内没有Object（subobject），而是Object的指针。
所以不需要初始化列表的语法。
（我个人认为都一样啊？只是C++中对象构造还要调用subobject的构造函数，Java中貌似不是）

### 利用this关键字，在构建器中调用另一个构建器
Java的这一特性很有意思，可以利用this关键字，在构建器中调用另一个构建器，例如：

```java
// 某个构建器
public Employee(double s) {
    // calls Employee(String, double)
    this("Employee #" + nextId, s);
    nextId++;
}
```

这样的话，就可以设计一个通用的构建器（例如上例中的Employee(String, double)），其他的构建器形式做成适配接口。

**C++没有这样的特性，不能在构建函数中调用另一个构建函数。**

### initialization blocks

如上所述，Java可以通过在构建器内初始化，也也可以通过直接声明时初始化（显式域初始化）。
Java还有第三种初始化方法，通过initialization blocks实现。格式如下：

```java
clss Employee {
    //方法1 构建器内初始化
    public Employee(String name, double salary) {
        this.name   = name;
        this.salary = salary;
    }

    public Employee() {
        this("", 0); //name = ""; salary = 0;
    }
    // ...
    // 方法2 声明时显式初始化
    private static int nextId = 1;

    private int id;
    private String name;
    private double salary;
    //...
    // 方法3 initialization block
    {
        id = nextId;
        nextId++;
    }
}
```

注：其实“方法2 声明时显式初始化”以及“方法 3 initialization block”的语是一个类型的，如果不考虑其在类中，风格上跟没有类的C也是一样的啊，
而且也是按照其声明顺序或者块内执行顺序初始化。
所以我觉得之所以采用这样的语法是为了照顾C语言编程风格。

#### 初始化顺序
Java中类的域初始化顺序为：

**Step1** 所有域初始化为default value，数值设为0，boolean设为false，对象设为null

**Step2** 如上声明时显式初始化或者初始化块将被执行，按照声明顺序或者块内代码执行顺序。

**Step3** 如果第一个构建器this调用第二个构建器，则第二个构建器先执行，即按照调用栈顺序。

**Step4**
在Step3的构建器顺序确定后，每个构建器代码块内的代码依次执行。

#### 静态域的初始化
通常采用方法2和方法3，即声明时初始化或者用static关键字的初始化代码块（通常用在初值需要通过较复杂的计算获得的场合），语法如下：

```java
class StaticBlock {
    private static int nextId;

    static { // 静态初始化块
        Random generator = new Random();
        nextId = generator.nextInt(10000);//获得0~9999内的随机整数
    }
}
```

所有的静态初始化语句及静态初始化块都按照类定义顺序执行。

### 对象析构与finalize方法
许多面向对象的语言，比如C++都有**显式析构**方法，用来执行清理代码。常见操作是回收分配给对象的存储空间。

Java有自动GC。不需要人工回收内存空间，所以Java不支持析构器。

但如果Java与外部交互，例如文件或者使用了系统资源的另一个对象句柄，就需要考虑回收与再利用了。

这种情况则为类添加`finalize`方法。
finalize方法将在垃圾回收器清除对象前调用。

在实际应用中，**不要依赖于finalize方法回收任何短缺资源**。
原因是很难知道GC什么时候回收，那么这个资源如果比较紧俏，会有很多等待时间。

#### 主动释放对象的方法
退出前释放对象
用`Runtime.addShutdownHook`添加关闭钩子，不推荐用`System.runFinalizersOnExit(true)`

如果某个资源需要使用完毕后立即关闭，用`close`方法。

## 包 package
用于管理代码库，例如`java.lang`, `java.util`, `java.net`等都是标准的Java类库。

**使用包的主要原因是确保类名的唯一性**，

包通常可以用嵌套的组织方式，例如`java.util`和`java.util.jar`。但从编译器的角度来说，嵌套包之间没有任何关系，都是各自独立的集合。

### 类的导入
可使用的类包括所属包中的所有类，以及其他包中的公有类。

导入包的方式比如`import java.util.Date`, 就导入了util中的特定类，
也可以导入所有类（整个包），如`import java.util.*`
而且导入所有类也没有任何负面影响。

#### C++注释
Java中的`import somepackage.*`类似于C++中的`using namespace somepackage;`
即Java中的包与C++中的命名空间较像，而不是`#include`，
C++中之所有必须使用`#include`加载外部特性是因为C++编译器无法查看任何其他文件的内部，只是告诉它的地址。
而Java对于文件的查看由它特定的规则。

### 静态导入
import不仅可以导入类，还有**静态方法**和**静态域**

例如，可以`import static java.lang.System.*;`,
就可以直接使用`out.println("Hello world");`

### 将类放入包中
要想将类放入包中，就需要将包名放在源文件开头，例如：

```java
package com.horstmann.corejava

public class Employee {
    // ...
}

// 这样就可以用 com.horstmann.corejava.Employee 调用
```

而且该文件必须放在对应的文件夹内，即放在`./com/horstmann/corejava/Employee.java`中。

如果没有在开头设置包，则是放在default package中，相当于C++中的没有在namespace中。

### 包作用域
`public`标记部分可以被任意的类使用
`private`标记的部分只能被定义它的类使用
没有`public`和`private`标记的类、方法或变量可以被同一个包（即同一个文件夹下）中的所有方法访问。

因为如果没有private修饰，可以通过向包中添加自己的代码的方式破坏封装性，为了解决这个问题，可以用包密封（package sealing）机制解决。

## 类路径
除了将类存储在文件系统的子目录，并于包名匹配外，
还可以将其存储在JAR（Java归档）文件中，JAR文件实际上是一个ZIP压缩包，因此可以用zip程序查看。压缩包既节省又可以改善性能。

一个JAR文件中可以包含多个类名和子目录（多个子目录那就可以有多个包了）

为了使类能够被多个程序共享，需要以下几点：

1. 如果共享一个文件夹，则将类放在一个基址文件夹的目录中（或者对应放在子文件夹中），例如`/home/user/basedir/com/horstmann/corejava/Myclass.java`
2. 如果共享一个JAR文件，则将JAR文件放在一个目录中,例如`/home/user/archives/archive.jar`
3. 设置类路径,WIN中用分号分隔`;` UNIX中用冒号分隔`:`。例如在UNIX中`/home/user/basedir:.:/home/user/archives/archive.jar`
分号分隔了三个路径，第一个是文件夹基址，第二个点`.`代表当前目录，第三个是JAR文件地址。

设置类路径的命令行方式是：

```bash
~ $ java -classpath /home/user/basedir:.:/home/user/archives/archive.jar Myprog
```

此外，运行时库(`rt.jar`以及`jre/lib`、`jre/lib/ext`目录）会默认包含，自动搜索，因此无需将其显式列在类路径中。

也可以设置`CLASSPATH`环境变量。

## 文档注释
JDK有一个很有用的工具是javadoc，可以由源文件自动生成HTML文档，
javadoc将从以下地方获取文档信息：

* 包
* 公有类与接口
* 公有的和受保护的构造器及方法
* 公有的和受保护的域

应该为以上特性添加注释`/** ... */`，并放在所描述特性前面。

注释`/** ... */`中有两个要素：

1. **标记**，以@开头，例如`@auther`、`@version`等
2. **自由格式文本**，紧跟着**标记**之后的文本，可以用HTML语法，
例如`@description 这是<em>强调</em>`

自由格式文本第一句应该是一个概要性的句子，javadoc自动将该句取出，形成概要页

注意：如果自由文档中有其他链接，例如图像文件`<img>`，较好的习惯是将这些链接文件放入`doc-files`文件夹中。javadoc将自动拷贝该文件夹即其中的文件到文档目录中。

### 类注释
可以对类进行描述，例如：

```java
/**
 * A <code>Card</code> object represents a playint card,   ---该句生成概要页
 * such as "Queen of Hearts". A card has a suit(Diamond,
 * Heart, Spade or Club) and a value (1 = Ace, 2 ... 10,
 * 11 = Jack, 12 = Queen, 13 = King)
 */
public class Card {
    // ...
}
```

### 方法注释
示例如下：

```java
/** 
 * Raise the salary of an employee
 * @param byPercent the percentage by which to raise the salary
 * (e.g. 10 means 10%)
 * @return the amount of the raise
 */
```

除了通用标记外，还可以有以下标记：

##### @param 变量描述
用于参数的描述，一个方法的所有@param标记必须放在一起

##### @return 描述
返回描述

##### @throws 类描述
表示这个方法可能抛出异常。

### 域注释
只需要对公有域（通常指静态常量）建立文档，例如：

```java
/** 
 * The "Hearts" card suit
 */
public static final int HEARTS = 1;
```

### 通用注释

#### 可用于类文档中
##### @author 姓名
作者条目

##### @version 文本
版本条目

#### 可用于所有文档注释
##### @since 文本
将产生一个since（始于）条目，说明在哪个版本引入了该特性，例如:
`@since Java SE 7`、`@since version 1.7.1`

##### @deprecated 文本
将对类、方法或者变量添加一个不再使用的注释，即说明这个不再使用，而是被新的替换，例如：`@deprecated Use <code> setVisible(true) </code> instead`。
用新的`setVisible(true)`方法代替本方法。

##### @see 引用
即可以用HTML格式引用，例如`@see <a href="www.baidu.com">百度一下</a>`。

也可以是内部类、方法、域的引用，
格式是 `@see package.class#feature label`
例如：`@see com.horstmann.corejava.Employee#raiseSalary(double)`引用raiseSalary(double)

也可以是用引号括起来的纯文本而没有链接，
例如`@see "more description is here"`，那么javadoc将这段文字放入`see also`部分。

一个特性可以用多个`@see`标记，但这些标记必须放在一起。

### 包与概述注释
以上可以实现类、方法和变量的注释，
但如果想要包注释，则需要为每个包目录中添加一个单独的文件，有以下两种选择：

1) 提供一个以`package.html`命名的HTML文件，该文件中的`<body> .. </body>`部分将被取出作为包描述

2) 提供一个以`package-info.java`命名的java文件。该文件中包含`/** */`注释。

还可以为所有文件制作一个概述性的注释，名为`overview.html`，并放在包含所有源文件的父目录中（相当于github中的README.md）
该文件中的`<body> .. </body>`部分将被取出，作为导航栏中的Overview那一栏中的内容。

### 注释的生成
命令如下：

```bash
~ $ javadoc -d docDirectory nameOfPackage
```

也有其他方式，不详述。

## 类设计的tips

1. 一定要保证数据私有化
2. 一定要对数据初始化
3. 不要在类中使用过多基本类型，即可以试着把多个基本类型再划分为一个类
4. 不是所有域都需要独立的访问器和设置器
5. 将职责过多的类进行分解
6. 类名和方法名要能体现它们的职责

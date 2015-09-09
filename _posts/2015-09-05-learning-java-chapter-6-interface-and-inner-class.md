---
layout: post
title: "Java学习 第6章 接口与内部类"
description: "《Java核心技术1 基础知识》《Core Java for the impatient》笔记"
keywords: Java, 语法, 笔记, 接口, 回调, 内部类, 代理
category: Java
tags: [Java, 接口, 代理]
---

## 接口
接口不是类，（虽然初看上去就是把`class`关键字替换成了`interface`关键字）
而是对类的一组需求描述，可以将接口理解为一个**更抽象的抽象类**，
或者说**给具体的实现限定了规则**，必须拥有某个方法的实现。
与抽象类比较起来，更抽象的地方在于：

1. 一个子类只能继承一个抽象类（虚类），但能实现多个接口；（所以Java多继承用接口实现）
2. 一个抽象类可以有构造方法，**接口没有构造方法**；（因为接口本就不是一个类）
3. 一个抽象类中的方法不一定是抽象方法，即其中的方法可以有实现（有方法体），接口中的方法**都是抽象方法**，不能有方法体，只有声明；（接口都是抽象的方法声明）
4. 一个抽象类可以是public、private、protected、default,接口只有public;（因为接口是给别人用的模板啊，当然是public的）
5. 一个抽象类中的方法可以是public、private、protected、default，接口中的方法只能是public和default

接口中**所有方法自动属于public**，因此接口**不必声明是public方法**。

接口可以含有常量，但不能有实例域，方法不能有具体实现。

#### 接口优点之一是支持泛型的方法
举例说明：
如果要实现一个泛型的sort方法，那么必然会有比较，即：

```Java
if(a[i].compareTo(a[j]) > 0) {
    // do sth.
}
```

但是由于Java是强类型语言，编译器必须**确定compareTo方法存在**，这样用接口标识就能保证了。

例如如果有`Arrays.sort(myClass)`的话，`MyClass`类必须`implements Comparable`接口，否则报错。

另外就要注意必须个人保证compareTo的方法的具体实现是用于比较，否则sort跟本意相去甚远。

#### 接口提供抽象，实现提供具体实现，那可否在实现中有abstract方法呢？
猜测应该不行，待研究

#### 如果涉及超类中两个不同子类的比较
如果涉及到继承，这个问题较为复杂，这就跟继承中equals方法的情形一样，处理方法也类似：根据比较的逻辑关系，将比较的接口实现放在超类中，并声明为`final`保证子类不会将其覆盖。

### 接口的特性
#### 接口不能new，但是可以用接口变量
接口不是类，尤其不能用`new`运算符实例化一个接口。
虽不能实例化，但可以声明**接口变量**。
所谓**接口变量**，必须new为一个含有该接口的类，例如：

```java
x = new Comparable(...); //ERROR

Comparable x; //Ok
//then
x = new Employee(...); //OK
```

#### 接口的检测可以用instanceof运算符
如第五章所述`instanceof`不仅可以用于类、超类，还可用于接口，即：
`if(anObject instanceof Comparable) {...}`。

#### 接口可以抽象继承
例如：

```java
public interface Moveable {
    void move(double x, double y);
}

//继承了Moveable接口特性
public interface Powered extends Moveable {
    double milesPerGallon();
}
```

#### 接口不能包含实例域和静态方法，但可包含常数
跟接口中的方法默认设置为public一样，
接口中的域默认将被自动设置为public static final
且Java语言规范建议不要写这些修饰符（指的是public、static、final）。

例如：

```java
public interface Powered extends Moveable {
    double milesPerGallon();
    double SPEED_LIMIT = 95//这里不是变量！而是public static final 常数
}
```

#### 超类只有一个，接口可有多个
例如`class Employee implements Cloneable, Comparable`。

### 接口与抽象类，多继承与接口的比较
简单来说，接口就是让Java也具备多继承的能力。

对于C++本来支持多继承，所以无需接口，而Java认为多继承会让语言本身变得复杂，效率也会降低。

实际上，接口提供了多继承的大多数好处，同时还能避免多重继承的复杂性与低效。

C++的多继承带来了诸如**虚基类**、**控制规则**、**横向指针类型转换**等复杂特性。

## 对象clone
### 浅拷贝与深拷贝
**Java默认的clone方法是浅拷贝**，例如：

```java
Employee other = origin.clone();
```

虽然clone确实复制出两份一样的Employee对象，other与origin指向不同对象。
但是，Employee.name因为也是引用String，所以other.name与origin.name指向同一个对象。（好在String对象是不可变的）

所以，浅拷贝指的是被拷贝对象中的引用类型并没有递归地clone。

#### 如果对象中的引用对象可变，最好用深拷贝
涉及到拷贝时，一定先做以下判断：

1. 是否不应该用clone方法
2. 如果要用clone方法，默认clone方法是否满足要求
3. 默认的clone方法是否能够通过调用可变子对象中的clone方法得到修补

如果确定了步骤1的结论是需要用clone方法的话，就必须：

1. 实现Cloneable接口
2. 使用public访问修饰符重新定义clone方法。

以下直接用实例说明更合适：

```java
package clone;

import java.util.Date;
import java.util.GregorianCalendar;

public class Employee implements Cloneable {
    private String name;
    private double salary;
    private Date hireDay;

    public Employee(String n, double s) {
        name = n;
        salary = s;
        hireDay = new Date();
    }

    //注意要用public，否则只能用在类、子类和包中
    //Object.clone是protected，但所有类都派生自Object所以没问题
    public Employee clone() throws CloneNotSupportedException {
        //如果这里含有没有实现Cloneable接口的对象，
        //例如假设Date没有实现Cloneable接口
        //则会抛出CloneNotSupportedException异常

        // call Object.clone()
        Employee cloned = (Employee) super.clone();

        // clone mutable fields
        // 因为Date是可变的类，所以要实现深拷贝，必须在此指定其为拷贝而不是赋值
        cloned.hireDay = (Date) hireDay.clone();

        return cloned;
    }

    public void setHireDay(int year, int month, int day) {
        Date newHireDay = new GregorianCalendar(year, month - 1, day).getTime();
        hireDay.setTime(newHireDay.getTime());
    }

    public void raiseSalary(double byPercent) {
        double raised = salary * byPercent / 100;
        salary += raised;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "name='" + name + '\'' +
                ", salary=" + salary +
                ", hireDay=" + hireDay +
                '}';
    }
}
```

如果一个对象需要克隆，但没有实现Cloneable接口，会产生一个检验异常（checked exception）。clone应用也并非想象中那么普遍，标准类库中，只有约5%的类实现了clone。

所有数组类型均包含一个clone方法。

Java中还有另一个克隆对象的机制，使用序列化功能，容易实现但效率低。

即使clone的默认浅拷贝能够实现需求，也要通过Cloneable接口重新实现。只需调用`super.clone()`即可自动生成，并选择是否用instanceof还是getClass，所以代码省略。

## 接口和回调
回调（callback）是常用设计模式，可以指出特定事件发生时应该采取的动作。
例如单击鼠标，点击按钮等。
这里有待补充。

## 内部类
内部类（inner class）是定义在另一个类中的类，
为什么需要内部类呢？以下是原因：

* 内部类方法可以访问该类定义所在作用域中的数据，包括私有数据
* 内部类可以对同一个包中的其他类隐藏起来
* 当想定义一个**回调函数**，但不想编写大量代码时，使用**匿名（anonymous）内部类**比较便捷。

首先需要明白的是，内部类与外部类是相互独立的类关系，也就是说外部类内存并不直接包含内部类（如果只有声明类而没有定义对象的话），
内部类和外部类只是一种作用域或者说可见性上的关系，详情见如下补充参考中的内容。

另外只有内部类才可以是私有类，这样就仅对外部类可见了。

#### C++的嵌套类与Java的内部类
C++的嵌套类包含在外围类的作用域中。
例如如下典型例子，一个链表类定义了一个存储节点的类和一个迭代器

```cpp
class LinkedList {
public:
    class Iterator { // 嵌套类，迭代器
    public：
        void insert(int x);
        int erase();
        // ...
    };

private: //因此Link对外不可见
    class Link {
    public: //这里数据是public也是安全的，因为Link是定义在private下的
    //这样做的优点是在LinkedList内除Link外的地方可以方便得直接用以下成员变量，
    //而不需诸如getData()、setData()
        Link* next;
        int data;
    };

};
```

嵌套有两个好处：

1. 命名控制，例如上述迭代器LinkedList::Iterator，这样就不会与其他迭代器命名冲突。但在Java中，这个并不重要，因为Java包提供了命名空间类似的功能。
2. 访问控制，参考如上关于C++内部类的可见性规则。

Java内部类还有另一个重要功能，内部类对象有一个**对外围类对象的隐式引用**。通过这个外围类对象指针，可以访问外围类对象的全部状态。（C++外围中的成员对内部类全部可见，所以感觉并不需要这个指针）

##### 补充参考：
[C++ 内部类、嵌套类、局部类](http://blog.163.com/aobbcn@126/blog/static/4891041620104207376425/)
>C++内部类与java内部类最大的区别是：
>
>* C++的内部类对象**没有**外部类对象的指针，**不能访问外部类对象的非静态成员**（完全可以访问啊，为什么不能访问？）
>* Java的非静态内部类对象**有**外部类对象的指针，**能访问外部类对象的非静态成员**。

[C++内部类](http://blog.csdn.net/hihui/article/details/4822412)
测试g++的规则如下：（这个规则还是很明确的）
public修饰的内部类与private修饰的内部类描述的是对外的可见性，对外层嵌套类而言都是可见的。外层嵌套类中的成员对内部类则是全部可见的。

内部类中的private成员和public成员则描述的是对外层嵌套类的可见性。private不可见，需要get、set；public则可见，在外层嵌套类中可以直接用。但private内部类的public成员对外依然是不可见。

### 使用内部类访问对象的域
#### Java的内部类含有隐藏的指向外部类指针
如上所述，Java的非静态类有外部对象的指针，这是因为，外部类对象对于内部类而言可见，可以直接引用。
那么引用外部类中的域的话，Java就用一个默认的隐含的指向外部类的指针实现。在内部类的构造函数中隐藏实现，示意性的代码如下：

```Java
// 内部类构造指向外部类的指针的示意。
public InnerClass(OuterClass outer_class) { //内部类的构造函数
    outer = outer_class; //用一个指针（假设名为outer）指向外部类；
}
```

### 内部类的特殊语法规则
也就是如上所述的outer指针，
当然实际并没有名为outer的指针，而是用`OuterClass.this`的方式引用，感觉该语法用的场合还不多，暂且略过。

### 内部类是否有用、必要和安全
内部类的一大特点是可以访问外围类的私有数据，而如果是两个独立的类，则需要get、set。
由于内部类拥有访问特权，所以比常规类功能更强大。


另一个问题安全性问题，
内部类与外部类对于虚拟机而言跟普通类没有区别，只是编译器根据内部类的语法语义作了手脚。
例如内部类访问外部类的私有域时，编译器实际上为了内部类访问外部类提供了诸如`access$0(OuterClass)`这样获得访问权限的方法，这个如果被黑客利用将带来安全性问题，当然，这需要高超的技巧，难度较大。

### 局部类
可以在方法中定义类，作用域仅此方法，称为局部类。
局部类不能用public、private等访问说明符声明。
局部类对方法之外完全隐藏。

### 由外部方法访问final变量
这部分未来很有可能变动，所以略过

### 匿名局部类
也就是函数中的局部类可以不必按照 声明-> new对象 的顺序，将声明和new对象混在一起。

由于构造器需要类名，所以匿名类不能有构造器，取而代之的是将构造器的参数传递给超类构造器。

好处是代码更精简。

感觉很多局部类的场合可以被匿名类代替。

### 静态内部类
有时候，使用内部类只是为了把类隐藏在另一个类内部，而无须内部类引用外部类中的域。这时候可将内部类声明为static，以便取消产生的引用（即外部类指针，假设名为outer）

那么，此时用内部类最重要的好处类似于namespace。

声明在接口中的内部类自动作为static和public类。

## 代理
#### 代理是在运行期执行未知对象的方法
我们知道反射是在运行期获取未知的对象的信息
而代理则更进一步，可以运行未知对象的方法。

利用代理可在运行期创建一个实现了给定接口的新类。
这个功能在编译前无法判断需要实现哪个接口时才有必要使用。
此技术对应用程序需求很少，但系统设计需要（跟反射一样）。

假设没有代理，那么运行期执行未知对象的方法将变得很困难，
可以想到的步骤是：

1. 首先通过反射获得对象信息，然后按照语法输出成一个.java文件
2. 编译该文件并添加到程序中

所以，感谢代理！

具体代理如何实现讲解起来十分困难，以如下代码所谓例子演示：
如下代码中代理的作用是：

1. 任意一个对象都可以通过`proxy(anyObject)`构造的代理代替，并执行指令，例如如下的anyObject即为`Integer value`。
2. 可以在执行指令的时候附加我们想要的其他命令，比如如下的打印指令（通过实现InvocationHandler接口的invoke方法）

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;
import java.util.Random;

public class ProxyTest {
    public static void main(String[] args) {
        Object[] elements = new Object[1000];

        //fill element with proxies for the integer 1...1000
        for (int i = 0; i < elements.length; i++) {
            Integer value = i + 1;

            //运行期才知道对象是Integer
            //important
            InvocationHandler handler = new TraceHandler(value);
            /**
             * null 默认类加载器
             * Class对象数组，每个元素都需要实现的接口
             * 一个调用处理句柄，也就是实现InvocationHandler接口的那个类
             */
            //important
            Object proxy = Proxy.newProxyInstance(null, new Class[]{Comparable.class}, handler);
            /**
             * 这样将elements和代理绑定在一起，无论elements[i]调用了什么方法，
             * 比如二分查找中的compareTo方法,即调用elements.compareTo()--proxy.compareTo方法
             * 则会打印该方法（即如下TraceHandler.invoke的实现）
             */
            elements[i] = proxy;
        }



        //Construct a random Integer
        Integer key = new Random().nextInt(elements.length) + 1;

        //search for key
        int result = Arrays.binarySearch(elements, key);

        //print match if found
        if (result >= 0) {
            System.out.println(elements[result]);
        }

    }
}


class TraceHandler implements InvocationHandler { //important
    // 代理必须要InvocationHandler接口并实现invoke方法
    private Object target;

    public TraceHandler(Object target) {
        this.target = target;
    }

    //invoke方法的实现
    public Object invoke(Object proxy, Method m, Object[] args) throws Throwable{
        //print implicit argument
        System.out.print(target);
        //print method name
        System.out.print("." + m.getName() + "(");

        //print explicit arguments
        if (args != null) {
            for (int i = 0; i < args.length; i++) {
                System.out.print(args[i]);
                if (i < args.length - 1) {
                    System.out.print(", ");
                }
            }
        }
        System.out.println(")");

        //invoke actual method
        /**
         * 以上是添加的打印结果，而如下方法则是必须的，即调用某个方法
         */
        return m.invoke(target, args);  //important
    }
}
```

注意如上`//important`部分是实现代理功能中比较重要的点。

总的来说，如果对象编译期未知，需要反射或者代理
如果需要执行任意对象的指令的时候再附加一些指令，则需要代理并在InvocationHandler接口中invoke方法进行实现。

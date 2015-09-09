---
layout: post
title: "Java学习 第5章 反射"
description: "《Java核心技术1 基础知识》《Core Java for the impatient》笔记"
keywords: Java, 语法, 笔记, 反射
category: Java
tags: [Java]
---

## 反射
能够分析类的能力的程序称为**反射(reflective)**。
作用当然是用于动态分析代码咯。
缺点是安全性差，有可能在Debug阶段运行良好，但到客户虚拟机上却出错，
所以也提出应用程序中尽量不用反射。

### Class类
程序运行期间，Java运行时系统始终为所有对象维护一个被称为**运行时类型标识**。
该标识跟踪每个对象所属类信息。

Java中可以通过专门的Class类访问这些信息，Class类中保存这些信息。

#### 获得Class类的三种方法
例如：

```java
Employee e;
//获得Class的方法1： getClass()方法将返回一个Class类型的实例，
Class cl = e.getClass();

//获得Class的方法2： 通过forName静态方法
String className = "java.util.Date" //如果不是类将会throw异常
Class cl = Class.forName(className);

//获得Class的方法3： T.class
Class cl = int.Class;
```

#### 利用反射分析类的例子
在java.lang.reflect包中有三个类分别是：Field、Method、Constructor分别保存了类的域、方法、构造器信息。
如下例子通过反射获得了类的Field、Method和Constructor、超类等全部类信息，
这也是反射机制最重要的内容。
具体参见《Java核心技术I》P196 - P201。不详述。

```java
package reflection;

import java.lang.reflect.*;
import java.util.Objects;
import java.util.Scanner;

/**
 * This program uses reflection to print all features of a class.
 * this code example is from [Core Java 1]#reflection
 */
public class ReflectionTest {
    public static void main(String[] args) {
        String name;
        if(args.length > 0)
            name = args[0];
        else {
            Scanner in = new Scanner(System.in);
            System.out.println("Enter class name(e.g. java.util.Date):");
            name = in.next();
        }

        try {
            //print class name and superclass name(if != Object)
            Class cl = Class.forName(name);
            Class supercl = cl.getSuperclass();
            String modifiers = Modifier.toString(cl.getModifiers());

            //print modifiers
            if(modifiers.length() > 0)
                System.out.print(modifiers + " ");

            //print name
            System.out.print("class" + name);
            //print super class
            if (supercl != null && supercl != Objects.class) {
                //if exist and not the root class Objects
                System.out.print(" extends " + supercl.getName());
            }
            System.out.print("\n{\n");
            printConstructor(cl);
            System.out.println();
            printMethods(cl);
            System.out.println();
            printFields(cl);
            System.out.println("}");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.exit(0);
    }

    /**
     * Print all constructor of a class
     * @param cl a class
     */
    public static void printConstructor(Class cl) {
        Constructor[] constructors = cl.getConstructors();
        for (Constructor c : constructors) {
            String name = c.getName();
            System.out.print("    ");//tab空四格
            String modifiers = Modifier.toString(c.getModifiers());
            if(modifiers.length() > 0)
                System.out.print(modifiers + " ");//修饰符 如final static等
            System.out.print(name + "(");

            //print parameter types
            Class[] paramTypes = c.getParameterTypes();
            for (int j = 0; j < paramTypes.length; j++) {
                if( j > 0 ) System.out.print(",");
                System.out.print(paramTypes[j].getName());
            }
            System.out.println(");");
        }
    }

    /**
     * Prints all methods of a class
     * @param cl a class
     */
    public static void printMethods(Class cl) {
        Method[] methods = cl.getDeclaredMethods();
        for (Method m :methods) {
            Class<?> retType = m.getReturnType();
            String name = m.getName();

            System.out.print("    "); //tab 4个空格
            //print modifiers
            String modifiers = Modifier.toString(m.getModifiers());
            if(modifiers.length() > 0)
                System.out.print(modifiers + " "); //修饰符 如final static等
            //print return types and method name
            System.out.print(retType.getName() + " " + name + "(");

            //print parameter types
            Class<?>[] paramTypes = m.getParameterTypes();
            for (int j = 0; j < paramTypes.length; j++) {
                if(j > 0) System.out.printf(",");
                System.out.print(paramTypes[j].getName());
            }
            System.out.println(");");
        }
    }

    /**
     * Prints all fields of a class
     * @param cl a class
     */
    public static void printFields(Class cl) {
        Field[] fields = cl.getDeclaredFields();

        for (Field f :fields) {
            Class<?> type = f.getType();
            String name = f.getName();
            System.out.print("    "); //tab, 4 spaces
            String modifiers = Modifier.toString(f.getModifiers());
            if(modifiers.length() > 0)
                System.out.print(modifiers + " ");
            System.out.println(type.getName() + " " + name + ";");
        }
    }
}
```

### 反射中容易出现异常
由于反射很有可能会出现异常，所以通常需要try-catch配合捕获异常，比如`ClassNotFoundException`异常等等。

### 运行时使用反射分析对象
如上可以获得域名，域类型，域的修饰符（static、public等）但没有获得域值。
如需获得域值用`aField.get(aClass)方法`，比如：

```java
Class cl = getClass(Employee[1]);
Field fieldName = cl.getDeclaredField("name"); //获得名为name的域
Object objName = fieldName.get(Employee[1]); //获得雇员名（即name域的值）
//get函数有些不漂亮，fieldName来自Employee[1]却还需要Employee[1]作为get的参数
//这样有些耦合在一起了，猜测可能是考虑到有些域是间接继承的，这样更明确吧
```

#### 用setAccessible()方法使get方法获得访问域值的权限
如上由于name域是私有的，所以要用get方法访问还需要用`field.getAccessible(true)`提权。

### 使用反射编写泛型方法
有时候为了编写适用于多种Class类型的方法会用到获取Class实际类型的办法实现。

### 调用任意函数
#### C/C++的函数指针对应Java中的Method对象
在C/C++的函数指针对应Java中的Method对象中可以用函数指针执行任意函数。
而Java设计者则认为方法指针很危险，所以Java没有提供方法指针，
而是采用接口的方式。
反射机制则可以实现调用任意方法。（注：用代理才上调用任意方法吧？。。。）

我猜测过程就是，先通过反射获得一个类类型，然后new一个一样类型的类，并在类中调用。

而答案是... Java的Method类提供一个invoke方法，允许调用包装在Method对象中的方法。

而用Method类的invoke方法比较慢，并不是最好的选择，建议使用接口进行回调，代码将更快，更易于维护。


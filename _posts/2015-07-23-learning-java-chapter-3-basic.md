---
layout: post
title: "Java学习 第3章 基础"
description: "《Java核心技术1 基础知识》《Core Java for the impatient》笔记"
keywords: Java, 语法, 笔记
category: Java
tags: [Java]
---

## 前言
写此文档目的在于快速掌握Java核心内容，所以对基本的内容不详细复述，直接看书的即可。为了快速学习内容，我只突出两点，一个是Java和C++的区别，如果可以的话深入分析想两者区别的深层原因；另一个是Java特有的特性会着重描述。

## 搭建环境
详细的搭建过程网上有很多相关内容，不详述，只简要提一下：
命令行还是没有IDE好使，所以推荐用IDE
主流的IDE是：
[Eclipse 下载地址](http://www.eclipse.org/downloads/)
[Netbean 下载地址](https://netbeans.org/downloads/)
[Intelij IDEA 下载地址](https://www.jetbrains.com/idea/download/)
我尝试用的是Intelij IDEA，用了一段时间，发现实在是太爽了，尤其智能补全，做的相当出色，
具体参考：[十大Intellij IDEA快捷键](http://blog.csdn.net/dc_726/article/details/42784275)

下载安装IDE后只能算有了架子，可以当编辑器用，
要想编译Java还需要装JDK（Java Development Kit）
[JDK 下载地址](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

需要说明的是JDK的bin路径还得手动添加到系统环境变量`$PATH`中去，所以它的安装路径没有空格比较好，所以windows下装在program file不如单独见一个文件夹安装。

安装完毕后，也需要告诉IDE Java的JDK在哪里。

此外，有Intellij IDEA的[Solarized 配色风格](https://github.com/jkaving/intellij-colors-solarized)

Intellij IDEA确实好用，实用技巧参见：
[十大Intellij IDEA快捷键](http://blog.csdn.net/dc_726/article/details/42784275)

需要说明的是Intellij IDEA虽然很好用，但非常吃性能，电脑不好卡成翔，如需要有所改善可以参考：

<http://www.zhihu.com/question/35203203>

## Java基础
### 注释
Java除了C++常用的`//...`单行注释和`/* ... */`多行注释外，
还提供可**用于自动生成文档**的注释格式，`/** ... */`，例如可以按如下：

```java
/**
 * Description
 * Author:
 * Date:
 * License...
 */
```

### 数据类型
Java一共8个基本数据类型：
#### 4个整型
byte、short、int、long，分别是8位、16位、32位和64位

##### 整型常数
长整型为 12345L
16进制数 0x12345
8进制 012345（尽量不用，容易混淆）

此外，SE 7还加入了2进制
2进制 0b1001 = 9
同时，为了解决数值太长，分不清位数的问题，可在数值间加下划线
例如`0b100110011001`可以写为`0b1001_1001_1001`这样就很清晰了。

#### 2个浮点数
float、double分别是32位和64位
全部用IEEE 754 浮点格式。
关于该浮点格式参考：[CSAPP要点总结 第2章 信息的表示和处理 part3 浮点数]({% post_url 2015-07-10-csapp-chapter-2_3 %})

##### 浮点常数
float类型为：3.14F
double类型为：3.14D 或者 默认的 3.14
IEEE 754还有正无穷、负无穷和NaN
在Java中
正无穷用`Double.POSITIVE_INFINITY`表示
负无穷用`Double.NEGATIVE_INFINITY`表示
NaN用`Double.NaN`表示。

需要注意，两个NaN之间不相等，
所以判断x是否为NaN， x == Double.NaN 是错的，应该用 Double.isNaN(x)

浮点常量注意舍入误差，如对精度要求极高，例如金融领域，应该用BigDecimal类

#### 1个char类型
`'A'`,单引号表示字符，`"A"`,双引号表示字符串。
char = 字符 （正确）
char = 整数 （正确）
char = \u数字 （unicode 正确）
需要注意的是Java中，char是16位，用Unicode编码格式表示。
然而也用Unicode是历史原因，因为当时认为16位共65536个字符足够容忍全世界的语言了，然而，却低估了中、日、韩等象形文字的数量，因此不得不扩充，对于扩充的字符，就需要两个char的空间。

强烈建议不用char类型，

具体参见： <http://blog.csdn.net/happylee6688/article/details/33306069>

```
那么，到底为什么java里不推荐使用char类型呢？其实，1个java的char字符并不完全等于一个unicode的字符。char采用的UCS-2编码，是一种淘汰的UTF-16编码，编码方式最多有65536种，远远少于当今Unicode拥有11万字符的需求。java只好对后来新增的Unicode字符用2个char拼出1个Unicode字符。导致String中char的数量不等于unicode字符的数量。

然而，大家都知道，char在Oracle中，是固定宽度的字符串类型（即所谓的定长字符串类型），长度不够的就会自动使用空格补全。因此，在一些特殊的查询中，就会导致一些问题，而且这种问题还是很隐蔽的，很难被开发人员发现。一旦发现问题的所在，就意味着数据结构需要变更，可想而知，这是多么大的灾难啊。
```

#### 1个boolean类型
boolean类型有两个值，true和false，注意，**整型和boolean之间不能隐式转换**。

>虽然 Java 虚拟机定义了 boolean 这种数据类型，但是只对它提供了非常有限的支持。在
Java 虚拟机中没有任何供 boolean 值专用的字节码指令，在 Java 语言之中涉及到 boolean
类型值的运算，在编译之后都使用 Java 虚拟机中的 int 数据类型来代替。
  Java 虚拟机直接支持 boolean 类型的数组，虚拟机的 newarray 指令可以创建这种数组。
boolean 的数组类型的访问与修改共用 byte 类型数组的 baload 和 bastore 指令

同样，关于boolean占1 byte还是1 bit也是众说纷纭，具体要看虚拟机实现。


#### 与C++的比较
##### （1） 大小确定
首先，Java中对于变量的大小都是确定的，而不像C++那样，只规定变量的最小范围。
C++这样做目的估计是兼容C，而C这样做，主要是为了适应很多古老的设备，
有些16位的设备可能int表示16位更好。我觉得看似方便，扩大了适用性，而实际上也给平台移植带来困难。给编程带来了本不需要考虑的困扰，所以我觉得Java这方面更好一点。

##### （2） 二进制数
Java新版增加了0b1111_1111这类二进制，且可插入下划线，非常人性化。

##### （3） 无unsigned类型
Java没有unsigned类型，避免了隐式转换可能带来的bug，还有其他的C++的隐式转换被取消掉，这样更好。总体而言，C/C++带有隐式转换更偏底层一点，因为对于寄存器而言，我知道是二进制数就够了，根本不管是不是unsigned，是char还是int。但也更容易出错。

##### （4） 无指针
Java基本类型中没有指针，内存GC管理，这样也避免了一些危险操作，当然C/C++更有灵活性一些。

### 变量
Java中的命名很有意思，比C++范围大很多，虽然也是字母+数字，必须字母开头的方式，**但`$` `_`以及unicode中任意可表示字母和数字的字符都算**。

变量的声明最好靠近第一次使用的地方。

Java中变量**必须被初始化才能被使用**。而C++没有这个要求。

Java**不区分声明和定义**，可以都看做声明；
而C/C++是区分声明和定义的。

### 常量
用关键字`final`，表示该值只能被赋值一次，例如：

```java
final double PI = 3.1415926535
```

有时候需要可以多个方法同时使用的常量，在Java中，可以用**类常量**，用`static final`修饰，
同一类的方法都可以调用该常量，如果是public类常量，则其他类方法也可以调用它
例如:

```java
public class Math {
    public static final double SQRT2 = 1.414;
}
```

在C++中常量用`const`关键字，Java中也有这个保留字，但还“保留着”没用，常量用`final`声明。

## 运算符
同样也有`+ - * / %`，其中`/`如果还有浮点数，就是浮点除法，否则是整数除法。`整数/0`将会异常,`浮点数/0`得NaN或无穷大。

Java设计哲学是可移植性，所以对于任何运算在不同机器上的结果应该一致。
然而有些比如很多Intel处理器使用80位浮点寄存器，这样就存在截断。（当然如果不溢出还是一样的结果咯）
必须在结果一致与计算效率中作出选择。
如果要严格保持浮点数计算结果一致，应该在main函数上标记`strictfp`关键字，如下：

```java
public static strictfp void main(String[] args) {};
```

### 自增自减运算
同C++

### 关系运算与boolean运算
同C++

### 位运算
C++中并没有规定右移是算术右移（高位补sign）还是逻辑右移（高位补0）。
执行中通常取效率较高的那一种。

Java对此作了严格定义。

`>>`运算为算术右移；`>>>`运算为逻辑右移。

### 数学计算
在Math类库中。
Math类为了达到更快的性能，通常调用计算机自带例程。
如果对于移植性要求更高，用StrictMath类，确保所有平台得到同样结果。

### 数值类型之间的转换
数值类型之间的转换关系图：

![数值之间的合法转换]({{ '/assets/images/CoreJava/CoreJava_3_1.JPG' }})

大致原则就是位数小的数可以往位数大的数转。
但long转float似乎是个例外。
需要注意3点：

1. 虽然int可以转float，flout不能转int，
同样，long可以转double，但double不能转long
2. boolean不能转换成其他数据类型
3. 图中虚线表示可能会存在精度损失

貌似Java的数值之间计算也存在隐式转换，

按照double, float, long, int的优先级顺序。
例如，对于`a + b;`
如果a或者b存在double类型，则是double + double
else then 存在float类型，则是float + float
依此类推。

#### 强制类型转换
也和C一样，用`(类型)变量名`的形式，例如`(int) floatnum;`

如果超过数据表示范围，则存在截断误差。

注意：尽量不要让boolean类型参合进强制转化，
可以用`boolean b = a ? 1 : 0;`代替。

### 括号与运算优先级
Java中没有逗号运算符

运算优先级：

![运算优先级]({{ '/assets/images/CoreJava/CoreJava_3_4.JPG' }})

关于优先级好难记，觉得通常无必要，需要时查看。

### 枚举
`enum`关键字，格式举例：

```java
enum Size {SMALL, MEDIUM, LARGE, EXTRA_LARGE};
Size s = Size.MEDIUM；
Size nu = null  //表示暂未设定任何值
```

## 字符串
Java字符串就是Unicode字符序列。

字符串并非Java的内置类型，而是用String类，并用双引号表示。

### 子串
用`substring(int startpos, int endpos)`方法。

### 拼接
用`+` 即可实现，非常方便，而且和JavaScript一样，对于数字转字符串很方便。

### 不可变的字符串
对于C/C++而言，字符串是字符型数组，每单个字符位是可以改变的，
然而，对于Java而言，实现字符串的变化实际上是修改了引用，例如，如果将`"Hello Jay"`改为`"Hello Jack"`,
Java的方法是：

```java
String s = "Hello Jay";
s = s.substring(0 , 8) + "ck";
```

实际上`"Hello Jay"`还是那个`"Hello Jay"`，`"Hello Jack"`是另一个字符串，只不过s引用了另一个。

这种处理方式是因为，Java设计者认为共享带来的高效率远胜过提取、拼接字符串的效率。

### 检测是否相等
用`s.equals(t)`来检测两字符串是否相等，返回true/false。
注意，s和t都可以是字符串变量或者常量。

例如：`"Hello".equals("hello")` 也是可以的，结果为false
如果要忽略大小写，用`"Hello".equalsIgnoreCase("hello")`结果为true。

不能使用`==`来判断两个字符串是否相等。

`s == t`是检测两个字符串地址是否相同。
如果两字符串地址相同，那必然两字符串相同。
但**也有相同的字符串分布在不同地址的情况**。

### 空串与Null串
空串即长度为0的字符串`""`,
可用`if(str.length() == 0)`或者`str.equals("")`检测是否是空串。

空串是Java的一个对象，
Null则是表示一个目前没有任何对象与该变量关联，在null上调用方法会出错。

### 代码点与代码单元
Java字符串是char序列，
char数据类型是一个采用UTF-16的Unicode代码点的代码单元。
Unicode多少占用一个代码单元，但也有少数占用两个，这给有些处理带来麻烦，
此处略。

### 字符串API
略

### 构建字符串

有时用`+`连接字符串的方式来构建字符串效率较低，毕竟需要另外开辟空间，
这时候可以用`StringBuilder`

```java
StringBuilder builder = new StringBuilder(); //new一个空字符串构建器

builder.append("Hello ");
builder.append("World!");

String str = builder.toString(); //将最终字符串赋给str
```

可以看到`StringBuilder`有点像一个字符list，有`append`,`delete`,`insert`方法等。

## 输入输出
### 读取输入

对于控制台输入，需要先构建一个Scanner对象，并于标准输出流`System.in`关联

```java
import java.util.*   //为了包含Scanner类

Scanner in = new Scanner(System.in); //构建一个Scanner对象，与标准输出流`System.in`关联

String oneSentence = in.nextLine(); //读取一行字符串
String firstName = in.next(); //读取一个单词字符串，空格作为分隔符
int age = in.nextInt(); //读取整数
double money = in.nextDouble(); //读取浮点数
```

Scanner类不适用于控制台读取密码，因为这样密码可见了。
Java SE 6引入Console类解决这一问题：

```
Console cons =System.console();
String username = cons.readline("User name: ");
char[] passwd = cons.readPassword("Password：");
```

密码存放在一维字符数组而不是字符串中，是为了安全，对密码处理后，因马上覆盖该数组元素。

### 格式化输出
与C语言的printf函数格式类似，且也是printf(frint format)函数。
但有些功能更强大，比如可输出散列码或者时间。
具体内容不解释，需要时查询即可。

![用于printf的转换符]({{ '/assets/images/CoreJava/CoreJava_3_5.JPG' }})

![用于printf的标志]({{ '/assets/images/CoreJava/CoreJava_3_6.JPG' }})

![日期时间转换符-1]({{ '/assets/images/CoreJava/CoreJava_3_7_1.JPG' }})

![日期时间转换符-2]({{ '/assets/images/CoreJava/CoreJava_3_7_2.JPG' }})

对于日期，需要用`new Date()`生成系统日期，
例如：

```java
System.out.printf("%1$s %2$tB %2$te, %2$tY", "Due date:", new Date());
```

其中 `%2$te, %2$tY`, 中的`2$`是参数索引，指的是该格式输入为第二个参数，即`new Date()`

也有一种简写方法，用`<` 表示上一个参数，以下函数等同于上一个：

```java
System.out.printf("%s %tB %<te, %<tY", "Due date:", new Date());
//          默认参数1 默认参书2 用上一个，即参数2 继续用上一个，即参数2
```

还有一种用格式化的地方，例如：

``` java
String message = String.format("Hello, %s. Next year you'll be %d", name, age);
```

### 文件输入输出
我们知道，想控制台输入，就构建一个Scanner对象与`System.in`关联。
对于文件输入，则构建一个Scanner对象与File对象关联，例如：

```java
Scanner in = new Scanner( Paths.get("myfile.txt") );
Scanner in = new Scanner("c:\\mydir\\myfile.txt");
```

想要写入文件，则需要构造一个PrintWriter对象。

```java
PrintWriter = new PrintWriter("out.txt");
```

需要注意的是可能会发生异常，例如用一个不存在的文件，或者文件名格式不对。
解决方法是throw IOExcetion

```java
public static void main(String[] args) throw IOExcetion
{
    Scanner in = new Scanner(Path.get("myfile.txt"));
}
```

Java 还有其他输入输出方法，例如,可以用`new File(dir, fileName)`的方式，就不一一详解了，需要时查。

## 控制流程
### 与C++的比较
Java控制流程与C/C++类似，仅少数例外：

1. 没有goto
2. break 可带标签，类似C的goto
3. 一种变形的for循环，类似C#中的foreach ---- C++2011标准已经加入此语法
4. C++中可以在嵌套块中重定义，内层定义覆盖外层，而Java不行，因为Java认为这样有可能导致程序设计错误

### 块作用域
需要注意的是，不能在嵌套块中声明同名变量，例如：

```java
{
    int n;
    \\ ...
    {
        int n; \\ error 在嵌套块中发生重定义
    }
}
```


### if-else while for循环
对于for循环，除了常规的用法，还有简化写法，用冒号表达式，例如遍历一个数组：

```java
for(variable:collection)
    statement;
```

此foreach循环相对常规写法更简介，更不易出错。

其他同C++，略

### switch
注意switch可输入类型：

1. char、byte、short或者int(以及它们的包装类Character、Byte、Short、Integer)
2. 枚举常量
3. 从Java SE7开始，也可用字符串常量

### 中断控制
Java提供带标签的break用于跳出多重嵌套循环，这点C++似乎没有（goto就可实现）。
需要注意的是只能跳出语句块，而不能跳入语句块。

永洋continue也可带标签。

## 大数值
大数值是为了解决计算精度的问题，当long，double不能解决精度问题时，可以用`java.math`中的`BigInteger`和`BigDecimal`。它们可实现任意精度。

由于Java并不提供C++那样的重载，所以不能直接用 `+ - * \`,取而代之的是如下方法：

```java
BigInteger add(BigInteger other) //       +
BigInteger subtract(BigInteger other) //  -
BigInteger multiply(BigInteger other) //  *
BigInteger divide(BigInteger other) //    /
BigInteger mod(BigInteger other) //       %

int compareTo(BigInteger other) // 比较大小，返回 正、负、0
static BigInteger valueOf(long x) //返回等于x的大整数，可当作括号用
```

对于`BigDecimal`类似，有少许不同，但暂时没必要记住细节。

## 数组
可用两种方法声明数组：

```java
int[] a;
// or
int a[];
```

多数Java程序员喜欢第一种方法，因为变量名与类型分开。

不是说Java没有声明，且必须赋初值的吗？ 这里需要调查一下，暂略

以上是声明数组，要想真正初始化一个数组，应该：

```java
int[] a = new int[100];
```

创建一个数组时：
数字数组所有元素初始化为0
boolean数组会初始化为false
对象数组所有元素初始化为null

获得数组元素个数用：`array.length`

一旦创建一个数组，不能改变其大小，如需要运行时扩展大小的数组，用**数组列表（array list）**

### 数组初始化以及匿名数组

对于数组初始化，常规方法是：

```java
int[] primes = {2, 3, 5, 7, 11, 13};
```

这样就创建了一个数组变量（或者说指针），并分配一个数组空间。

还有另一种方法，用已有的变量，重新指向另一个数组空间：

```java
primes = new int[] {17, 19, 23, 29};
//其中 new int[] {17, 19, 23, 29}为匿名数组
```

此方法，先创建一个匿名数组空间，再让已有的变量primes指向该数组，对于为释放的原数组空间无需多虑，虚拟机的GC会自动将其空间释放。

Java甚至允许数组长度为0，`new int[0]`，如果一个方法的结果为空，那么可以返回这个。

### 数组拷贝
同样用`array a = array b`的方式拷贝，
只是，对于C++是值拷贝，而对于Java则是引用，两者指向同一内存空间。

Java如果要实现值拷贝可以用：

```java
int[] copiedArr = Arrays.copyOf(Arr, Arr.length) //值拷贝

int[] copiedArr = Arrays.copyOf(Arr, 2*Arr.length)
//第二个参数控制长度，这里除了拷贝还扩展了空间
```

```java
int[] a = new int[100]; //java
```

不同于C++的：

```cpp
int a[100]; //C++
```

而是类似于C++的：

```cpp
int* a = new int[100]; //C++
```

有就是相当于动态分配在堆（heap）上的数组指针。
且Java没有指针运算，因此，不能用 a+1来指代下一个元素。

### 命令行参数
对于C++，有两个命令行参数表示长度和参数，而Java只包括`String[] args`
需要长度时用`args.length`。

此外，Java第一个参数与C++的不同，Java中的`args[0]`等同于C++中的`argv[1]`。

### 数组排序

```java
import java.util.*  //Array在util中

// some codes

int[] a = new int[10000];
// do something
Array.sort(a); //排序
```

sort方法是用优化的**快速排序算法**。快速排序算法对于大多数数据集合而言都效率较高。

其他排序算法在后续章节介绍。

排序测试首先得生成随机数，用如下方法：

```java
int r = (int) (Math.random() * n); //产生 0 ~ n-1 之间的随机数
//random() 范围为 [0, 1)
```

### 多维数组
和C++类似，构建方式包括：

```java
double[][] a = new double[M][N];

double[][] b = {
    {1, 2, 3, 5, 6},
    {3, 5, 7, 8, 7},
    {2, 5, 6, 3, 6}
};
```

需要注意的是foreach处理多维数组的格式，类似于：

```java
for(int[] row : arr)
    for(int value : row)
        //do something
```

#### 快速打印数组

```java
System.out.println( Array.toString(a) ); //针对一维数组
System.out.println( Array.deepToString(a) ); //针对多维数组
```

### 不规则数组
Java的数组看似与其他语言相似，但实际上Java没有真正的多维数组，多维数组是数组的数组。换句话说，多维数组中，一个维度里的每个元素存放着一整个数组的起始地址（引用一个数组）。
这样做的好处是：

1. 数组行交换极为高效
2. 方便构造不规则数组

不规则数组的构造方法：

```java
int[][] odds = new int[NMAX][]; //首先需要确定1维行数

//接下来为每行分配数组
for(int i = 0; i < NMAX; i++)
    odds[i] = new int[i + 1];    // 此处构造一个三角形数组
```

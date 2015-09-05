---
layout: post
title: "Java学习 第5章 继承"
description: "《Java核心技术1 基础知识》《Core Java for the impatient》笔记"
keywords: Java, 语法, 笔记, 类, 超类, 子类, Object, 泛型数组列表, 枚举类
category: Java
tags: [Java]
---

继承是复用类的方法和域，并在此基础上还可添加新的方法和域。
反射(reflection)指运行期发现更多类及其属性的能力。

## 类、超类、子类
继承用关键字`extends`,如下：

```java
class Manager extends Employee {
    // ...
}
```

在Java中，所有继承**都是公有继承**，而没有C++中的private继承和protected继承。

关键字`extends`表明正在构造的类派生于一个已存在的类，已存在的类称为**超类(superclass)**, **基类(base class)**或者**父类(parent class)**。新类称为 **子类(subclass)**、**派生类(derived class)** 或者**孩子类(child class)**。

“超”与“子”来自于数学中的超集与子集。

将通用的方法和域放在超类中，将特殊的方法和域放在子类中是常见处理方法，
然后有的情况是，超类中的方法对子类并不适用。
然而，虽然子类可以增加域和方法、以及覆盖超类的域和方法，但不能禁用继承来的超类的域和方法。

#### 覆盖超类中的方法
例如，假设Employee和Manager都需要一个`getSalary()`方法。
这时候就需要Manager新建`getSalary()`方法以覆盖超类中的方法。

然后这并不那么简单，如下为错误示例：
其中salary是超类私有的不能直接访问，我觉得可以用比如C++中的`protected`关键字解决，但是并不是一个好的设计模式，最好用超类定义的`getSalary()`作为接口。
也就是著名的**Program to an interface, not an implementation**

```java
class Manager extends Employee {
    private double bonus;
    public double getSalary() {
        return salary + bonus; //错误，salary是超类私有的不能直接访问
    }
}
```

既然用接口来解决这一个问题，但如下错误示例中并不能区分getSalary()来自超类还是子类，造成循环调用。

```java
class Manager extends Employee {
    private double bonus;
    public double getSalary() {
        return getSalary() + bonus; //错误，不能区分getSalary()来自超类还是子类
    }
}
```

正确的方法是用`super`关键字：

```java
class Manager extends Employee {
    private double bonus;
    public double getSalary() {
        return super.getSalary() + bonus; //super.getSalary()
        //注意此处的super不能替换为Manager，
        //这是因为如果用Manager.getSalary()，那getSalary必须是static方法
    }
}
```

`super`关键字可以用`this`关键字类比，用法类似，只不过`super`引用超类，`this`引用自身。又因为Java拒绝多重继承，所以超类是唯一的，没有问题。
当然super调用超类接口方法而不是域保证封装行，this则是调用域。


如果是C++，可以用`Employee::getSalary()`解决这一问题。

`super`也可以跟`this`一样，用在构造函数引用构造函数中，例如：

```java
public Manager(String name, double salary, int year, int month, int day) {
    super(name, salary, year, month, day);//引用超类的构造函数
    bonus = 0;
}
```

如上super调用超类构造器必须是放在第一条执行，如果没有super这一句，则系统默认调用超类的无参数的构造器，例如`Employee();`

如果是C++则通常用初始化列表解决，或者也采用默认无参数构造函数，例如：

```cpp
// C++
Manager::Manager(String name, double salary, int year, int month, int day) 
: Employee(name, salary, year, month, day) //用参数化列表调用
{
    bonus = 0;
}
```

#### 默认动态绑定
Java是默认的动态绑定(dynamic binding)来实现多态，相当于默认情况下，方法都是`virtual`的。如果不需要一个方法具有虚拟特性，则将它标记为`final`。

这方面C++则需要`virtual`关键字说明。

我猜测Java为了提高效率会检测运行期是否有多态来对进行优化。

多态的代码示例如下：

```java
for(Employee e : staff)
    System.out.println("name =" + e.getName() +
                        "salary" + e.getSalary());
```

其中e虽然是Employee的引用，但既可以引用Employee，也可以引用Manager，所以e.getSalary()即可以是Employee.getSalary(),也可以说Manager.getSalary();

### 继承层次
超类与子类是一对多的关系。

Java也有多重继承的实现方法。

### 多态
有一个用来判断时候应该设计为继承的简单规则，就是**"is-a"**规则。例如经理是一个雇员。

**"is-a"**规则的另一个表述是**置换法则**。
**置换法则：程序中出现超类对象的任何地方都可以用子类对象替换**

以下为超类、子类中方法调用和赋值的细节，
包括：
超类不能调用属于子类的方法
子类不能赋给超类的对象变量

```java
Manager boss = new Manager("zhangsan", 10000, 1990, 1, 1);
Employee[] staff = new Employee[3];
staff[0] = boss; //OK 超类引用子类
Manager m = staff[1] //ERROR, 子类不能引用超类
boss.setBonus() //OK setBonus()是子类的方法，没有问题
staff[0].setBonus() //ERROR, 编译器会提示找不到该方法，因为子类没有超类的方法
//这也是为什么要先用一个子类boss执行方法再赋值给超类变量的原因
```

#### 注意子类与超类对象变量的new对象
例如:

```java
DerivedClass[] dc= new DerivedClass[10]; //
BaseClass[] bc = dc; //OK 超类引用了子类，看上去像是要多态的样子
bc[0] = new BaseClass();
//虽然编译器不会提示错误，但是，本来应该是new DerivedClass的啊
//
dc[0].derivedClassMethod();//看上去没有错，子类对象变量引用子类方法
//但是，dc[0]明明是bc[0] new的超类对象啊！！因此，该方法将执行未知内存的指令
```

之所以出现上述错误是因为：
**通过超类new了超类对象，又通过子类执行子类的方法造成的错误**

正确的多态应该是子类new子类对象，通过超类执行动态绑定的多态方法。

这类错误将可能会出现在子类**数组中**，这是因为：

```java
DerivedClass dc1 = new DerivedClass; //ERROR, 子类对象声明一定要立刻new生成实例，
//即 应该执行 DerivedClass dc1 = new DerivedClass();

DerivedClass[] dc2 = new DerivedClass[10]; //OK
//数组可以生成子类的对象变量，但并不需要立即new生成实例对象
```

所以说用子类数组就要赶紧先new生成子类的对象实例，避免后面被new成了超类对象实例。

### 动态绑定
#### 编译器的对象方法执行过程
编译器对对象方法的调用过程如下：

1) 根据调用方法的方法名和类名查找方法，如果有重载，一一列出，范围为所有该类的方法，以及其超类中的public同名方法。
至此获得所有可能的候选方法。

2）查找参数类型是否匹配，匹配则勾选这个方法，这个过程称为**重载解析（overloading resolution）**。同时再考虑子类覆盖超类的情况，由于允许类型转换，这个过程较为复杂。
至此获得需调用的方法名和参数类型。

3）如果是private方法、static方法、final方法或者构造器，编译器能准确判断调用哪个方法。这种调用方式称为**静态绑定（static binding）**。
而如果调用方法依赖于隐式参数的实际类型（即this的实际类型）则在运行时实现动态绑定（dynamic binding）**。看是否是动态绑定就看有没private、static、final关键字。

4）当程序运行时，且采用动态绑定调用方法时，虚拟机一定调用所引用对象的**实际类型**最合适的那个类的方法。

每次调用方法都要进行搜索，时间开销极大。因此，虚拟机预先为每个类创建一个**放发表(Method Table)**,列出所有方法的前面和实际调用的方法。
这样，当真正调用方法时，虚拟机仅查看此表即可。


#### 子类覆盖超类的规则
注：方法的名字和参数列表称为方法的签名，注意**不包含返回类型**。
此外子类的可见性不能低于超类，例如超类声明为public，子类也必须是public

总结起来在如下情况下子类可以覆盖超类方法：

* 相同的方法名和参数表
* 不能有private、static、final修饰
* 返回类型可以不同
* 子类方法可见性不低于超类

### 阻止继承：final 类和方法
有时希望某个类没有子类，这用final类，final类中的方法都是final，但域不是
有时希望超类的方法不被子类覆盖，用final方法

只需将`final`关键字即可。

Java默认方法是可以动态绑定的，即默认是多态的，所有有些Java程序员将大部分方法声明为final

而如果一个方法没有被覆盖且很短，编译器可以优化处理，该过程称为**内联（inlining）**。 
而如果方法在另一个类中被覆盖，则编译器无法知道覆盖的代码将做什么，自然就不能内联处理。

比较幸运的是，虚拟机的JIT编译器比传统编译器处理能力强的多。JIT编译器可准确知道类之间的继承关系，并检测出类中是否真正存在覆盖方法，如方法短小、频繁调用且未真正覆盖，则编译器可以内联优化。
如果虚拟机又加载另一个类覆盖了内联方法，则编译器取消对覆盖方法的内联，此过程很慢，但极少发生。

### 强制类型转换
主要说对象变量的强制类型转换，有时候需要将某个类的对象引用转换成另外一个类的对象引用。例如：

```java
Manager boss = (Manager) staff[0]; 
//由于超类变量赋值给子类变量不合法，需要强制类型转换
```

**使用对象的强制类型转换的唯一原因是：在暂时忽略对象实际类型之后，使用对象全部功能**。

因为通常强制转换需要考虑类型匹配的问题，以免出现类似如上函数调用错误的现象，所以可以先类型检测，实例代码如下：

```java
if(staff[1] instanceof Manager) { // instanceof是一个运算符返回true/false
    Manager boss = (Manager) staff[1];
    // do sth.
}  
```

虽然Java对象的强制类型转换格式上看上去像C中的强制转换语法，
但其实**Java对象的强制类型转换本质类似于C++中的`dynamic_cast`**

上述代码如果用C++来写，则是：

```cpp
Manager* boss = dynamic_cast<Manager*>(staff[1]); //C++
if(boss != MULL)
    // do sth.
```

Java和C++的强制转换主要区别是，当类型转换失败时：

* C++返回NULL，所以先比较是否为NULL判断
* Java则抛出ClassCastException异常，所以为了避免抛出异常，先用`instanceof`检测类型。

**通常情况下，应该尽量少用类型转换和instanceof运算符，有时可能是超类设计不合理，可把需要的函数放在超类中就可避免类型转换。**

### 抽象类
抽象类的特点是：

1. **从编译器的角度，是禁止抽象类生成实例，C++没有这一关键字作限定。**
2. **抽象类其实可以含有具体的方法和域**
3. **含有抽象方法的类一定是抽象类，这是因为抽象方法一定要子类多态实现，如果具体实例对象含有抽象方法，执行就错误了。**
4. **反之，抽象类不一定有抽象方法，主要看自不需要禁止生成实例**
5. **从继承的角度，对4补充，只有子类完全不含抽象方法后，才可以不被abstract限定，否则包括其在内的所有父类都必须是抽象类**
4. **抽象类通常用于多态（废话）**

抽象方法类似于C++的纯虚函数。只要有一个纯虚函数，该类就是抽象类，但C++没有表示抽象类的关键字。
C++的虚函数就说明子类函数**可以覆盖**该类函数，纯虚函数则是子类函数**必须覆盖**该类函数。

### 受保护的访问
Java控制可见性的4种级别：

1. private          仅对本类可见
2. 默认，无修饰符   对本包可见
3. protected        对本包及所有子类可见
4. public           对所有类可见

通常将域标记为private，方法标记为public。
如需要允许子类方法访问超类的域，将域声明为protected
而实际过程中需要**谨慎使用protected**，这是因为如果子类有改变父类域的权限，那每次改变都得通知所有使用该类对象的程序员，**这破环了OOP提倡的数据封装原则**。

而**protected方法更有实际意义**，保证该方法仅能被子类使用而不被其他类使用，例如Object的clone方法。

#### 如C++比较
Java的protected概念比C++安全性差，这是因为Javaprotected还对所有包中的其他类可见。

## Object: 所有类的超类
注意是**所有类**的超类。
在Java中只有基本类型（数值、字符、布尔）不是对象，其他都是，包括数组。

本章节主要熟悉Object的常用方法，毕竟这是所有类都会继承的。

#### 与C++比较
C++中没有一个所有类的根类，但每个指针都可以转换成void*指针。

### equals方法
检测一个对象是否等于另一个对象。
**Object默认的equals是判断两个对象是否具有相同的引用。因此两个不同对象逻辑上相等也会判断为false，因此要想获得逻辑上的equal方法，必须自己扩展。**

如下是扩展equals方法的基本代码形式，其中name为String对象，String对象也对继承自Object的equal进行了逻辑上的修正，因此可以直接用。
因此，**在对类中每个对象域进行相对比较时，先查一下该类的API是否对equals进行了逻辑修正。**
注：数组类型的域可以用`Arrays.equals`方法作为检测条件。

```java
class Employee {
    public boolean equals(Object otherObject) {
        //先判断是否引用同一对象
        if(this == otherObject) return true;

        //如果otherObject是Null显然不相等
        if(otherObject == null) return false;

        //如果对象的类的类型都不一致，显然也不相等
        if(this.getClass() != otherObject.getClass())
            return false;

        //既然是同一类，那就比较各个域是否相对
        Employee other = (Employee) otherObject;

        return name.equals(other.name) 
                && salary == other.salary
                && hireDay.equals(other.hireDay);
    }
}
```

以下代码是来自IntelliJ IDEA的自动补全equals写法，更加规范：

```java
    public boolean equals(Object o) {
        if (this == o) return true;//先判断是否引用同一对象
        if (!(o instanceof Employee)) return false;
        //如果对象的类的类型都不一致，显然也不相等
        //

        Employee employee = (Employee) o;

        //double 为何不用==操作数啊
        if (Double.compare(employee.getSalary(), getSalary()) != 0) return false;
        if (getName() != null ? !getName().equals(employee.getName()) : employee.getName() != null) return false;
        return !(getHireDay() != null ? !getHireDay().equals(employee.getHireDay()) : employee.getHireDay() != null);
    }
```

上述中的：

```java
        if (getName() != null ? !getName().equals(employee.getName()) : employee.getName() != null) return false;
```

还有另一个简单的写法：

```java
        if Object.equals(getName(), employee.getName);
```

因为上述Object中的`static Boolean equals(Object a, Object b)`方法含有null检测：
如果a, b都为null返回true；如果其中一个为null返回false；否则返回a.equals(b)

#### 判断类型一致性时，用getClass还是instanceof

```
    if (!(o instanceof Employee)) return false;
//子类 instanceof 超类 == true
```

所以如果类型不完全一致也不要紧，也就是也可以用于 超类.equals(子类) 这样的相对性判断。

如果要严格控制类型，即子类与对应超类比较也不算，就用如下的方式：

```java
    if (o == null || getClass() != o.getClass()) return false;
```

相关也可参考： <http://stackoverflow.com/questions/596462/any-reason-to-prefer-getclass-over-instanceof-when-generating-equals>

getClass和instanceof关键的不同在于：

* getClass仅同一类时才为true
* instanceof不仅是用一类，还可以是超类、接口等

#### 相等测试与继承
如上所述，如果用instanceof来处理equals方法会带来非对称性，这个不是Java规范的要求（当然具体用什么看逻辑）

Java语言要求equals方法具有如下特性：

1. **自反性** ： `IF x != null THEN x.equals(x) = true`
2. **对称性**： `IF x.equals(y) = true THEN y.equals(x) = true`
3. **传递性**： `IF x.equals(y) = true && y.equals(z) = true THEN x.equals(z) = true`
4. **一致性**：如果x、y引用对象无变化，则x.equals(y)不会变
5. `IF x != null THEN x.equal(null) = false`

如果用instanceof，由于`subClass instanceof superClass`为真，而`superClass instanceof subClass` 为假，例如经理是一个雇员，但雇员不是一个经理，因此违反了Java提倡的**对称性规则**。

但有的情况需要用instanceof，例如：
当一个SuperCLass有两个子类，SubClass1和SubClass2， 而我们需要广义地比较SubClass1和SubClass2是否相对，所以注意这里相等的概念了，
要么用getClass显然不会相等。
这个时候，就在SuperClass中定义一个通用的equals方法(通常还需是final，防止子类覆盖误用，少数情况下为了子类扩展也不声明为final)，用`if other instanceof SuperClass`来判断，并在子类中用`if super.equals(other)`作为判断条件。
这个在IDE中，例如IntelliJ IDEA中有现成的模板，只需要<kbd>alt</kbd> + <kbd>insert</kbd> 即可自动生成，并选择是否用instanceof还是getClass，所以代码省略。

总的来说：

1. **严格意义的相等，用getClass**, 保证相等的对称性
2. **广义的相等，用instanceof superClass**，例如比较一个集合中的两个子集是否相等，

### hashCode方法
如果重新定义了equals方法，就**必须重新定义**hashCode方法。两者息息相关。
如果`x.equals(y) == true`, 那么必须保证`x.hashCode() == y.hashCode()`。
例如，如果Employee.equals方法比较雇员的ID，那么Employee.hashCode方法就需要通过ID生成散列表，而不是雇员姓名或其他。

关于hash的实现细节因为方式很多就不深究了，
想简单一点实现就用`public static int Object.hash(Object... values)`这个方法。

### toString方法
随处可见的`toString`方法是为了：**保证一个对象与一个字符串用操作符“ + ”时，Java编译能够自动调用`toString`方法合成字符串。**

同样的，如果x是任意对象，当调用`System.println(x)`时，`println`方法直接调用`x.toString()`

Object类则定义了一个默认的toString方法，打印输出**对象所属类名**和**散列码**。

#### 数组的toString方法的调用用Arrays.toString/Arrays.deepToString
一个数组对象的toString方法的输出不是为了供普通程序员使用的，应该改用Arrays的静态toString方法，例如：

```java
int[] primeNum = {2, 3, 5, 7, 11, 13};
System.out.println(primeNum); //输出： I@4554617c
System.out.println(Arrays.toString(primeNum));// 输出： [2, 3, 5, 7, 11, 13]
```

多维数组则用Arrays.deepToString方法

#### 强烈建议每个自定义类增加toString方法
这样可以用` + `和`println`实现输出字符串，便于调试，也是一种程序员合作的约定俗成。

## 泛型ArrayList
#### 使用ArrayList作为可动态扩展的数组
编译期确定数组大小令人讨厌，许多程序设计语言中，**特别是C++中**，必须在编译时就确定整个数组大小，程序员对此十分反感。
其实，C++也提供可动态变更大小的STL的vector模板。

Java有纯粹的语法支持可在运行时再确定数组大小，这一点C++没有，毕竟Java是解释性动态语言，其语法为：

```
int actualSize = ...; //运行期确定
Employee[] staff = new Employee[actualSize];
```

以上语法有一个限制是**一旦...在运行期确定后，很难改变。
而要想使用**可动态扩展的数组**，Java中用`ArrayList<typename>`，类似于C++中的vector模板。

语法类似于：

```java
ArrayList<Employee> staff = new ArrayList<Employee>();
//可简写为：
ArrayList<Employee> staff = new ArrayList<>();//编译器自动类型推导，更符合DIY原则
```

使用`add`方法将元素添加至数组列表，例如：

```
staff.add(new Employee("Tom"));
```

如果调用add时数组已满，则自动创建一个更大的数组，并将所有对象从较小的数组中拷贝到较大的数组。

size()方法返回ArrayList大小
如果基本确定该数组列表大小不再改变，可以用**trimToSize()方法**释放掉多余的空间。

#### ArrayList与C++中的vector比较
C++中vector为了便于访问元素，重载了`[]`操作符，Java没有重载操作符的能力；
C++中vector拷贝是值拷贝，`a = b`将会构造一个和b一样的新向量a，而Java则是让a和b引用同一个数组列表。

### 访问数组列表元素
如上所述，Java不能像C++那样重载`[]`访问数组列表，而是：

```java
staff.set(i, harry); //功能等同于数组中  staff[i] = harry
staff.get(i);  //功能等同于数组中的 staff[i]
```

如果为了方便使用数组，可以将数组列表拷贝到数组中去：

```
int[] array_a = new int[list_b.size()]; //list_b is a arraylist
list_b.toArray(array_a);  //copy to array
```

### 可以使方法的参数和返回类型更加泛型
如果ArrayList作为参数类型或者返回类型会怎样呢？
例如：

```java
public class EmployeeDB {
    public void update(ArrayList list) { /*do sth*/ };
}
```

上述代码中 `public void update(ArrayList list)`的参数类型无需指明是`ArrayList<int>`还是`ArrayList<Employee>`等类型，这样更加泛型，使用也很方便：

```java
ArrayList<Employee> staff = ...;
employeeDB.update(staff); //用于ArrayList<Employee>没有问题，会
```

警告：上述用法特点是使得方法**更加泛型**，缺点是很有可能带来安全性问题，编译器
有可能察觉不出，需要人工仔细确认是否安全，是否可用于多种类型而无错误。

#### 编译器处理泛型参数时进行泛型检查
例如，实参为`ArrayList<Employee>`,对应形参为`ArrayList`时，编译器针对该方法，将所有类型化的数组列表（如上的实参）**转换成**原始ArrayList对象，运行时，所有类型都一样看待，即没有虚拟机中的类型参数。
因此，类型转换后的ArrayList和实参`ArrayList<Employee>`执行**相同的运行时检查**。

#### 返回类型为泛型和参数泛型一样，只是会给出警告
例如：

```java
public class EmployeeDB {
    public ArrayList find(String query) { /*do sth*/ };
}

ArrayLis<Employee> result = employeeDB.find("query");
//与泛型参数一样，只是会给出警告
```

如果在确定代码没有产生严重后果的情况下，取消警告的方法如下：

```java
@SuppressWarnings("unchecked") //标签 取消警告
ArrayList<Employee> result = (ArrayList<Employee>) employeeDB.find("query");
```

## 对象包装器与自动装箱
有时需要将int等基本类型转换成对象，所有基本类型都有对应的类，称为**包装器（wrapper）**。

各wrapper如下：

1. Integer 对应 int
2. Long 对应 long
3. Float 对应 float
4. Double 对应 double
5. Short 对应 short
6. Byte 对应 byte
7. Character 对应 char
8. Void 对应 void
9. Boolean 对应 boolean

其中1到6派生于公共超类Number

#### 对象包装器不可变且不能被继承
一旦构造了包装器，就不允许更改包装器内的值。同时包装器是final类，不能定义其子类

#### 包装器用途之一是泛型的类型化
例如不允许写`ArrayList<int>`,取而代之用`ArrayList<Integer> list = new ArrayList<>()`。

#### 自动装箱与自动拆卸
即将函参的隐式类型转化，将基本类型转换为对应wrapper（自动装箱）
又或者返回类型隐式转化，将wrapper转化为对应基本类型（自动拆箱）

例如对于如上list如果要增加一个整数3，用`list.add(3)`，
其实被编译器自动转换成了`list.add( Integer.valueOf(3) )`，所谓自动装箱（autoboxing）,装箱一词源自C#

比如`int n = list.get(i)`就是将返回类型wrapper转化为对应基本类型（自动拆箱）
编译器翻译成`int n = list.get(i).intValue()`

#### 注意包装器的相等比较用equals方法
包装器通常保证与基本类型操作习惯一致，都进行中自动装箱、拆箱这样的隐式转化，
例如`Integer n; n++`的自增运算就很自然地使用。

但是需要注意相等比较，例如：

```java
Integer a = 1000;
Integer b = 1000;
if (a == b) //比较对象地址，可能返回false，
//除非a、b介于-128~127之间才一定为true，
//-128~127是boolean,byte,char的范围，
//-128~127中的short、Integer包装到固定对象
```

取而代之应该用equals方法。

#### 包装器的另一好处是扩充了基本类型的能力
例如`java.lang.Integer`API中的
`static int parseInt(String)`将字符串转换成数值
`static Integer valueOf(String)`将字符串转换成Integer
`static String toString(int i)`将数值转换成字符串

**一定注意包装类是不可变的**，所有有些改变值的方法需要用可变的类，
例如int对应IntHolder类。

可以看出Java为了扩展基本类型的应用使用包装类，而有些语言干脆一切皆对象，例如python。

## 参数数量可变的方法
例如printf语句就是典型的可变参数方法：

```java
System.out.printf("%d %s", 10, "abc");
```

实际是printf的定义为：

```java
public PrintStream printf(String fmt, Object... args) {
    return format(fmt, args);
}
```

#### 可变参数可理解为一个数组

例如：

```java
public static double max(double... values) {
    double result = Double.MIN_VALUE;
    for(double v : values)
        if (v > result) result = v;
    return result;
}
```

可见其可理解为数组，只是可以在参数列表中将每一项元素展开。

#### 可变参数必须是最后一个参数
显然，只有这样编译器才能区分哪些参数划分到可变参数。

## 枚举类
参见： [Java学习 第5章 枚举类]({% post_url 2015-08-31-learning-java-chapter-5-enum %})

## 反射
由于算Java中比较有特点的地方，所以单独开一篇，见下一篇：[Java学习 第5章 继承]({% post_url 2015-08-31-learning-java-chapter-5-reflection %})

## 继承设计的技巧
#### 1. 将公共域和方法抽象出来放到超类中
#### 2. 不要使用protected域
Java的protected机制并不能带来好的保护，原因是：

1. 只要将一个类派生出一个子类就能访问protected域了，破坏了封装性
2. Java中同一个package的任意类都可以访问protected域

#### 3. 使用继承实现is-a关系
重点考虑是不是is-a的关系，是才能用继承，否则只会带来麻烦。

#### 4. 除非继承的方法都对子类有意义，否则不使用继承
例如有些方法会给子类带来意料之外的更改之类。

#### 5. 在覆盖方法时，不要改变预期的行为

#### 6. 使用多态，而非具体类型
例如遇到下面的情况，很有可能可以抽象成一个超类。

```
if x is type1
    action1(x)
else if x is type2
    action2(x)
```

使用多态方法或者接口编写的代码比使用对多种类型进行检测的方法更易于维护和扩展。

#### 7. 不要过多使用反射
反射很脆弱，编译器难以发现错误，常常会导致异常，所以编写应用程序时尽量不要用反射。

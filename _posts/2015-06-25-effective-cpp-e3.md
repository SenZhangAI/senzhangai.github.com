---
layout: post
title: "C++填坑系列：tip3 尽可能用const"
description: "C++填坑，主要来自effective c++"
keywords: c++, cpp, effective
category: c++
tags: [c++]
---

### 20160104 补充
再读《inside C++ object model》之后(第四章 Function语意学开始的例子)，
对于const又有了新的认识，所以这里补充相关内容，
这部分内容是我自己的理解，所以不一定正确，但我有把握应该是对的。

#### 先修知识-关于const、static、virtual
首先，对于类的成员函数而言有三种修饰符，
分别是`static`，`const`和`virtual`。
且这三种修饰符是**互不兼容的**。
即static函数一定不是const。
原因是它们代表了互不兼容的不同的语意。
static说明没有this指针
const/non-const说明this指针是const/non-const形式的，
virtual说明函数由vtbl间接调用实现运行期多态。

#### const成员函数的深入理解及例子

```cpp
//1. static成员函数相当于普通的非成员函数，没有this指针
    static void MyClass::function_static();
    //等价于
    function_static();

    //因为没有this指针，所以也不能操作实例中的成员数据或者虚表(自然也就不可能为virtual了)
    //但通常操作static数据成员没有问题，毕竟static数据成员不含this指针。

//2. nonstatic 普通成员函数
    //其实编译器将其转换成了非成员函数，只不过加上this指针，以针对不同类实例
    void MyClass::function_normal();
    //等价于
    void function_normal(MyClass *const this);

    //2-2. nonstatic const 成员函数，都是const是修饰this指针的，现在才终于理解了，如下：
        void MyClass::function_const() const;
        //等价于
        void function_const(const MyClass *const this);
        //**注意** 这里this指针不仅要求指针不能变，指针所指对象也不能变
        //所以包括旧文中总结的const、non-const规则实际上就是const和non-const指针的重载


    //对于const实例，其指针必定是const的，所以决议时只能是const成员函数

    //对于non-const实例，其指针是non-const，决议时优先non-const成员函数，
    //如果没有则决议const成员函数

//3. virtual成员函数
    //virtual成员函数通常通过虚表指针、虚表实现运行期动态

    virtual void MyClass::function_virtual();
    //相当于在运行期执行
    ( * obj.vptr[2] ) ( &obj );

    //其中obj.vptr[2]是函数指针，上述格式为调用该函数，数字2是我假设给的，实际根据虚函数多少
    //&obj为this指针
    //virtual函数更多内容以后再开篇博客综述
```

如上从C++语意学层面理解const、non-const、static成员函数

关于重载则是name mangling机制，就是根据不同类名、不同函参设计一个唯一的命名，
然后再决议觉得调用哪个函数，内容就不扩展了。

====
以下为旧文

### const与指针

```cpp
char sample[]="Hello world";
const char *p = sample; //non-const pointer, const data,
// so const char *p == const (char *p)
char * const p = sample; //const pointer, non-const data
```

### 迭代器的const写法

```cpp
//一些const易出错的场合

std::vector<int> vec;

// iter 不能移动，但所指向的数，即*iter可变
const std::vector<int>::iterator iter = vec.begin();

//const_iter 可以移动，但所指的数不能变
std::vector<int>::const_iterator const_iter = vec.begin();
```

之所以尽可能用const基于以下优点：

1. 函数易于理解
2. 更健壮，便于查错

### const成员函数
成员函数声明为const是为了确认该函数可以作用于const对象上，没有声明为const则不行，以下是例子：

注： 对于成员函数的const关键字，其本质是作用于this, 即this类型为`const MyClass* this`, this指向的数据不可改变，所以this可以作用于const实例，所以该函数可用于const实例。

``` cpp
class MyClass {
public:
    void getMember(void) const; //因为是get，不会改变成员，故声明为const
    void getMember(void);
    //虽然也可以被声明为non-const（此处重载），但意义不一样，对于const实例不能用。
private:
    int member = 10;
};

//如果实例为const对象，即 const MyClass constClass
//则constClass.getMember();调用的是void getMember(void) const;
const MyClass constClass;
constClass.getMember();

MyClass non_constClass;
non_constClass.getMember();
/* 结论:
 * 如果有重载：
 *     non const实例(Text t;)调用non const成员函数
 *   const实例(const Text t;)调用    const成员函数
 *
 * 如果没有重载
 * non const实例在没有non const成员函数情况下，可以调用const函数
 * 而  const实例只能调用const函数，不能调用非const函数
 * 因此，要想有const实例，一定要对可声明const的函数声明为const
 */
```

#### bitwise constness和logical constness
对于const成员函数的判断标准有两种观点，bitwise constness和logical constness

bitwise constness 顾名思义指的类中每一个bit都不会因成员函数而改变才算作const 成员函数。这个特点对编译器而言实现很容易做到，只需要检测成员函数的赋值动作即可。

然而在某些情况下意义上应该是const却满足不了bitwise constness;或者意义上不是const成员函数却符合bitwise constness，所以从应用层面说，应该考虑的是logical constness

logical constness 主张一个const成员函数可以修改对象内某些bits

##### 错误示例1-符合const bitwise的成员函数，但不应作为const处理
以下典型例子只对指针取值，因指针并未改变，所以从bitwise角度应该是const，
然而指针所指之物却是可变，这与该函数本身的目标不一致。

```cpp
/*例子来自Effective C++ P21 首先是错误例子，然后是修正例子*/
class CTextBlock_Err {
public:
    CTextBlock_Err(char* cstr) {
        pText = cstr;
    };
    inline char& operator[] (std::size_t position) const { //不应该声明为const
        return pText[position];
    };
private:
    char* pText;
};
```

##### 正确解法
应该把const 实例和non-const实例分开处理，一个返回const保证不被修改；一个返回non-const，保证可被修改。

``` cpp
class CTextBlock_Right {
public:
    CTextBlock_Right(char* cstr) {
        pText = cstr;
    };

    //正解是将const 和non-const分开，重载
    inline const char& operator[] (std::size_t position) const {
        return pText[position];
    };

    inline char& operator[] (std::size_t position) {
        return pText[position];
    };

private:
    char* pText;
};
```

##### 错误示例2-不符合const bitwise的成员函数，但应作为const处理
试举一例(有些牵强)，代码略
假设上例中有一个`reverse`函数将`pText`指针指向末尾后反输出
那么pText改变了，不满足bitwise。
然而这并不影响到所指向的chars的内容，故实际上对于const实例也是成立的。
故应该也适用于const成员函数

#### mutable确保const函数内部分成员可以被赋值
要想满足上述需求，即在const成员函数中修改部分成员的内容，则**需要告诉编译器有些变量即便对于const实例也需要被修改**，即将这些变量用mutable修饰。
例如对于上述例子，应将`char *pText` 改为 `mutable char* pText`，即：

```cpp
class CTextBlock_Mutable {
public:
    // ....
private:
    mutable char* pText;
};
```

#### non-const与const的重复问题
不难看出，很多情况下const是为了确保const成员函数与其对应的non-const成员函数绝大多数都是相同的，仅仅只是返回结果一个是const，一个是非const，**出现重复了**。

解决办法，第一个想到的是把重复的部分剥离出来单独做出一个private函数，然后const和non-const版本都调用该函数，但这个单独分出的函数意义不明确了，而且调用函数也是重复了代码。

更好的办法是用non-const调用const版本，并利用cast改变const属性，代码见下：

``` cpp
class CTextBlock_Cast {
public:
    CTextBlock_Cast(char* cstr) {
        pText = cstr;
    };

    inline const char& operator[] (std::size_t position) const {
        // ... 此处省略很多代码，故不能容忍重复代码
        return pText[position];
    };

    inline char& operator[] (std::size_t position) {
        return
            const_cast< char& > ( // 再将const函数的返回的const属性去掉
              static_cast< const CTextBlock_Cast& > (*this)
              // 先将non-const的class转换为const便于调用其函数
                [position] // 调用函数，即(*this)[position]
            );
    };

private:
    char* pText;
};
```

### 要点小结

1. const可提高可读性和便于编译器纠错
2. 注意指针以及迭代器的const写法
3. 成员函数是否为const根据自己逻辑判断决定，对象主要成员不变设为const，
例如`size()`函数返回大小，并不改变内容，则设置为const。
4. 其实本质而言`non const成员函数`是屏蔽了`const实例`的调用，规则见下，因此主要思考`const Class a`实例能够调用该`成员函数`呢？
（例如赋值就不行，取大小就可以）如果能就加上const
5. 有些const函数却部分改变了class的值，根据bitwise constness的看法，编译器报错，
因此， 需要将`const成员函数`中可变的变量用`mutable`修饰。
6. 对于有些const实例返回值为const，而non-const实例返回值为non-const等存在差异的情况，则需要用重载。
7. 6中所述情况non-const和const重载函数很多重复代码，
优化方法是用non-const调用const版本，并利用cast改变const属性。
8. const关键字是修饰隐式参数this的，也就是`const MyClass* this`,
这样，this就能用于const实例，也就可以在const实例中执行const成员函数了。

``` cpp
// ******* const成员函数 与 non-const成员函数 的调用规则: *******
// 如果有重载(即同时有const版和非const版成员函数)：
//     non const实例(Text t;)调用non const成员函数
//         const实例(const Text t;)调用    const成员函数

// 如果没有重载
// non const实例在没有non const成员函数情况下，可以调用const函数
// 而  const实例只能调用const函数，不能调用非const函数
// 因此，要想有const实例，一定要对可声明const的函数声明为const
```

---
layout: post
title: "C++填坑系列：tip3 尽可能用const"
description: "C++填坑，主要来自effective c++"
keywords: c++, cpp, effective
category: c++
tags: [c++]
---

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
// 如果有重载：
//     non const实例(Text t;)调用non const成员函数
//         const实例(const Text t;)调用    const成员函数

// 如果没有重载
// non const实例在没有non const成员函数情况下，可以调用const函数
// 而  const实例只能调用const函数，不能调用非const函数
// 因此，要想有const实例，一定要对可声明const的函数声明为const
```

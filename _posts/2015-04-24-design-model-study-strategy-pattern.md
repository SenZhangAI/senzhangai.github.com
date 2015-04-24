---
layout: post
title: "Design Model Study Strategy Pattern"
description: "关于设计模式的学习实践：1. 策略模式"
keywords: "Design Pattern, Strategy Pattern"
category: "programming"
tags: [Design Pattern, Strategy Pattern]
---

## 1. 策略模式简述

根据我个人的理解。策略模式应用的场景应该像选电脑那样，选择不同的配置。

**技巧是在类中包含某c++的基类指针 或者 java中的interface（可能有误）**

我觉得需要注意以下几个方面：

1. 应用于**多选多**。

    如果是1选多，例如只有一个人选电脑配置，把一大串的清单列出来。列完了就

2. 有可复用的地方。

    如果是1选多，例如只有一个人选电脑配置，把一大串的清单列出来。列完了就

#### 其他代替方式比较

当然可以有其他替代方式，有以下我觉得可以的方法来做策略选择，然而肯定有缺陷咯。其他更好的方法没想到啊。

1. 在namespace中建立不同策略，例如attack::sword()。然后放到一个.h文件里面。也可以达到复用的目的。需要的时候直接include调用啊。**缺点是如果这样的策略选择很多的话例如attack，move，speak，defence等，需要include很多头文件咯，显得很凌乱。**

2. 当然也可以用继承实现。（个人认为其实包含某类实例或者某类指针本质上就相当于继承了）**缺点是过多依赖于继承会是的类变得无比复杂，例如多重继承**

## 2. 试例总体思路

假设场景为游戏中的一个角色`elf`可以拿`sword`,`gun`,`arc`或者赤手空拳`noweapon`。

那么选择`sword`,`gun`,`arc`,`noweapon`即为不同的**策略**。

因为c++没有java那样的interface,所以我猜测应该是：

1. 创建一个`weapon`的基类
2. `elf`类中加入`weapon`基类指针

    ``` cpp
    //类指针就相当于继承了啊，真爽
    weapon* pweapon;// = null_ptr;
    ```

3. 拿武器，例如`sword`就是：

    ```cpp
    pweapon= new sword(); //sword继承自weapon
    ```

    > delete怎么办？会把`pweapon`删除掉吧？
4. 攻击就是：

    ```cpp
    elf.attack();
    // 以下是elf::attack的定义
    float elf::attack(){
        return pweapon -> attack();
    }
    ```


5. 换武器就是：

    ```cpp
     /* 之所以用动态new的方式而不是直接用一个实例gun
      * 是因为我觉得一个实例如果多人同时使用会造成性能瓶颈吧（猜测）
      */
    elf.changeweapon(new gun()) //这里new gun()显得好生硬。貌似实际真是这么干的
    // 以下是changeweapon的定义。
    bool elf::changeweapon(weapon* selectedweapon){
        delete pweapon;
        pweapon = selectedweapon
        return 1;
    }
    ```

## 3. 感言

用基类里的vtable来模仿java里的interface总感觉有些麻烦啊。很生硬。
而且这样的关系不明显。没有明确的weapon子类可以查看啊
总之，感觉设计模式还没学好。

---
layout: post
title: "haskell intro"
description: ""
keywords: haskell
category: Programming
tags: [haskell]
---

## Values, Types

haskell是纯functional language

每个`value`都有对应的`type`，`type`可看做value的集合。

`values`是一等公民，可作为参数，返回值或数据成员。`types`则不是，`types`可理解为`values`的描述。

`expressions`就是通常意义上的表达式

`type expressions`则是`type`的值，例如`Integer`，`Char`,`Integer -> Integer`, `[Integer]`, `(Integer, Char)`

`Functions`由一系列的`equations`定义，例如：

```haskell
inc n = n + 1
```

`equations`是一种`declaration`，还有一种`declaration`是`type signature declaration`，如：

```haskell
inc :: Integer -> Integer
```

Haskell的`static type system`定义了types和values之间的关系，static type system 确保了Haskell类型安全，静态类型帮助程序员`reasoning about programs`
也帮助编译器生产更高效的代码(例如不需要运行期的type tags)

### Polymorphic Types

例如：

```haskell
length :: [a] -> Integer
length [] = 0
length (x:xs) = 1 + length xs

head :: [a] -> a
head(x:xs) = x

tail :: [a] -> [a]
tail(x:xs) = xs
```

其中`a`就用于多态类型，称为`type variables`，在Haskell中为了区别，`specific types`用首字母大写，例如`Int`
而`universal types`用首字母小写

可以发现`[a]`比`[Char]`更通用，后者应该可由前者推导得到(derived)，Haskell的类型系统有两个重要性质来保证这种`generalization ordering`

1. *every well-typed expression is guaranteed to have a unique principal type*
2. *the principal type can be inferred automatically*

我当前的理解就是每个表达式都有个具体的类型(*the least general type*)，可由抽象的类型推导得到。

```
The existence of *unique principal types* is the hallmark feature of the *Hindley-Milner* type system, which forms the basis of the type systems of Haskell
```

### User-Defined Types

例如：

```haskell
data Bool    = False | True
data Color   = Red | Green | Blue
data Point a = Pt a a
```

其中`Color`称为`type constructors`，`Red`之类称为`data constructors`

`Bool`和`Color`都是枚举类型（*enumerated types*）其包含有限数量的data constructors

而`Point`只含有一个data constructors，称为`tuple type`，它本质上是其他类型的笛卡尔积`cartesian product`，
multi constructors types，例如`Bool`和`Color`则称为`(disjoint) union 或 sum types`


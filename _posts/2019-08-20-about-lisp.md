---
layout: post
title: "lisp相关知识"
description: "about lisp"
keywords: lisp, macro
category: Programming
tags: [lisp, macro]
---

## 前言

一直断断续续得了解了一些lisp引入的概念，但是依然遗留一下问题未理清，好比拼图缺少了几块不完整，
这里整理一些知识点。

## macro

### 卫生宏与普通宏

卫生宏(Hygiene macro)相对普通宏而言，有种说法是类似于词法作用域函数和动态作用域函数，

微生活在展开时，会将内部的name改名，而普通宏不改名，捕捉外部的名字。

### 宏与函数的区别

<https://stackoverflow.com/questions/4980520/what-can-you-do-with-lisp-macros-that-you-cant-do-with-first-class-functions>

<https://stackoverflow.com/questions/11595523/lisp-macros-vs-functions>

大致意思是比函数强大，能实现普通函数实现不了的，例如`break`,`continue`，`loop`之类，编译期之前处理。通常为了实现DSL之类。但是也会引入一些复杂性。

## Symbol

因为看红红的用Julia实现Parserc引入Symbol的概念，说lisp里有，便查了一下。

lisp的Symbol有点像`key-value`字典，而ruby的Symbol感觉像一个字符串常量，但是实际上不是字符串常量。

Symbol像一个Singleton的常量对象，当作相等比较时，不需要像字符串那样逐字符比较，而是只需要比较对象地址是否相等, 常用于作为字典的key之类。

参见 <https://www.local-guru.net/blog/2009/2/10/ruby-symbols-vs-string-vs-constant>


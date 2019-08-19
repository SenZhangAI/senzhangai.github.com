---
layout: post
title: "[转]Chomsky四型文法"
description: "Chomsky grammars"
keywords: chomsky, grammars
category: Programming
tags: [grammars, parser]
---

## 前言

[什么是0型文法，1型文法，2型文法，3型文法？](http://www.iteye.com/topic/593981)

这篇文章对Chomsky的0型文法、1型文法、2型文法、3型文法总结的不错，故转载此贴并补充部分内容。

## 正文

乔姆斯基把方法分成四种类型，即0型、1型、2型和3型。

参见: <https://en.wikipedia.org/wiki/Chomsky_hierarchy#Type-0_grammars>

### 0型文法

参见: <https://en.wikipedia.org/wiki/Unrestricted_grammar>

```
设G=（VN，VT，P，S），如果它的每个产生式α→β是这样一种结构：

α∈(VN∪VT)*且至少含有一个非终结符，而β∈(VN∪VT)*，则G是一个0型文法。
```

0型文法也称短语文法，是递归枚举的，任何0型文法都可以用图灵机(Turing)实现。

0型文法是这几类文法中，限制最少的一个。

其定义是：

> The characteristic property of a Type 0 grammar is that it may contain rules that transform
> an arbitrary (non-zero) number of symbols inio an arbitrary (possible zero number of symbols)

### 1型文法

1型文法也叫上下文有关文法，此文法对应于线性有界自动机。

```
它是在0型文法的基础上每一个α→β,都有|β|>=|α|。这里的|β|表示的是β的长度。
```

注意：虽然要求|β|>=|α|，但有一特例：α→ε也满足1型文法。

例如 `, N E -> and N` 就不是1型文法，因为左边三个symbol，右边两个。

如有A->Ba 则|β|=2,|α|=1，符合1型文法要求。

反之, 如aA->a，则不符合1型文法。

### 2型文法

2型文法也叫上下文无关文法，它对应于下推自动机。

2型文法是在1型文法的基础上, 再满足：

```
每一个α→β都有α是一个非终结符(non-terminal symbol)。
```

如A->Ba,符合2型文法要求。

如Ab->Bab 虽然符合1型文法要求,
但不符合2型文法要求，因为其α=Ab，而Ab不是一个非终结符。

#### 补充：上下文相关与上下文无关

关于上下文相关(Context-Sensitive Grammar CSG)，参见[Wiki](https://en.wikipedia.org/wiki/Context-sensitive_grammar)

> A **context-sensitive grammar (CSG)** is a formal grammar in which the left-hand sides and right-hand sides of any production rules may be
> surrounded by a context of terminal and nonterminal symbols. Context-sensitive grammars are more general than context-free grammars, in the
> sense that there are languages that can be described by CSG but not by context-free grammars

上下文无关(Context-Free Grammar, CFG)，参见[徐辰的知乎问答](https://www.zhihu.com/question/21833944))，或者Wiki

> In context-free grammars, all rules are **one-to-one, one-to-many, or one-to-none**. These rules can be applied regardless of context.
> **The left-hand side of the production rule is always a nonterminal symbol**. This means that the symbol does not appear in the resulting formal language.
> Production rules are simple replacements.


比如

```
aSb -> aaSbb
S -> ab
```

因为它的第一个产生式左边(`aSb`)有不止一个符号，所以是上下文相关文法

此外，**上下文相关文法，上下文无关语法和语言的歧义性质无关**

**上下文无关文法也可能有歧义**，例如，对于如下规则：

```
S -> S1 | S2

S1 -> ab

S2 -> AB

A -> a

B -> b
```

此规则是CFG，但是对于字符串`"ab"`，一种推导方式是`S -> S1 -> ab`，另一种是`S -> S2 -> AB -> aB -> ab`，所以是有歧义的。

2 型文法 （上下文无关文法）的上下文无关是指每个产生式规则的展开都和上下文无关，就可以无需检查上下文(Production rules are simple replacements)。

### 3型文法

3型文法也叫正则文法，它对应于有限状态自动机。参见 <https://en.wikipedia.org/wiki/Regular_grammar>

```
它是在2型文法的基础上满足:A→α|αB（右线性）或A→α|Bα（左线性）。
注意一个语法中，只能选择右线性或左线性的一个规则（A regular grammar is a left or right regular grammar）。
```

#### 补充

正则表达式用一行非递归规则就能写出来，不能递归引用自己，不能表达嵌套结构。
（不过 PERL 的正则表达式比较强大，是接近 2 型文法的存在）

其他库比如 lex 或者 StringTokenizer 都可以对应到一个 3 型文法的产生规则。

#### 再补充

所谓的3型,即Finite Automata,不需要记忆任何状态，只根据输入进行状态转移。

2型则对应Pushdown Automata，唯一依靠的记忆是栈，例如html的产生式`ELEM -> <TAG>CONTENT</TAG>`，就需要栈。

1型的数学意义多于实际意义。

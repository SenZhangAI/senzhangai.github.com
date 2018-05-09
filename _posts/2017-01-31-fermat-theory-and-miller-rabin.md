---
layout: post
title: "费马小定理与Miller Rabin算法求质数"
description: "Fermat Theory and Miller-Rabin primality test"
keywords: Fermat Theory, Miller Rabin, prime number
category: Math
tags: [math, prime]
---

## 前言

费马小定理可被利用快速求质数，在大概率上成立，是概率算法，Miller Rabin算法则是其改进，进一步保证质数准确率。

## Fermat Theory

若`p`是质数，且`gcd(a,p)=1`(`a`与`p`的最大公约数为1，亦即`a`不是`p`的倍数)，
则:

\[
    a__{(p-1)} \equiv 1 \pmod{p}
\]

假如`a`是整数，`p`是质数，且`a`,`p`互质，则a的(p-1)次方除以p的余数恒等于1。

也有另一种变型，若a小于质数p（必然互质），则：

\[
    a__{(p)} \equiv a \pmod{p}
\]

即a的p次方除以p的余数恒定与a。

若不满足以上条件并为和数，若满足上述条件，则**极大概率**为质数，可多选取几个a测试，增大几率。

当然，也存在那种满足上述等式却不是质数的情况，称为**Carmichael数**，
毕竟费马小定理是质数成立则等式必然成立，反之不一定成立。

## Miller-Rabin primality test

参见：<https://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test>

以及：<https://zhuanlan.zhihu.com/p/24888769>


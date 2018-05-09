---
layout: post
title: "Concurrency and Message Passing in Newsqueak"
description: "并发编程与消息传递讲座整理"
keywords: Concurrency, Actor
category: Programming
tags: [Actor, Concurrency]
---

## 前言

参见链接： [Concurrency and Message Passing in Newsqueak](http://www.tudou.com/programs/view/vZEZKwCcuos/)

Rob Pike 介绍自己在 Plan 9 发明的 Newsqueak 语言的演讲。对于Golang 或者Erlang的理解很有帮助。

## 讲义

### Concurrency

The world is concurrent, but computers are not.

This is a profound(深远的) mismatch

Two approches:

    1. Make(interface to) the world sequential
    2. Make software concurrent

This talk is about approach 2.

### NOT

A common approach to concurrency is to consider it a necessary evil.

    The world is concurrent, computers are becoming multicore,
    it's all ugly, yuck, better learn it.

NO!

This talk is about how concurrent software makes **programmers** - not machines - more effcient.

Concurrency Programming provides a model for interface and design the simplifies even nonparallel software.

But still, when it's parallel you want...

### State
an executing process has:

    * program counter(PC)
    * stack

Write this as (PC, stack). This is a powerful way to represent state.
(Consider stack traceback.)

A concurrent program is (PC, stack)+.

Exponentially more powerful, yet conceptually just as easy to understand -
and only linearly harder to debug.

**What do we need to do?**

### The Model
Think about software as

    * interacting
    * independently executing
    * processes

In Hoare's original formulation,

    Communicating Sequential Processes

Deliberately(故意的) missing terminology(专有名词):

    * threads, share memory, locks, semaphores, ...

This is not Andrew Birrell's model, which is too low-level for easy programming.

### Selected History

Hoare, 1978:CSP

    - processes and communication

Cardelli & Pike, 1985:Squeak

    - CSP with channels(but really just a toy)
    - for interactive programs

Pike, 1988:**Newsqueak**

    - a full, interpreted, experimental language

Winterbottom, 1995:Alef

    - a full, compiled, systems language

Dorward, Pike, Winterbottom, 1996:Limbo

    - approximately JIT-ed Newsqueak

### Overview

    * An overview of Newsqueak
    * Processes
    * Channels
    * Programs
    * Interface design
    * System software

### An overview of Newsqueak

    * Looks a lot like Sawzall(but recher)
    * Pascal-like type declarations
    * C-like control structures
    * Functions are just variables(lambdas, prog)
    * Process controls(begin, become)
    * Channels are first-class citizens
    * Memory: value semantics(警戒)(no sharing)



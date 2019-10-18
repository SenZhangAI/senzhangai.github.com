---
layout: post
title: "think of cpp parser"
description: ""
keywords:
category:
tags: []
---

## 目标

```

stringstream ss = "GET uri HTTP/1.1\n...";

ss >> FindToken("GET") >> MustConsume(" ") >> UriToken() >> MustConsume(" ") >> MustConsume("HTTP/1.1") >> MustConsume("\r\n");

```

这样写起来一定非常爽, 但是问题， stringstream的state 只有 good eof fail bad 这么几种，不能表示parsing state

这样，只能扩展stringstream才行

一种扩展是继承，但是重载 `>>` 这个复杂度有点大，不想这样做

另一种不用扩展，作为成员变量，但是要写几个函数，还是算了，

stringstream 的问题是不好回溯，所以还是用string比较好，也能写得比较漂亮。


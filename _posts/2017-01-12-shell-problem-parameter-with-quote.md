---
layout: post
title: "Shell脚本编写中参数传递的问题,未解决"
description: "shell problem parameter with quote"
keywords: shell, parameter, quote
category: Debug
tags: [bug]
---

## 前言
我希望实现一个功能，能够完美传递参数给一个函数执行，例如：

```sh
curlx() {
    # do something
    curl $*
    # do something
}

$VALUE_1

curlx -X POST -d '{"someKey_1":"'$VALUE_1'", "someKey_2":"'$VALUE_2'" }' www.somecompany.com/api/some
```

然而问题在于shell传递参数时会处理单引号`'`以及双引号`"`

这样给这种用法带来很大的麻烦，我已尝试各种办法，例如通过添加单、双引号的方式，依然没有解决这个问题。

貌似shell并不能完美传递这种带特殊符号的参数？



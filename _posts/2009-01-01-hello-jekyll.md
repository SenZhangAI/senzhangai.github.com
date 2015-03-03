---
layout: post
title:  "Hello Jekyll!"
keywords: "hello, jekyll"
description: "hello jekyll"
category: others
tags: [jekyll]
---

# 1. 标题与文字格式

## 标题

```
# 测试 h1
## 测试 h2
### 测试 h3
#### 测试 h4
##### 测试 h5
###### 测试 h6
```

效果：

# 测试 h1
## 测试 h2
### 测试 h3
#### 测试 h4
##### 测试 h5
###### 测试 h6

## 文字格式

```
**这是文字粗体格式**
*这是文字斜体格式*
~~在文字上添加删除线~~
```

**这是文字粗体格式**
*这是文字斜体格式*
~~在文字上添加删除线~~

# 2. 列表

## 无序列表

```
* 项目1
* 项目2
* 项目3
```

* 项目1
* 项目2
* 项目3

## 有序列表

```
1. 项目1
2. 项目2
3. 项目3
   * 项目1
   * 项目2
```

1. 项目1
2. 项目2
3. 项目3
   * 项目1
   * 项目2
   
# 3. 代码

测试行代码： `code`

测试段落代码：

```ruby
/* hello world demo */
#include <stdio.h>
int main(int argc, char **argv)
{
        printf("Hello, World!\n");
        return 0;
}
```

# 4. 表格

```
|head1|head2|head3|head4
|---|:---|---:|:---:|
|row1text1|row1text2|row1text3|row1text4
|row2text1|row2text2|row2text3|row2text4
|row3text1|row3text2|row3text3|row3text4
|row4text1|row4text2|row4text3|row4text4
```

效果如下：

|head1|head2|head3|head4
|---|:---|---:|:---:|
|row1text1|row1text2|row1text3|row1text4
|row2text1|row2text2|row2text3|row2text4
|row3text1|row3text2|row3text3|row3text4
|row4text1|row4text2|row4text3|row4text4

# 5. 其他

## 图片

```
![图片名称](/static/images/loading.gif)
```

![图片名称](/static/images/loading.gif)

## 链接

```
[链接名称](http://blog.javachen.com)
<http://blog.javachen.com>
```

[链接名称](http://blog.javachen.com)
<http://blog.javachen.com>

## 引用

```
> 第一行引用文字
> 第二行引用文字
```

> 第一行引用文字
> 第二行引用文字

## 水平线

```
***
```

***

# 6. 其他资料

- [Jekyll 扩展的 Liquid 设计](http://havee.me/internet/2013-11/jekyll-liquid-designers.html)
- [Jekyll 扩展的 Liquid 模板](http://havee.me/internet/2013-07/jekyll-liquid-extensions.html)

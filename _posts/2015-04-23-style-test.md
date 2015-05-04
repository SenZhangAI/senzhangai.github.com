---
layout: post
title: "Style Test"
description: "Discription:Style Test"
category: "default"
tags: [Test,Style]
summary: "测试默认风格"
---

# markdown basic #

---

# 这是标题h1 

正文走起啊山东啊soda是的asd大师的红飒 
阿斯顿啊送的号是的好滴
啊啥都撒hodasd

大师的红飒
啊实打实hdoa

## 这是标题h2

正文走起啊山东啊soda是的asd大师的红飒 
阿斯顿啊送的号是的好滴
啊啥都撒hodasd

大师的红飒
啊实打实hdoa

###这是标题h3

正文走起啊山东啊soda是的asd大师的红飒 
阿斯顿啊送的号是的好滴
啊啥都撒hodasd

大师的红飒
啊实打实hdoa

####这是标题h4

正文走起啊山东啊soda是的asd大师的红飒 
阿斯顿啊送的号是的好滴
啊啥都撒hodasd

大师的红飒
啊实打实hdoa

#####这是标题h5

正文走起啊山东啊soda是的asd大师的红飒 
阿斯顿啊送的号是的好滴
啊啥都撒hodasd

大师的红飒
啊实打实hdoa

######这是标题h6

正文走起啊山东啊soda是的asd大师的红飒 
阿斯顿啊送的号是的好滴
啊啥都撒hodasd

大师的红飒
啊实打实hdoa

这是**星号加粗**
这是__短线加强__
这是*星号斜体*
这是_短线斜体_
这是code format `#include<iostream>`
这是引用[Baidu链接用"[]()"实现](www.baidu.com)

1. 这是 item1
2. 这是 item2
3. 这是 item3

* 这是item`*`
* 这是item`*`

- 这是item`-`
- 这是item`-`
    + 这是 nested item`+`
    + 这是 nested item`+`
        * nested item`*`

这是图片：
![Image of Yaktocat](https://octodex.github.com/images/yaktocat.png)

> 这是Blockquotes
> 这是**加粗**

> 这是*斜体*
> 这是code formate `#include<iostream>`
>  >nested Blockquote 
> 这是引用[Baidu链接用"[]()"实现](www.baidu.com)

以下是Multiple lines Code format

```
//使用```实现
x = 1
y = sin(x)
```

    //使用tab
    x = 1
    y = sin(x)

    //使用four space
    x = 1
    y = sin(x)

# GitHub Flavored Markdown

####GFM ignores underscores in words. like this:

> do_this_and_do_that_and_another_thing.

####URL autolinking:

<www.baidu.com>

<https://github.com>
<http://youku.com>

####Strikethrough:

~~Mistaken text.~~


####Syntax highlighting

```ruby
# using highlight
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
```

```
# using ```xxx
require 'redcarpet'
markdown = Redcarpet.new("Hello World!")
puts markdown.to_html
```


```bash
~ $ gem install jekyll
~ $ jekyll new myblog
~ $ cd myblog
~/myblog $ jekyll serve
# => Now browse to http://localhost:4000
```

以下来自[https://help.github.com/articles/github-flavored-markdown/](https://help.github.com/articles/github-flavored-markdown/)

> We use [Linguist](https://github.com/github/linguist) to perform language detection and syntax highlighting. You can find out which keywords are valid by perusing [the languages YAML file](https://github.com/github/linguist/blob/master/lib/linguist/languages.yml).

####Tables

这是表格：


First Header  | Second Header
:------------- | -------------:
~~Content Cell~~  | **Content Cell**
Content Cell  | *Content Cell*

####Task List

tasks lists are my favorite: (此特性没有)

- [x] This is a complete item
- [ ] This is an incomplete item

#### Footnotes

引用[^1]，引用[^2]

#### highlight

==这是highlight标记==

#### quote

This is a "quote"

#### Latex


试一试inline>>>> <div display="inline">$E=MC^2$ </div><<<<inline

<div>
$$
\begin{matrix}
1 & x & x^2 \\
1 & y & y^2 \\
1 & z & z^2 \\
\end{matrix}
$$
</div>


[^1]: 这是引用1.
[^2]: 这是引用2.

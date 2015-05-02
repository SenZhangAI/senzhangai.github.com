---
layout: post
title: "解决redcarpet中Mathjax渲染问题"
description: "解决redcarpet中不能渲染Mathjax的问题，并优化redcarpet配置支持大部分GFM"
keywords: "mathjax, redcarpet, jekyll, blog"
category: programming
tags: [mathjax, redcarpet]
---

## 心路历程

这个不得不吐槽一下，首先测试了jekyll上的四个markdown渲染引擎。

最后把目标锁定在了`redcarpet`和`kramdown`上。

经过测试以及查找资料发现`kramdown`对`GFM`格式支持没有`redcarpet`好，

尤其是对`fenced_code_blocks`的支持方面。遂放弃。

然后发现`redcarpet`对Mathjax的渲染会有问题（以下详述）。

于是查找各种资料和方案。浪费了好几天时间，无奈自己实在忍不了没有`Latex`支持的博客。

吐槽完了，以下是正题。

## 配置redcarpet支持GFM格式

关于github-flavored-markdown（GFM）参见：

<https://help.github.com/articles/github-flavored-markdown/>

GFM格式写起来方便不少，因为实在忍受不了jekyll自带的代码高亮方案，

用GFM的`fenced_code_blocks`要爽很多啊。

方法是修改`_config.yml`文件。加入或者修改如下内容：

```yaml
markdown:    redcarpet
highlighter: pygments
redcarpet:
  extensions: [ "fenced_code_blocks",
                "hard_wrap",
                "autolink",
                "tables",
                "strikethrough",
                "superscript",
                "with_toc_data",
                "highlight",
                "prettify",
                "no_intra_emphasis"]
```

`extensions`内的配置是`redcarpet`的扩展选项，详情参见：

<https://github.com/vmg/redcarpet>

配置好后就可以用到table，fenced_code_blocks等功能了。

## 配置支持Mathjax

关于mathjax语法参考：
<http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-re

为了让redcarpet支持Mathjax。跟kramdown的配置方式一样，首先得在`<head>`中添加如下内容：

```html
</head>
<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML" type="text/javascript"></script>
</head>
```

对于redcarpet，写Latex语法时会有bug，这一点kramdown要好很多。有一个折中的办法解决这一个问题，就是在Latex代码前后加上`<dir></div>`。参考自：<https://github.com/jekyll/jekyll/issues/1888>

以下输入Mathjax无法渲染：

```
$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\ 
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\ 
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$
```

结果如下：
<br>
$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\ 
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\ 
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$
<br>

如果加上`<dir></div>`,即可正常显示：

```
<div>
$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\ 
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\ 
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$
</div>
```

结果如下：

<div>
$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\ 
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\ 
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$
</div>

此外对于inline的形式即类似`$x^2 + y^2 = 1$`的形式，还没有好的办法解决。


---
layout: post
title: "解决redcarpet中Mathjax渲染问题及优化redcarpet的配置"
description: "解决redcarpet中不能渲染Mathjax的问题，并优化redcarpet配置支持大部分GFM"
keywords: "mathjax, redcarpet, jekyll, blog"
category: "Tools"
tags: [mathjax, redcarpet]
---

## 心路历程

这个不得不吐槽一下，首先测试了jekyll上的四个markdown渲染引擎。

最后把目标锁定在了`redcarpet`和`kramdown`上。

经过测试以及查找资料发现`kramdown`对`GFM`格式支持没有`redcarpet`好，

尤其是对`fenced_code_blocks`的支持方面。遂放弃。

然后发现`redcarpet`对Mathjax的渲染会有问题（以下详述）。

于是查找各种资料和方案。浪费了好几天时间，无奈自己实在忍不了没有`Latex`支持的博客。

好在功夫不负有心人，其实解决办法很简单。重点是分析原因。

吐槽完了，以下是正题。

## redcarpet与Mathjax渲染问题分析

由于Mathjax中的**小标语法** `a_b`和**上标语法** `a^b` 有可能会提前被redcarpet解析，这样Mathjax获得的被redcarpet解析后的源文件已面目全非。所以要想避免这一个问题，就得关闭redcarpet对上下标的解析。好在这个配置很简单，只要在`redcarpet.extensions[]`中不要加上`superscript`即可。相关redcarpet配置如下。


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
                "with_toc_data",
                "highlight",
                "prettify",
                "no_intra_emphasis"]
  # DON'T USE"superscript", never use this with Mathjax,this will cause bug
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



## 关于inline的书写格式

参考：<http://docs.mathjax.org/en/latest/start.html#putting-mathematics-in-a-web-page>

通常预设的block形式用`$$...$$`或者`\[...\]`的书写格式。而inline默认用`\(...\)`的书写格式。这是为了避免错误的解析，因为考虑到**$**美元符号使用较为平凡（国内应该用得少吧）。所以如果要开启`$...$`的功能需要额外设置。方式也很简单，参考代码如下：

```html
<!DOCTYPE html>
<html>
<head>
<title>MathJax TeX Test Page</title>
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}});
</script>
<script type="text/javascript"
  src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
</head>
<body>
When $a \ne 0$, there are two solutions to \(ax^2 + bx + c = 0\) and they are
$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$
</body>
</html>
```

When $a \ne 0$, there are two solutions to \(ax^2 + bx + c = 0\) and they are
$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$

## 多行文字

对于如下Mathjax渲染：

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

$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$

本想多行显示，却显示为单行。

如果想多行显示有两种办法：

方案一 通过添加`<dir></div>`,即可多行显示：

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


方案二 用`\\\\` 代替`\\`即：

```html
$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\\\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\\\
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\\\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\\\
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$
```

结果如下：

$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\\\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\\\
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\\\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\\\
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$

---
layout: post
title: "项目文档生成工具介绍"
description: "介绍比较好用的项目文档生成工具readthedocs, sphinx, mkdocs"
keywords: "read the docs, sphinx, mkdocs, 项目文档, 托管, 生成"
category: Tools
tags: [read the docs, sphinx, mkdocs]
---

## 简介

[read the docs](https://readthedocs.org/)是开源文档生成的托管平台和社区。

文档编辑主要有两种格式，reStructuredText（.rst）格式以及Markdown（.md）格式。

其中rst格式用sphinx工具生成，md格式用mkdocs工具生成。

以上两个工具都可以用`pip`下载安装，即`pip install sphinx`和`pip install mkdocs`

简单查看了一下mkdocs对格式的支持，感觉没有sphinx支持地好，所以觉得还是用rst格式编辑文档，用sphinx生成比较合适。

而且readthedocs的官方默认[sphinx theme](https://github.com/snide/sphinx_rtd_theme)非常不错，值得下载学习。

## 功能

大致看了一下，可以实现的功能包括：

1. 用rst或markdown格式自动生成文档，这个基本功能自然不用说

2. 支持Mathjax

3. 支持版本tag，可以自动选择不同版本，这个估计是read the docs扩展的功能

4. 支持toc， table of content导航

5. 支持站内搜索

6. 支持code block，rst格式的code block写法简洁程度也很高。用pygments引擎支持

7. 支持转换为pdf，epub等格式文档。这个太赞了，应该是read the docs扩展的功能

8. 支持代码自动生成文档说明，尤其是python，用其标准的代码说明写法可以自动生成文档。这个没有实践过，但是针对很好用。

9. 还有些细节性的功能，比如说在页面上查看rtf/md源码，

## 学习资料

参考：[Documentation for Read the Docs](https://docs.readthedocs.org/en/latest/index.html)

这个资料写得极为详细，值得参考。

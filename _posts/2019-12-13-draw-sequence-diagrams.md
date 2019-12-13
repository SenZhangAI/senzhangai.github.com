---
layout: post
title: "快速绘制时序图"
description: "how to quickly draw sequence diagrams"
keywords: sequence diagrams
category: Tools
tags: []
---

## 简介
工作需要快速绘制时序图，虽然图形化操作界面可以很方便地绘制，但是需要调整图形比例之类的，还是太慢了，所以这里寻找一个快速的通过代码生成时序图的方式

## 方案一 mscgen

<http://www.mcternan.me.uk/mscgen/>

其教程对时序图有丰富的讲解，值得一看


优点：

* 教程棒
* 命令行操作
* 可以集成到Doxygen

缺点：

* **不支持中文**
* 丑

### 安装及运行

```sh
brew install mscgen

mscgen -T png FILE
```

## 方案二 mermaid

Generate diagrams, charts, graphs or flows from markdown-like text via javascript

<https://mermaid-js.github.io/mermaid/#/README>

<https://mermaid-js.github.io/mermaid/#/sequenceDiagram>

优点：

* 好看，清晰
* 支持各类UML图
* 支持中文

就它了！

### 安装及运行

#### 方案A

```sh
npm install -g mermaid.cli

# <https://github.com/mermaidjs/mermaid.cli#examples>

mmdc -i input.mmd -o output.pdf -w 2048 -H 1536
```

注意生成格式有`.svg` `.png` `.pdf`，其中pdf格式最清晰，所以只考虑生成pdf

#### 方案B Typora(不好！)

在Typora用如下方式使用mermaid

**但是!!!**

* 语法跟mermaid不一样！？
    - 不支持`+/-`的语法`->>+`
    - `participant A as B` 中`A`和`B`和标准反了！！!

不足之处是不支持`->>+`加号这样的语法，另外也不

<pre>
<code>```sequence
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```</code>
</pre>

#### 方案C Vim 非常好!
vim的Markdown-preview插件支持预览

原生的mermaid支持，丝般爽滑！

例如：

<pre>
<code>```mermaid
sequenceDiagram
    Alice ->> Bob: Hello Bob, how are you?
    Bob-->>John: How about you John?
    Bob--x Alice: I am good thanks!
    Bob-x John: I am good thanks!
    Note right of John: Bob thinks a long<br/>long time, so long<br/>that the text does<br/>not fit on a row.

    Bob-->Alice: Checking with John...
    Alice->John: Yes... John, how are you?
```</code>
</pre>

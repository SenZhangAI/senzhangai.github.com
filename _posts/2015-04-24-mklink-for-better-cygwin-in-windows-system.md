---
layout: post
title: "连接Windows和Cygwin的桥梁,mklink和explorer"
description: "介绍一种利用mklink和explorer更好地使用Cygwin的技巧"
keywords: "mklink, explorer, cygwin"
category: "Tools"
tags: [mklink, explorer, Cygwin]
---

2015.12.01 补充：
之所以用mklink是因为ln似乎对windows的文件夹不起作用，
然后实际上Cygwin早就考虑了这一点，考虑了与windows之间的交互。
Cygwin 将`C:`、`D:`等盘符挂载在`/cygdrive/c`以及`/cygdrive/d`等位置，
因此完全可以用cygwin中的`ln`命令来连接或者生成快捷方式。

以下为原文：

在Windows系统下如果要体验linux的某些有用的东东可以装个cygwin，

比如说需要用到Vim,Emacs,python或者我在搭建博客时用到的jekyll等。

通常Cygwin可以看成Windows下的一个独立的软件，那么要想让Cygwin能在

Windows下发挥更大的作用。我觉得以下技巧一定能对您有所帮助。

#### 技巧1 利用explorer命令。在Cygwin下打开Windows下只需要双击的文件或文件夹

在windows下只需要双击就能打开的文件和文件夹，

如果在cygwin下如果打开呢？否则在cygwin和windows下来回切换多麻烦啊。

其实很简单，cygwin下已经默认设置了`explorer.exe`命令（软链接了windows中的explorer.exe）。只要输入

``` bash
explorer.exe target
```

即可，target可以是文件或者文件夹。如果是文件夹就在windows下打开了该文件夹。
**注意target不能加'\''即`explorer.exe target` 正确。`explorer.exe target\`错误**

如果是文件，例如index.html就用windows下的默认程序打开。例如如果默认是chrome就会在chrome中浏览该index.html

减少了cygwin和windows的频繁切换。当然这里还没用到`mklink`命令

#### mklink命令简介

简单说相当于cygwin下的ln命令  `mklink /D `命令对应对Cygwin下的`ln -s`命令。

之所以要用是`mklink /D ` 简单说是因为Cygwin下的ln命令如果链接Cygwin内的文件和Cygwin外的文件时会失效。而软链接在以下情况下很有用。

mklink命令的使用方式：
1. 在cmd中运行，需要Administrator权限。所以要右键单击cmd选择`run as Administrator`。
2. 命令格式为：

``` bat
mklink /D Link Target
```

注意`link`和`target`的顺序，在ln命令中则是`ln -s Target Link`的顺序

#### 技巧2 建立cygwin下windows程序的"快捷方式"

linux下我们都想用简单的方式直接输入几个字母+tab就能启动程序，
如果要包含windows下的程序可以在`.bashrc`中设置环境变量`PATH`包含该文件夹，省事儿，然而这样显得很不**干净**。
我通常做法是建立一个`~/bin`文件夹，里面专门放快捷命令，然后在PATH中添加`~/bin`。

第一个想法是，直接建立该程序的快捷方式放到该文件夹里。---失败
第二个想法是，用Cygwin下的`ln`命令 ---失败
例如我要将我的Sublime Text做成快捷方式，方便打开文件或文件夹则用：

``` bat
mklink /D ~/bin/sublime C:/mySublimeTextPath/sublime_text.exe
```

补充：
后发现可以用`ln`实现，比`mklink`方便，命令如下：

```bash
$ ln -s /cygdrive/c/mySublimeTextPath/sublime_text.exe /usr/bin/sublime_text
```

感觉放在/usr/bin中更方便，免得还要添加到PATH环境变量。

#### 技巧3 建立cygwin下windows文件夹的"快捷方式"

比如在cygwin下要频繁打开某个很长路径的文件夹，每次输入cd都是一件很痛苦的事情，（顺带说一下，cygwin也可以输入`cd C：`命令到C盘。对应于Cygwin中的`/cygdrive/c`目录。）
那么只需来一次`mklink`命令搞定：

``` bat
mklink /D ~/shortcut C:/long/long/path/folder
```

则每次只需要`cd shortcut`了

补充：
后发现mklink还是太麻烦了，可以直接用`ln`实现，方法是：

```bash
$ ln -s /cygdrive/c/long/long/path/folder ~/shortcut
```

#### 技巧3 `共享`cygwin 和windows下的某些配置文件夹。

例如,我在windows下装了`git`h在cygwin下也装了`git`。`git`如果要`push`需要`.ssh`文件夹中的配置，那我何必分别配置两个文件夹呢。只需要`mklink`这两个文件夹即可。同理可应用于`.vim`。这个还没试过。

其他技巧不再累述。总之:
**mklink**对应于于linux下的**ln**命令，因为在cygwin下ln命令不能直接作用于cygwin之外的windows下的文件或文件夹，故用mklink代替。

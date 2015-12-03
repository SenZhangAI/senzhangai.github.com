---
layout: post
title: "我的Sublime Text 3配置"
description: "my setup and config for Sublime Text 3"
keywords: sublime text, setting, setup, 配置, 安装, config
category: tools
tags: [sublime text]
---

## 前言
用Sublime Text已有很长时间，但每次重新安装都忘记了上次的配置，很不愉快啊，
所以现在较为完整地总结一下吧，
如果能做到像vim那样自动化安装就好了，可惜比较麻烦，以后有待改进。

此外，为了避免同时用vim和sublime text的学习成本，我觉得坚持只用vim

## 1. install Package-Control

需要用<kbd>ctrl</kbd> + <kbd>~</kbd>进入Console，输入对应命令安装，
详情参见：

<https://packagecontrol.io/installation>

## 2. 安装必要的package
Sublime Text所有的package可在如下链接中检索：
<https://packagecontrol.io/browse>

有如下package推荐安装：

### CoolFormat
自动化代码格式生成。
主页： <http://akof1314.github.io/CoolFormat/>

快捷键：
Ctrl + Shift + Alt + Q : Quick Format
Ctrl + Shift + Alt + S : Selected Format

### SidebarEnhancements
扩展sidebar功能，最好仅在Sublime Text3中用。

### BracketHighlighter
高亮Bracket

### Alignment
用于对齐格式

选择区域，按`Ctrl + Alt + a`即可对齐。

### Docblockr
强大的文档注释功能

常用的用法是在`/*` 或者 `/**`后按<kbd>Tab</kbd>。

详见： <https://github.com/spadgos/sublime-jsdocs>

### SublimeLinter
检查语法错误，最好用于Sublime Text 3，对于Sublime Text 2 往后不再支持。

### ConvertToUTF8
防中文乱码必备，什么GBK之类都自动转化成UTF-8格式。

### Git 以及 GitGutter
直接在sublime text中使用git，
以及GitGutter显示修改行。

用`Ctrl + Shift + P`来打开命令执行窗口，
比较有用的命令如下：
Git: Graph All (gga)


**需要注意的是系统PATH环境变量中必须有git**，否则会出现乱码以及无法使用。
详情参见：<http://www.zhihu.com/question/20537304>

如果需要gitk或者GUI则也得需要对应的程序在path环境变量中，
毕竟本软件包只是一个在ST中可直接与git交互的桥梁。

我用的Cygwin，git在`Cygwin\bin`文件夹中，
因为直接在windows环境变量path中添加`D:\Cygwin\bin`即可。



### C Improved
自带的C高亮弱爆了

### AllautoComplete
ST默认的autoComplete仅考虑当前文件，
此软件包将使得代码提示可以收集所有打开的文件的关键词。

### IMESupport
解决中文输入法单词候选框不跟随光标的问题。

## 3. 可选的package
### ColorHighlighter
用于高亮颜色，例如`#0000FF`在ST中将辅助显示蓝色。`"red"`将辅助显示为红色。
总之前端需要的格式都能搞定。
详细参见： <https://packagecontrol.io/packages/Color%20Highlighter>

### ColorPicker
颜色拾取及选取

### SublimeREPL
用于直接在Sublime中单步运行解释型语言，例如R、Python、Ruby、Scala等
还是非常强大的。

参见：<https://packagecontrol.io/packages/SublimeREPL>

### Cscope
不得不说，Cscope真的很强大，只是sublime的插件没用过，不知道是否好用。

## 4. 修改user-setting

我是如下设置的：

```javascript
{
	"create_window_at_startup": false,
	"font_size": 14.0,
	"highlight_line": true,
	"highlight_modified_tabs": true,
	"ignored_packages":
	[
		"Vintage"
	],
	"rulers":
	[
		80
	],
	"save_on_focus_lost": true,
	"show_encoding": true,
	"translate_tabs_to_spaces": true,
	"trim_trailing_white_space_on_save": true
}
```
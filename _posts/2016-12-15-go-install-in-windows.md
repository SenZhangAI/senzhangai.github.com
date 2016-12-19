---
layout: post
title: "Go在Windows下的环境配置，Cygwin-Sublime-vim"
description: "Go install in Windows"
keywords: Go, cygwin, sublime, vim
category: tools
tags: [Go, install]
---

## 前言

Windows下估计Sublime配置使用Go比较通用。
但我觉得用Cygwin有些场合，有些操作会方便一些，Vim没试过，但应该也适合某些场合使用。
故写此文配置Go使得方便应用与Windows环境下的Sublime以及仿Linux环境下的vim。

## 安装
直接下载MSI文件安装即可，
下载地址： <https://golang.org/dl/>

## 环境变量配置
无论Cygwin环境下还是Windows环境下，都需要配置 GOPATH，GOBIN等环境变量，
Cygwin以配置到我的[oh-my-zsh](https://github.com/SenZhangAI/oh-my-zsh-sen)配置文件中
Windows也要在系统环境变量中设置GOPATH，GOBIN，其中GOPATH可以是多个目录，例如Windows下用`;`分隔

我的配置是：

```
GOPATH=D:\Cygwin\home\Sen\GoWorkSpace
GOBIN=D:\Cygwin\home\Sen\GoWorkSpace\bin
```

以上配置环境重装以后或许需要调整路径。

## Sublime text
1. Install Package: GoSublime
2. 配置`Preferences`->`package settings`-> `GoSublime`-> `Settings-Users`

通常GoSublime会用默认的Windows环境，需要覆盖时才需要改这些。

配置类似于如下：

```json
{
    "env": {
        "GOPATH": "F:/mygo",
        "GOROOT": "E:/Go"
    }
}
```

3. 配置只用编译运行 `Tools` -> `Build System` -> `New Build System`

写入：

``` json
{
    "shell_cmd": "C:/Go/bin/go run $file"
}
```

文件保存为GoBuild或者其他名字，快捷键单击`Tools`菜单查看

## Vim
一般装vim-go插件，这是一个集成的插件包，会自动下载比较好的插件，可以配合tagbar等很多插件
另一个插件为gocode。

可惜的是该插件对Cygwin的支持不好，主要因为Windows和Cygwin下路径格式混乱，所以Windows下开发只能用Sublime Text，然而Sublime Text的vim插件又太弱了。
所以转Ubuntu下开发。


---
layout: post
title: "配置Cygwin/X默认终端为gnome-terminal"
description: "将Cygwin/X图形界面的终端由xterm改为gnome-terminal"
keywords: "Cygwin, gnome-terminal, xterm, 默认, 终端, 好用"
category: "tools"
tags: [Cygwin, gnome-terminal, Xwin]
---

## 概述

由于部分应用比如gitk, gvim, IDLE等图形界面软件需要用到gtk等库，在默认的cygwin-terminal终端下没法运行，所以需要安装[Cygwin/X](http://x.cygwin.com/)来支持图形界面的linux系统。本质上Cygwin/X是以X11为基础的界面系统。

Cygwin/X上默认的终端是Xterm， 但是配置比较麻烦，需要非常细致地配置`~/.Xresources`文件才勉强能看，且使用中对中文的支持不好，出现中文文字间乱码。相关配置在此略过。

而gnome-terminal作为Ubuntu的默认终端，个人觉得还蛮不错的，至少简单配置下比Xterm好看多了，也没有中文乱码的现象。

当然Cygwin/X下的中文输入问题暂时还没解决，貌似只能用ibus输入法，这个。。。。看以后怎么解决吧。

## 安装Cygwin\X环境

首先用Cygwin的[安装文件](https://cygwin.com/install.html)安装相关软件，至少需要选择**xinit**和**X-start-menu-icons**安装包，[其他参考此处](http://x.cygwin.com/docs/ug/setup.html#setup-cygwin-x-installing)

然后还需要安装**gnome-terminal**终端，以及需要的图形软件，比如**gitk**、**gvim**、**idle**等等。

## 修改默认终端为gnome-terminal

搜索网上大部分资料都没有相关介绍，后来自己摸索了一下才搞定。

方法是将`/etc/X11/xinit/startxwinrc`拷贝到`~/.startxwinrc`中，即：

```bash
~ $ cp /etc/X11/xinit/startxwinrc ~/.startxwinrc
```

然后把`.startxwinrc`中的`/usr/bin/xterm`终端程序替换为gnome-terminal终端程序`/usr/bin/gnome-terminal`

这样应该就可以在启动Xwin时默认启动gnome-terminal了

启动Xwin的方法为在Cygwin终端中执行:

```bash
~ $ startxwin
```

或者在windows系统中双击`XWin Server`图标。

如果遇到错误可以查看`~/.xsession-errors`文件。看有何错误信息。

## 优化

上述方法应该可以默认启动gnome-terminal了，只是速度稍慢，可以把`~/.startxwinrc`中无关的启动内容删除掉，这样就可以更快启动gnome-terminal终端。

最终修改后的`.startxwinrc`文件内容为：

```sh
#!/bin/sh
if [ -z "$GDMSESSION" ]; then
    errfile="$HOME/.xsession-errors"
    if ( umask 077 && cp /dev/null "$errfile" 2> /dev/null ); then
        chmod 600 "$errfile"
        [ -x /sbin/restorecon ] && /sbin/restorecon $errfile
        exec > "$errfile" 2>&1
    else
        errfile=$(mktemp -q /tmp/xses-$USER.XXXXXX)
        if [ $? -eq 0 ]; then
            exec > "$errfile" 2>&1
        fi
    fi
fi

. /etc/X11/xinit/xinitrc-common

if [ -x /usr/bin/gnome-keyring-daemon ] ; then
    eval `/usr/bin/gnome-keyring-daemon --start`
    export GNOME_KEYRING_CONTROL GPG_AGENT_INFO SSH_AUTH_SOCK
fi

[ -x /usr/bin/gnome-terminal ]  && /usr/bin/gnome-terminal &
[ -x /usr/bin/fbpanel ] && exec /usr/bin/fbpanel -p multiwindow
```

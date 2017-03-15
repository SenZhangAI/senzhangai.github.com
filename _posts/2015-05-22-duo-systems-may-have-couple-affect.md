---
layout: post
title: "双系统下被引导系统也受引导系统影响"
description: "发现ubuntu中蓝牙启动也会影响到被引导的windows系统的蓝牙启动"
keywords: "双系统, 蓝牙, 没有显示, 正常"
category: Debug
tags: [debug]
---

之前一直以为双系统是相互独立不会相互影响的，这次一次偶然的经历发现其实不是。

我的电脑是用ubuntu系统的grub引导windows系统的启动。

启动中发现boot阶段蓝牙的灯是亮的，然后到了grub的时候蓝牙关闭了，随后进入windows系统没有蓝牙。

尝试了各种方法都没有解决，后来想到会不会双系统相互有影响，没想到是真的！

进入ubuntu系统启动蓝牙后，关机重启进入win7，问题解决了。

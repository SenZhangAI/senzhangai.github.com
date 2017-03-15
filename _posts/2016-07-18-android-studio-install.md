---
layout: post
title: "Android Studio安装攻略"
description: "install Android Studio"
keywords: Android Studio, install, performance, intellij, IDEA
category: Tools
tags: [android]
---

## Android Studio简介
Android Studio是Google基于intellij IDEA的基础上开发的专门针对Android的集成开发环境(IDE)，
随着Android Studio以及intellij IDEA版本的更新，其或多或少存在一定差异，
总体而言AS相对IDEA更针对Android，而intellij IDEA对于整个JAVA开发更全面。

AS有内置的虚拟机来模拟实际手机运心效果，IDEA应该会用第三方的模拟器Genymotion（没试过）。

无论AS，IDEA都是吃内存大户，对于我的4G电脑而言是一个挑战，
所以本文重点在于如何配置一个适用于4G内存的相对流畅的开发环境。
当然，用更大的内存(最好32G以上)以及SSD硬盘才是终极解决之道。

## 安装环境
* 内存: 4G
* 硬盘: 机械硬盘5400转
* 系统: Win10
* Android Studio版本: 2.1

## 安装步骤
本文只记录重要的点，无所谓的略过

### 1. 下载Android Studio安装软件
[下载地址](https://developer.android.com/studio/index.html)

我猜应该需要改hosts或者其他vpn翻墙，否则可能会打不开

### 2. 安装Android Studio 选择组件
在SDK Component Setup中会提示安装选项

其中包括1. Android SDK Platform, 2. 6.x版本的API， 3. Android Virtual Device

我的考虑是都装上，如果AVD的性能有问题再考虑改一下配置，反正也就多占用点硬盘空间

### 3. 模拟器内存设置
我的4G电脑顶多1G内存，再大要卡死

### 4. 下载应用组件
这里应该自动下载步骤3中的应用，估计要改hosts或者其他翻墙手段才能下

这里会安装很长时间，大概1小时，安装后提示错误信息：

```
Unable to create a virtual device: Missing system image required for an AVD setup
```

点击Finish结束

## 配置

### 1. 取消部分plugin

参见 [stackoverflow-AS-memory](http://stackoverflow.com/questions/27176353/android-studio-takes-too-much-memory)

```
You can disable VCS by going to File->Settings->Plugins and disable the following:

    CVS Integration
    Git Integration
    GitHub
    Google Cloud Testing
    Google Cloud Tools Core
    Google Cloud Tools for Android Studio
    hg4idea
    Subversion Integration
```

### 2. 官方Instant Run tips
AS 2.0对于编译慢的问题进行了改进，加入了Instant Run的功能，
这样除了第一次编译耗时比较长之外，
接下来的编译应该只编译改动的部分，所以会快一些，这里列出tips

1. gradle版本需要大于2.0.0，minSdkVersion应该大于等于15

### 3. Gradle配置
根据文章[Android Studio 编译、同步慢的解决方法](blog.csdn.net/fuchaosz/article/details/51146091)
Grale每次都会同步是导致慢的一个重要原因

简单将Gradle配置为Offline work，大约减少了1分多钟

### 4. 用真机测试而非虚拟机
开虚拟机需要非常多的内存，我的4G电脑开AS已经够呛，更不用说开虚拟机。

真机测试的方式是首先手机要授权可被测试:

勾选  设置-开发者选项-USB调试

然后直接USB连电脑就可以了，AS会自动安装应用程序包apk到手机上测试。

### 5. 用软件管家清理内存
网上有说AS使用过程中在缺内存情况下才会执行gc，这样内存用着用着就不断增加了。

一种可行的办法是用软件管家之类的软件主动释放内存，鼠标单击清理内存，效果很不错。

当然，从性能角度说，除非内存快满了，否则内存使用还是多多益善，可以加快速度。

因为垃圾清理后，说不定后来某个地方正好需要用，又需要重新计算，反而效率降低。

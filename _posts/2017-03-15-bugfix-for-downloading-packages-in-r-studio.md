---
layout: post
title: "解决R Studio下载Package失败的问题"
description: "bugfix for downloading packages in R Studio"
keywords: R studio, packages, download, failed,
category: Debug
tags: [R, debug, config]
---

## 前言
解决这个问题费了几个小时才好不容易找到，不然包都下载不了还怎么玩。。。

## 解决办法

安装Package的时候提示出错：

```
Warning in install.packages :
  unable to access index for repository http://cran.rstudio.com/src/contrib:
  cannot open URL 'http://cran.rstudio.com/src/contrib/PACKAGES'
```

在网上找了一圈，有说https不行，要用http的，改了global options没有效果。

我才会不会是mirrors没选好，所以用选mirrors的方式：

搜索`package is not available` 关键字，查到说因为被墙的缘故，
所以我尝试用vpn，连接anyconnect，但依然没有效果。

后来在stackoverflow中搜索关键字`InternetOpenUrl failed`找到一个方法：

```r
options(download.file.method="libcurl")
```

哈哈！ 搞定了！

我猜测这应该是一个小trick，绕过了某个无法连接的步骤。

前提是我电脑里装了Cygwin，所以已装了curl

如果用手动下载报文件的方法，因为各种包依赖，所以应该会很痛苦，还好解决了 ^_^

## 更进一步

首先，原mirror太慢了，嗯，需要配置默认的mirror，这很简单，网上随便搜就是。

其次，以上trick也不会自动保存，每次重启R Studio后就失效了，所以需要改配置。

配置文件通常为默认路径的`.Rprofile`或者我修改的是`C:\Program Files\R\R-3.3.3\etc\Rprofile.site`

添加以下内容：

```r
local({r <- getOption("repos")
       r["CRAN"] <- "http://mirrors.tuna.tsinghua.edu.cn/CRAN"
       options(repos=r)})

options(download.file.method="libcurl")
```

注意文件可能受到保护无法修改，我将其复制到桌面，改好后再复制回来即可。

测试结果： 完美！

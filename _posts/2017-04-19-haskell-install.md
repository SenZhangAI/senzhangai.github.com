---
layout: post
title: "haskell环境配置小结"
description: "haskell intallation"
keywords: haskell, install
category: Tools
tags: [haskell]
---

## 前言
总结haskell环境配置，网上众说纷纭，也不知道哪种好，所以总结一下。

这里，我的环境是Windows+Cygwin。

## 安装haskell基本款

<https://www.haskell.org/platform/windows.html>

我安装的是Haskell Platform Core版 ，包括GHC和Cabal和Stack。
这仅包括了core libraries，以及Cabal和Stack用于下载安装packages
这是最最基本的可自给自足的环境。

```
Stack is a cross-platform build tool for Haskell that handles management of the toolchain (including the GHC compiler and MSYS2 on Windows), building and registering libraries, and more.

The Cabal build system, which can install new packages, and by default fetches from Hackage, the central Haskell package repository.

The command line tools to download and install packages are either cabal or stack, each having different workflows.

Hackage is a repository of packages to which anyone can freely upload at any time. The packages are available immediately and documentation will be generated and hosted there. It can be used by cabal install.

Note that if you are not in a sandbox, this will install the package globally, which is often not what you want, so it is recommended to set up sandboxes in your project directory by running `cabal sandbox init`.
```

装完后，就可以用GHCi，这是命令行的haskell环境，以及WinGHCi，这是简单的GUI的haskell环境。

TODO 至少先看看官方文档，再找工具链。

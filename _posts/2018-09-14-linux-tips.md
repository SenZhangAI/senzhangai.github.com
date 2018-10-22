---
layout: post
title: "linux tips"
description: "linux tips"
keywords: linux tips
category: programming
tags: [linux]
---

## 前言

整理一些有用linux命令，todo...

## 常用监控命令

```bash
# 查看网络占用（macOS）
$ lsof -i
# 查看有哪些终端登录
$ who
```

## zsh

### profile

有时[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)配置后反应变慢了，这样首先通过如下命令定位哪一步使得系统变慢 看卡在哪里

```bash
zsh -xv
```


例如我发现我卡在`eval "$(chef shell-init zsh)"` 这一步，这一步是用来初始化[chef](https://github.com/chef/chef)环境，
参见[Configuring ChefDK](https://docs.chef.io/chefdk_setup.html)

类似的，我也曾可能卡在nvm的初始化`source $(brew --prefix nvm)/nvm.sh`

因为不是每次都用到这些功能，所以干脆做成一个alias快捷命令，即可解决。

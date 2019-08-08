---
layout: post
title: "linux tips"
description: "常用的linux命令"
keywords: linux tips
category: programming
tags: [linux]
---

## 前言

整理一些有用linux命令，todo...

## 常用查看监控命令

```bash
# 查看网络占用（macOS）
$ lsof -i
# 查看网络占用（Linux）
$ netstat -nutlp
# 查看有哪些终端登录
$ who

# 查看当前目录文件各文件大小
# -c 除了列出各子项大小，还给一个cumulative total 总计大小
$ du -ch *
$ du -sh *
```

## 网络

### SSH隧道

远程的服务器，例如阿里云可能对外网屏蔽了很多端口，或者比如远程服务器的redis仅支持本地端口`127.0.0.1`访问，现在仅知道ssh登录，然而想在远程服务器上测试，跑不了怎么办？

答案就是通过SSH隧道，原理就是一个代理，将本地的地址和端口号映射到远程服务器。

本地先运行ssh隧道：

```sh
# ssh -L local_port:remote_ip:remote_port remote_user@remote_ip
ssh -L 9999:127.0.0.1:6379 root@remote_ip
```
然后本地测试一下是否可以访问远程原本无法访问的redis服务器：

```sh
redis-cli -p 9999
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

## 其他

Centos不重启修改hostname的方法: `hostnamectl set-hostname NewName`

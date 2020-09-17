* * *

layout: post
title: "linux tips"
description: "常用的linux命令"
keywords: linux tips
category: programming

## tags: [linux]

## 前言

整理一些有用linux命令，todo...

## 查询帮忙

Linux的man手册共有以下几个章节：

1. Standard commands (标准命令)
2. System calls (系统调用)
3. Library functions (库函数)
4. Special devices (设备说明)
5. File formats (文件格式)
6. Games and toys (游戏和娱乐)
7. Miscellaneous (杂项)
8. Administrative Commands (管理员命令)

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

# 查询常见编码格式
$ file -I somefile
# result: somefile: text/plain; charset=us-ascii
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

补充: [SSH 命令的三种代理功能（-L/-R/-D）](https://zhuanlan.zhihu.com/p/57630633)

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

## linux库函数

### time

获取实现应该用linux的`clock_gettime()`或者macOS的`mach_absolute_time()`而不用`gettimeofday()`,因为`gettimeofday`不是单调递增的。

### centos系统

升级g++

```sh
sudo yum install centos-release-scl
sudo yum install devtoolset-7
scl enable devtoolset-7 bash
```

## 资源

## 优化及配置



修改backlog queue，作为服务器优化的手段

参见 <https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/>

```
sudo sysctl -w net.core.somaxconn=4096
```

ENOSPC: System limit for number of file watchers reached，但是

解决办法:

```sh
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

linux ulimit 调优

<https://www.cnblogs.com/sunsky303/p/8359592.html>

## 安全

ssh 登录黑名单，但是黑名单机制还是不如白名单

<https://blog.csdn.net/z13615480737/article/details/83028304>

用ssh私钥的方式最好


Nginx 配置ssl报错

NGINX SSL: error:0200100D:system library:fopen:Permission denied

查看错误编码 0200100D 为权限问题，但chown, `ls -al` 查看权限都没发现问题

根据 <https://serverfault.com/questions/540537/nginx-permission-denied-to-certificate-files-for-ssl-configuration>

应该是selinux的原因，可能文件来自网络，需要`restorecon`


## 常用安装

### Jenkins
按照怎么省心怎么来的原则，Jenkins yum安装比较方便

参见 <https://www.jenkins.io/doc/book/installing/#red-hat-centos>

```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
sudo yum upgrade
sudo yum install jenkins java-1.8.0-openjdk-devel
sudo systemctl daemon-reload
sudo systemctl start jenkins
```

---
layout: post
title: "Virturebox安装配置Centos"
description: "install centos in virturebox"
keywords: centos, virtualbox
category: Programming
tags: [centos, virtualbox]
---

## 前言

virtualbox用拷贝的Centos镜像不能直接联网，简单来说，是旧的配置不对，需要修正配置

## 配置NAT和host-only
为了最终的目标虚拟机与主机互联（ssh），以及虚拟机联网，需要两个接口，一个NAT接口，一个host-only接口，

Host-only相当于虚拟机和宿主机通过交叉线相连；
NAT，宿主机相当于虚拟机的路由器；

Host-only 允许 虚拟机与宿主机互访，但其不能直接访问外网
NAT，允许虚拟机访问主机，并间接允许虚拟机连接外网，但外网访问不了虚拟机

其中NAT接口用dhcp方式，host-only接口用静态ip，例如`192.168.56.xxx`

其中宿主host-only用到的ip需要与全局的host-only在同一个网络段内，ip最末位不同，
例如全局host-only ip选用`192.168.56.2`，虚拟机host-only ip选用`192.168.56.101`等。

全局host-only设置: 在virtualbox首页面点击 管理，主机网络管理器
虚拟机host-only设置: 点击该虚拟机，设置，网络

按照网上任意一篇`Centos virtualbox 联网`作为关键字的帖子即可查看详细配置。

如上硬件配置完成后，还得针对该Ethernet网卡软件配置，通常是`eth0`，`eth1`等，对于Centos文件地址为`/etc/sysconfig/network-scripts/`
注意通常eth0的MAC小于echo1，

否则NAT，host-only配置反了也出错，然后将对应的NAT配置为dhcp，host-only配置为静态ip，例如如上的例子`192.168.56.101`

然后`service network restart`，这时候还可能出错，
因为有个自动缓存的配置不对，文件地址为`/etc/udev/rules.d/70-persistent-net.rules`

简单的解决办法是：

```bash
rm -f /etc/udev/rules.d/70-persistent-net.rules
reboot
service network restart
```

详见： <https://blog.csdn.net/xiaobei4929/article/details/40515247>

### 测试

虚拟机：

```bash
ping www.baidu.com
ping <主机host-only ip> # 192.168.56.XXX
```

```bash
ping <虚拟机host-only ip> # 192.168.56.XXX
```

## 开启Centos的ssh服务

```bash
yum install openssh-server
service sshd start
```

ssh 默认配置即可，无需修改

## 共享文件夹
为了在主机上编辑代码，同时在虚拟机上编译执行代码，就需要两端的文件同步，
对吧virtualbox，最简单的方法应该是设置同步文件夹，直接搜关键词即可，
例如 <http://www.cnblogs.com/xing901022/p/5774677.html>

权限问题参考：
<https://blog.csdn.net/superbfly/article/details/50844206>

```bash
sudo usermod -aG vboxsf $(whoami)
```

## 安装mysql

参考 <https://www.cnblogs.com/renjidong/p/7047396.html>

安装完以后测试可本地启动mysql但无法远程访问

### 授权
授权任意ip访问 `root@'%'`，

```sql
-- GRANT all privileges ON database_name.table_name TO 'username'@'ip or localhost' IDENTIFIED BY 'password';

GRANT all privileges ON *.* TO 'root'@'%' IDENTIFIED BY '';

FLUSH privileges;
```

但依然报错

### 测试端口绑定
查看端口占用 `netstat -tunlp | grep mysql` 发现mysql已经确实占用3306端口没有问题。
netstat命令含义参见 <https://www.cnblogs.com/CEO-H/p/7794306.html>

### 测试远程端口连接
`telnet <虚拟机IP> <端口号>`

发现ssh 22端口没有问题，但3306端口连不上，猜测是防火墙的问题。

### 设置防火墙

参考 <https://www.cnblogs.com/liyuanhong/articles/7064582.html>
了解`init.d目录` 可以参考 <https://www.cnblogs.com/zhaopengcheng/p/5806379.html>

`ps -ef | grep iptables` 发现防火墙确实运行了。

`service iptables stop` 关闭防火墙后再用telnet测试3306端口没有问题，
用DataGrip测试连接也没有问题，看来除了授权还有防火墙的问题。

`service iptables start` 然后 `/etc/init.d/iptables status`可以查看防火墙配置状态。

<https://www.cnblogs.com/fefjay/p/6044413.html>
介绍了直接修改`/etc/sysconfig/iptables`的方法，更实用。

即`vi /etc/sysconfig/iptables` 添加开放端口的规则，
然后`service iptables restart`

除了mysql的3306 端口，6379端口，8080端口(api网关测试默认端口)也要打开。

注意：iptables方法适用于Centos6,对于Centos7的防火墙配置似乎有更好的方法。
`cat /etc/redhat-release` 查看版本。

## 安装Redis

```bash
yum install redis
service redis start
```

---
layout: post
title: "通过nfs同步主机与虚拟机"
description: "nfs-sync-host"
keywords: nfs, sync
category: programming
tags: [nfs]
---

## 前言

因开发需要用虚拟机，便出现需要本地代码与虚拟机内代码同步的问题，一种解决方案是通过开启nfs服务。这里记录其过程。

回过头来看，用vagrant管理虚拟机是一个长远来看更便捷的方式。

## 步骤

Mac 与 虚拟机中的Centos 6同步文件夹的一种方式是Centos开启nfs服务，
然后Mac mount到Centos，

可以通过主机调用 `rpc info -p 192.168.xxx.xxx` 来测试是否可以连上虚拟机的nfs服务，

正确的返回例如：

```
  program vers proto   port
    100000    4   tcp    111  rpcbind
    100000    3   tcp    111  rpcbind
    100000    2   tcp    111  rpcbind
    100000    4   udp    111  rpcbind
    100000    3   udp    111  rpcbind
    100000    2   udp    111  rpcbind
    100005    1   udp  36301  mountd
    100005    1   tcp  47231  mountd
    100005    2   udp  35004  mountd
    100005    2   tcp  38020  mountd
    100005    3   udp  37020  mountd
    100005    3   tcp  50207  mountd
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    2   tcp   2049  nfs_acl
    100227    3   tcp   2049  nfs_acl
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    2   udp   2049  nfs_acl
    100227    3   udp   2049  nfs_acl
    100021    1   udp  44956  nlockmgr
    100021    3   udp  44956  nlockmgr
    100021    4   udp  44956  nlockmgr
    100021    1   tcp  37992  nlockmgr
    100021    3   tcp  37992  nlockmgr
    100021    4   tcp  37992  nlockmgr
```

看到其中有nfs项

容易出错的地方一个是防火墙，试一试 `service iptables stop`禁用防火墙

然后需要开启 `service rpcbind start` 以及 `service nfs start`
通常就可以了。

然后mount nfs即可：

```bash
sudo mount -rw -t nfs -o resvport 192.168.xxx.xxx:/your/vm/folder ~/your/host/folder
```

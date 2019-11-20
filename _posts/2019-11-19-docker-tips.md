---
layout: post
title: "docker tips"
description: "docker tips"
keywords: docker, tips
category: Tools
tags: [docker, tips]
---

## macOS 共享文件夹

当尝试共享macOS文件夹

```sh
docker run -it -v /myinclude:/usr/include centos:latest
```

会得到错误信息

```
docker: Error response from daemon: Mounts denied:
The path /myinclude
is not shared from OS X and is not known to Docker.
You can configure shared paths from Docker -> Preferences... -> File Sharing.
See https://docs.docker.com/docker-for-mac/osxfs/#namespaces for more info.
.
```

如果宿主机是Centos系统则没有此问题

解决方案参见 <https://forums.docker.com/t/docker-shared-folders-on-a-mac/26964/6>

实际上是macOS的权限问题，在`Preferences->File Sharing`中可以看到设置了几个文件夹可以共享访问，例如`/tmp`文件夹权限没有问题

所以改为如下命令即可：

```sh
# 将宿主机共享目录设置在/tmp目录下
docker run -it -v /tmp/myinclude:/usr/include centos:latest
```


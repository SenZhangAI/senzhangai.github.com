---
layout: post
title: "源码编译实验tidb"
description: ""
keywords:
category:
tags: []
---

## 任务

题目描述：

本地下载 TiDB，TiKV，PD 源代码，改写源码并编译部署以下环境：
* 1 TiDB
* 1 PD
* 3 TiKV 
改写后：使得 TiDB 启动事务时，能打印出一个 “hello transaction” 的 日志

## 源码编译步骤

由于时间仓促，这里记录其简要步骤

### 0. 环境

本次测试机器操作为 macOS Catalina

### 1. 下载源码

```sh
git clone --depth=1 https://github.com/pingcap/tidb
git clone --depth=1 https://github.com/pingcap/pd
git clone --depth=1 https://github.com/tikv/tikv
```

### 2.编译源码

tidb产品编译相当简单，每个项目里都有Makefile

```sh
cd tidb && make
cd pd && make && mkdir -p ./bin && cp ./target/release/tikv-server bin
cd tikv && make
```

其中tidb和pd是golang，而tikv是rust，通过brew可以比较方便地安装其编译器

```sh
brew install rust
brew install go
```

### 3. 运行

#### 3.1 运行tidb

```sh
cd tidb && ./bin/tidb-server
```

#### 3.2 运行tidb

运行pd和tikv参见 <https://tikv.org/docs/4.0/tasks/deploy/binary/>

```sh
cd pd
./bin/pd-server --name=pd1 \
                --data-dir=pd1 \
                --client-urls="http://127.0.0.1:2379" \
                --peer-urls="http://127.0.0.1:2380" \
                --initial-cluster="pd1=http://127.0.0.1:2380" \
                --log-file=pd1.log
```


#### 3.3 运行tikv

```sh
cd tikv

./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                --addr="127.0.0.1:20160" \
                --data-dir=tikv1 \
                --log-file=tikv1.log

./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                --addr="127.0.0.1:20161" \
                --data-dir=tikv2 \
                --log-file=tikv2.log

./bin/tikv-server --pd-endpoints="127.0.0.1:2379" \
                --addr="127.0.0.1:20162" \
                --data-dir=tikv3 \
                --log-file=tikv3.log
```

这里可能运行tikv失败，查看日志报错:

```
the maximum number of open file descriptors is too small, got 256, expect greater or equal to 82920
```

解决方案参见 <https://unix.stackexchange.com/questions/108174/how-to-persistently-control-maximum-system-resource-consumption-on-mac>


## 改写源码

目标是改写后：使得 TiDB 启动事务时，能打印出一个 “hello transaction” 的 日志



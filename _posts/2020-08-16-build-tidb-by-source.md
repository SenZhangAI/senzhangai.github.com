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

## 1. 源码编译步骤

由于时间仓促，这里记录其简要步骤

### 1.1 环境

本次测试机器操作为 macOS Catalina

### 1.2 下载源码

```sh
git clone --depth=1 https://github.com/pingcap/tidb
git clone --depth=1 https://github.com/pingcap/pd
git clone --depth=1 https://github.com/tikv/tikv
```

### 1.3 编译源码

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

### 1.4 运行

#### 1.4.1 运行tidb

```sh
cd tidb && ./bin/tidb-server
```

#### 1.4.2 运行tidb

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


#### 1.4.3 运行tikv

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

```sh
sudo launchctl limit
```
```
	cpu         unlimited      unlimited
	filesize    unlimited      unlimited
	data        unlimited      unlimited
	stack       8388608        67104768
	core        0              unlimited
	rss         unlimited      unlimited
	memlock     unlimited      unlimited
	maxproc     2784           4176
	maxfiles    256            524288
```

解决方案参见 <https://unix.stackexchange.com/questions/108174/how-to-persistently-control-maximum-system-resource-consumption-on-mac>

配置 `/Library/LaunchDaemons/limit.maxfiles.plist` 并重启后：

```sh
sudo launchctl limit

	cpu         unlimited      unlimited
	filesize    unlimited      unlimited
	data        unlimited      unlimited
	stack       8388608        67104768
	core        0              unlimited
	rss         unlimited      unlimited
	memlock     unlimited      unlimited
	maxproc     2784           4176
	maxfiles    262144         524288
```

问题解决。

## 2. 改写源码

目标是改写后：使得 TiDB 启动事务时，能打印出一个 “hello transaction” 的 日志

### 2.1 定位代码
通过关键字`Begin`、`Commit`、`Transaction` 或者 `txn` 查找到关键代码，理解其含义。

可以发现事务是由底层的tikv支持，tidb在其上做了一层封装，并提供一个接口。

```go
//file: kv/kv.go

// Storage defines the interface for storage.
// Isolation should be at least SI(SNAPSHOT ISOLATION)
type Storage interface {
	// Begin transaction
	Begin() (Transaction, error)
	// BeginWithStartTS begins transaction with startTS.
	BeginWithStartTS(startTS uint64) (Transaction, error)
	// GetSnapshot gets a snapshot that is able to read any data which data is <= ver.
	// if ver is MaxVersion or > current max committed version, we will use current version for this snapshot.
	GetSnapshot(ver Version) (Snapshot, error)
	// GetClient gets a client instance.
	GetClient() Client
	// Close store
	Close() error
	// UUID return a unique ID which represents a Storage.
	UUID() string
	// CurrentVersion returns current max committed version.
	CurrentVersion() (Version, error)
	// GetOracle gets a timestamp oracle client.
	GetOracle() oracle.Oracle
	// SupportDeleteRange gets the storage support delete range or not.
	SupportDeleteRange() (supported bool)
	// Name gets the name of the storage engine
	Name() string
	// Describe returns of brief introduction of the storage
	Describe() string
	// ShowStatus returns the specified status of the storage
	ShowStatus(ctx context.Context, key string) (interface{}, error)
}
```

这里存储层的Storage接口包含Begin，并返回一个Transaction接口，重点关注tikv实现Storage接口,
并在其中记录日志

```go
//file tikv/kv.go

func (s *tikvStore) Begin() (kv.Transaction, error) {
	logutil.BgLogger().Info("hello transaction")
	txn, err := newTiKVTxn(s)
	if err != nil {
		return nil, errors.Trace(err)
	}
	return txn, nil
}
```

### 2.2 重新编译运行

```sh
cd tidb && make && ./bin/tidb-server
```

即可看到有事务执行时打印 "hello transaction"

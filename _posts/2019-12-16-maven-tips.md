---
layout: post
title: "maven tips"
description: "maven tips"
keywords: maven
category: Programming
tags: [maven]
---


文档参见：<http://maven.apache.org/ref/3.6.3/>

## settings.xml

> This is a reference for the user-specific configuration for Maven.
>
> Includes things that should not be distributed with the pom.xml file, such as developer identity, along with local settings, like proxy information.
>
> The default location for the settings file is ~/.m2/settings.xml

`$HOME/.m2/settings.xml`默认不存在，需要自己创建，可以先拷贝全局的`settings.xml`
参见`${M2_HOME}/conf/settings.xml`

测试macOS系统可以通过如下方式获得`${M2_HOME}`，位于`/usr/local/Cellar/maven/3.6.2/libexec`

```
mvn -v

Apache Maven 3.6.2 (40f52333136460af0dc0d7232c0dc0bcf0d9e117; 2019-08-27T23:06:16+08:00)
Maven home: /usr/local/Cellar/maven/3.6.2/libexec
Java version: 13, vendor: AdoptOpenJDK, runtime: /Library/Java/JavaVirtualMachines/adoptopenjdk-13.jdk/Contents/Home
Default locale: zh_CN_#Hans, platform encoding: UTF-8
OS name: "mac os x", version: "10.14.6", arch: "x86_64", family: "mac"
```

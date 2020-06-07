---
layout: post
title: "软件架构设计总结"
description: ""
keywords: software architecture, 软件架构
category: Programming
tags: [software architecture]
---

## 前言

总结软件架构的各种经验。

我目前最大的问题就是不太重视这些，觉得应该深入研究代码，实际上代码这一块只要不是特别的深的算法领域，基本没有能难住的，
而目前我还是太注重细节了。可能跟想了解Java常用套路相关吧。

另一个问题是不太注重业务，觉得没意思。绘图和业务在我潜意识里属于浪费时间。所以动力不大，实际上这个是必要的，所以我应该好好锤炼一下。

## 总体原则

软件架构是思考的艺术，对于解决同一个问题，可以从不同角度思考一个问题。
软件架构的一些经验或者说套路可以帮助我们规划系统


## 总体步骤

1. 需求分析
2. TODO


## 软件架构中的各类图

所谓一图胜千言，然后很难用一幅图表达一个架构的方方面面，

所以首先得明确每种图的设计目的，是给谁看的(也是给自己看的)，这样有助明确哪些信息需要在图中表达，哪些不需要

### 列举各类图

#### 应用场景图

下图选取自<https://help.aliyun.com/document_detail/29468.html>

这张图应该一方面可作为商业报告书用，一方面可明确产品未来方向。应该在产品开发早期阶段绘制。

![阿里云API网关应用场景](http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/29468/cn_zh/1484724886634/%E5%9C%BA%E6%99%AF1%E6%8B%A5%E6%8A%B1API%E7%BB%8F%E6%B5%8E-559-287%EF%BC%882x%EF%BC%89.jpg)

这些图很清晰地表面了API网关的应用场景，很不错。

#### 产品架构图

<https://help.aliyun.com/document_detail/30523.html>

这一张图

1. 反映了应用领域，

2. 设备-物联网平台-阿里云这样的交互关系

3. 划分了物联网系统的各个子模块。例如`固件升级`，`实时监控`，而且也交代了模块的交互关系，`IoT Hub`与外部设备交互。规则引擎执行内部阿里云产品能力。

我认为这个系统划分，图例交代非常经典。

![阿里云IoT产品架构](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/0291208751/p3364.png)

----------------------------------------------------------------------

<https://help.aliyun.com/document_detail/56515.html>

这张CodePipeline产品架构图第一眼没看明白，因为涉及到一些术语。

那么我先了解一下前文概述:

> 阿里云CodePipeline是一款提供*持续集成/持续交付能力*，并完全兼容Jenkins的能力和使用习惯的SAAS化产品

大致明白其用于持续集成、交付，类似于Jenkins

我觉得得到CodePipeline产品用途的情况下，还是没有看懂架构图。 是因为架构图中没有表达持续集成交付的步骤(或者说时序关系)，我便不知道该系统运行的起点在哪里，然而持续集成，持续交付肯定有一个分阶段的，首先是提交。

后一张图有交代，比较好。

#### 时序图

参考 <https://mermaid-js.github.io/mermaid/#/sequenceDiagram> 的不同种类的时序图

关于时序图，重点是，谁对谁发送什么消息

如下时序图

<https://help.aliyun.com/document_detail/127674.html?spm=5176.11065259.1996646101.searchclickresult.7afd72027BsmsR>

时序图反映的是谁跟谁发送了什么消息，谁对谁做出了相应这样一件事情。

感觉不应该太细节，现在的时序图太细节了，感觉变成了泳道图。但是我觉得不用刻意遵守时序图不应写细节这样的教条。而是根据实际情况
怎样表达清晰即可。

<https://docs.open.alipay.com/20180417160701241302/dvonpk/>

![支付宝预授权时序图](https://docs.open.alipay.com/20180417160701241302/dvonpk/)

这里的支付宝预授权时序图我觉得很棒。

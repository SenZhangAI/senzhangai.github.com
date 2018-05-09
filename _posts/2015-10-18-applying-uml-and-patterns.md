---
layout: post
title: "Applying UML and Patterns笔记"
description: "《Applying UML and Patterns》笔记"
keywords: UML,
category: 软件工程
tags: [UML, 设计模式, 敏捷开发]
---

## 前言
此文是关于如何实现面向对象设计开发（OOAD），从更抽象的软件构架的角度去思考。内容摘录自《Applying UML and Patterns》第三版

## Part1 Introduction
### Chapter1 OOAD(面向对象分析设计)
好的软件设计的目标：robustness, maintainability, and reusability

tip1："A critical ability in OO development is to skillfully **assign responsibilities** to software objects."
面向对象开发的一个关键技巧是对各个对象进行职责分配（assign responsibilities）

分析与设计的区别：do the right thing (analysis), and do the thing right (design).

#### 1.1 面向对象分析
finding and describing the objects——or concepts——in the problem domain. For example, in the case of the library information system, some of the concepts include Book, Library, and Patron.

在问题域（problem domain）中，发现并描述对象。

#### 1.2 面向对象设计
defining software objects and how they collaborate to fulfill the requirements.

定义软件对象，并设计它们如何协作实现需求。

#### 1.3 OOAD设计步骤
1. Define use cases
2. Define domain model
3. Define interaction diagrams
4. Define design class diagrams

##### Define use cases
简单来说就是脑补用例，假设这个软件做出来会有哪些应用场景，《Writing Effective Use Cases》这本书待看。例如做德州扑克，那么先实际玩一遍。

use cases的好处当然是快速认识软件的需求，可方便跟客户沟通。

##### Define domain model
定义域模型我的理解是注意几个关键词：concepts, attributes, and associations
concepts这里的意义没有理解，不就是类吗？或者说设计为类之前脑海中想到的那个东西。

我理解的思路是从不同对象的角度去思考，我有什么属性？跟其它对象之间有什么联系（association）？

例如德州扑克，我假设我是一张牌，那么我的属性是不同牌之间的区别，也就是suit花色和point点数。我跟其他的对象的关系呢？我属于牌桌52张牌中的一个，我可能是player的手牌中的一部分。

需要注意的是domain model并不等同于software objects，domain model是我们对现实世界对象的第一感觉上的认识，也称为conceptual object model（概念对象模型），（这么说software objects可能有部分反直觉的东西）

我感觉domain model是一个初步的设计。其后精细化的对象设计将会是针对software object，但domain model毕竟能提供要给草案

##### Assign Object Responsibilities and Draw Interaction Diagrams
OOD关注与software objects的各自的职责和它们之间的交互。

通常通过**sequence diagram（序列图）**的形式展示交互。展现软件对象之间的信息流和方法调用（invocation of methods）

也就是通过序列图并且了对象调用之间的关系。

##### Define Design Class Diagrams
这个将更加细化，明确每一个software的属性和方法以及之间的交互。
比较接近真实的代码结构了。

#### 1.4 UML
UML（The Unified Modeling Language ）可应用于画草图、作为蓝皮书或者通过完全的UML直接生成程序，但是通常完全的UNL没有必要，在敏捷开发或者逆向工程中，UML草图即可。

学会UML并不等于学会系统设计，UML只是一个表示工具，真正需要的是软件工程系统设计的思想。

#### 1.5 关于UML的参考资料
在《Applying UML and Patterns》第三版 第1.9章节中给出了关于uml的各种参考资料，这里不详细粘贴出来，需要的时候回头看。

### Chapter2 Iterative, Evolutionary, and Agile
[瀑布式开发-百度百科](http://baike.baidu.com/link?url=noszM0BLIdclufJkaYrAUGtZWsO82CI3mY1Zy_m0vaUZpDBNLpfmKovbAI9_urKc82p7C7qwLBHqPb5hanK0X_)是一种陈旧的开发方式，起源于传统行业的工作流程，流程划分过于清晰，难以应对需求变化（船大难掉头），研究表明失败率较高（约70%）

而迭代开发成功率较高，缺陷较少。

#### 2.1 什么是UP（unified process）？
**软件开发过程（software development process ）**主要包括三个步骤： 构建（building）, 部署（deploying）和维护（maintaining）。

UP是一个**面向对象的**、**迭代地**软件开发过程。

Rational Unified Process（RUP）则是取自UP的精华，应用广泛。

UP也汲取其他开发模式的精华，例如XP、Scrum等等。

例如：XP的**TDD**、**refactoring**以及**continuous integration**，
Scrum的common project room ("war room")以及daily Scrum meeting practice

#### 2.2 什么是Iterative and Evolutionary Development?
##### 什么是迭代开发？
1. In this lifecycle approach, development is organized into a series of short, fixed-length (for example, three-week) mini-projects called **iterations**;
2. the outcome of each is a **tested**, **integrated**, and **executable** partial system.
3. Each iteration includes **its own requirements analysis**, **design**, **implementation**, and **testing activities**.

然后将各个iteration子系统不断升级、细化。

注意其中的一个关键点是迭代的时间是**限定了的**，例如三周。这是一个非常重要的点，因为我们的脑容量的思考深度有限，很难遇见未来会遇到的问题，所以不如先尝试再发现问题再进一步开发。通过时间限定也避免了一开始就大迈步地去做深度很深的实现而陷进去。

早期的迭代开发也成为螺旋式开发（spiral development）、演进式开发（evolutionary development）。

##### 示例
一个典型的迭代开发包括：
启动会上确认goals和tasks，而工程师则通过UML演示软件执行过程或其他有价值的草图。余下的时间这是**结对开发**，在白板上设计草图、讨论实现等等。

中后期则是：coding（确切的说实现implement）、测试、further design, integration以及builds of the partial system。

其他的活动也包括**给持股人（stakeholders）演示和评估**以及**计划下一次迭代**。

需要注意的是迭代开发并没有一个极端地rush code或者另一个极端地详细设计每个细节。

##### How to Handle Change on an Iterative Project?
这应该是迭代开发最核心的内容之一了，

首先第一点要拥抱变化（Embrace Change），意识到改变是不可避免的，所以像瀑布式开发这样一开始就尝试明确所有步骤是不可能的。

each iteration involves choosing a small subset of the requirements, and quickly designing, implementing, and testing.

每次的需求实现并不一定是最终需求，但是经过原型开发，能够获得很好地反馈，以指导进一步开发战略。

迭代开发的优点就不阐述了，就想越早test效率越高一样。

##### How Long Should an Iteration Be?
通常2周到6周为一个时间节点，如果1周太短了，估计实现很困难。

注意迭代开发非常注重固定的时间节点完成，不允许拖延，如果实在完成不了，可以将部分feature移到下一次迭代开发，但不能拖延。

我猜测时间点的长度应该根据不同团队的实际情况选择，可以前期调整磨合出长度比较合适的时间窗口。

#### 2.3 瀑布式开发
即开始就做好全部或者大部分规划。
这个不用多讲，是效率比较低、成功率低的软件开发方式，有研究表明最后45%的特点都是白写了的。

至于为什么瀑布式开发表现比较差，我个人认为是因为人对于未知的预测往往存在偏差，比如需求的预测，比如软件构架设计的预测，等等。
所以不如先开发一个原型，然后看效果，分析不足并改进，所谓投石问路。

#### 2.4 How to do Iterative and Evolutionary Analysis and Design?
首先需要说明迭代分析并不意味着不思考不分析的情况下蛮干，
当然也不是走瀑布式开发那样“深思熟虑”地固步自封。
它是一个适度的过程，确定好一些大的框架后再执行。

以下举例说明：（假设一共需要20个迭代步骤）

1. Before iter-1，召开**需求研讨会**（注意先确定时间节点，例如两天），业务与开发（包括技术总监）必须到场。
    * 1st day morning，**high-level需求分析**，确定一些初略的use cases，以及关键性的non-functional需求。
    * CTO以及业务将从这些high-level list**选取10%top需求**（例如30个use cases中选3个），包括**1.core architect**：后期基于此架构迭代设计、构建、测试。**2. high business value**：以商业为导向，或者说核心目标要明确，别走歪了。**3. high rest**：风险控制。
    * 1.5 day remain，对这3个cases集中分析其functional和non-function需求，10%深入分析，剩下90%只是high-level。
2. Before iter-1，针对这三个use case，UC1、UC2、UC3，**iteration planning meeting**，**规划iter-1**接下来的时间节点（例如4周）的设计、构建、测试安排。注意并非所有功能都能在iter-1中实现。选择子目标，并将其分解成更细更小的子迭代任务。
3. iter-1，通常3到4周。
    * 1st 2 day，developers**结对**modeling和design。在一个公共的war room中画类UML图（需要很多白板），CTO指导。
    * Then，developers从modeling阶段转换到programming阶段，编码、测试、集成
    * test包括：unit、acceptance、load、usability等。
    * Monday of last week，确认初始的迭代目标是否可达成。如果不行，缩减目标，将余下目标放到todo list中。
    * Tuesday of last week，停止进一步编码（**code freezed**），**将所有代码**checked in、集成、测试以生成the iteration baseline。
    * Wednesday morning，将开发的原型demo演示给external stakeholders（例如天使投资），展示愿景，并**一定要求获得反馈**。
4. near end of iter-1，**iter-2 需求研讨会**。准备第一次研讨会的材料，并根据iter-1的情况，提炼(refine)第一次的材料。接下来挑选10-15%的重要的use cases，用1-2天时间分析use cases的细节，这样，大约25%的use cases和non-functional需求就已经有了细节部分，但并非最终完美的解决方案。
5. last Friday morning，计划下一次迭代开发。
6. Do iter-2；步骤类似。
7. 大约4次iter + 5次需求研讨会后，大约80-90%的需求细节已经设计完成，**但仅占整个system完成度的10%**。
    * 迭代的优势在于反馈(feedback)与演进(evolution)，因此较瀑布开发有更高质量。
8. 至此，大约占整个项目完成度的20%，在UP中，这是细化阶段(elaboration phase)的终点。此时应该详细估算整个项目所需时间并精炼高质量的需求。在前期多次realistic investigation、feedback、test的基础上做这些（即估算时间和分析需求）具有较高的可信度。
9. 至此，需求虽未固定但基本稳定，所以之后的需求讨论将不太一样。记得每次在iter接近尾声的那个周五问自己，"根据目前所知，最重要的技术和商业价值是什么？下一步我们该怎么做？"



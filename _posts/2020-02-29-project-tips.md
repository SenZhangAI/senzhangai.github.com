---
layout: post
title: "项目设计的相关经验"
description: "项目设计"
keywords: project tips
category: Programming
tags: [tips]
---

## 前言

如下是项目中踩过的坑或总结的经验

## 不同环境不可共用数据库

比如test环境redis不能给dev环境共用

原因是比方说test跟dev数据库同一条数据id不一样(自增id)，那么如果同时保存在同一个redis中，可能出很隐蔽的bug

---
layout: post
title: "Centos 7 安装 Gitlab"
description: "Centos 7 install gitlab"
keywords: Centos, gitlab
category: Tools
tags: [gitlab]
---

# 步骤

## 1. 更新为国内源
因直接从国外源下载网速有些慢，可以用其他途径下载rpm包后手动安装，或者更新为国内源安装。

更新源步骤参见：<https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/>

改国内源的方法不一定凑效，或许下载后手动安装更方便。

## 2. 安装gitlab

安装步骤参见： <https://about.gitlab.com/installation/#centos-7>

安装过程中`sudo systemctl start postfix`可能会执行失败，

错误原因可能是：

```
postfix报错postfix: fatal: parameter inet_interfaces: no local interface found for ::1
```

解决办法参见： <http://www.chhua.com/web-note4929>

在执行`sudo gitlab-ctl reconfigure` 前应该需要修改 `/etc/gitlab/gitlab.rb`，
将external_url改为自己的服务器的url，例如：

``` ruby
## GitLab URL
## ! URL on which GitLab will be reachable.
## ! For more details on configuring external_url see:
## ! https://docs.gitlab.com/omnibus/settings/configuration.html#configuring-the-external-url-for-gitlab

## ip 或者 Url都可以，前缀http，如果开了443 https端口应该也可以用https
#external_url 'http://119.23.28.251'
external_url 'http://www.mysite.com'
```

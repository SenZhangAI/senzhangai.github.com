---
layout: post
title: "Windows系统上利用github+Jykell搭建个人博客"
description: "Windows系统上，利用github主页以及jekyll搭建个人博客"
keywords: "github, jykell, redcarpet, cygwin, blog"
category: "Programming"
tags: [github, jykell, redcarpet, cygwin, markdown]
---

## I 前言
很多人都说过分享知识的同时也有助于凝练提高自己的技术水平，所以我也尝试建立个人博客。

故尝试在[github](https://github.com/)上建站同步更新，互为补充。同时也是给自己扩充一下网页相关知识。

折腾了十多天的时间，想着各种可以改进的思路。结果最后却发现有一个博客的[Theme](https://github.com/javachen/javachen-blog-theme)完美解决了我的问题。而且只需要改几个设置即可。心情很复杂。

关于jekyll+github搭建博客的技术贴很多。我就仅讲下windows系统下搭建历程，及一些周边的内容。

## II 名词解释

### 1.[git](http://www.git-scm.com/)
git是软件开发的分布式版本控制系统。具有多人协作，多台设备代码共享，历史回退。分支管理等功能，可用于辅助软件项目管理。相当于软件界的dropbox吧 ：)，只不过学习曲线有点陡。刚开始有些不习惯。

* git好的在线学习网站：[http://pcottle.github.io/learnGitBranching/](http://pcottle.github.io/learnGitBranching/)
* windows下好用的git工具：[sourcetree](http://www.sourcetreeapp.com/)。
    * 配合[git for windows](http://msysgit.github.io/)以及[BeyondCompare](http://www.scootersoftware.com/)相当给力。
    * 之所以不推荐Cygwin下的git是因为在windows下存在中文乱码的问题，很麻烦。
    * 单用git for windows也会存在乱码的问题，不知为何。且易用性不好。
    * 不足之处是可能会比较卡。

### 2. [github](https://github.com/)

本为[git](http://git-scm.com/)代码管理网站，而非真正意义上的个人网站。但其提供静态网页托管服务。所以算是非常规用法 :)

### 3. [jekyll](http://jekyllcn.com/)

>jekyll是一个自定义blog网站模板，通过[markdown](http://wowubuntu.com/markdown/)与[Liquid](https://docs.shopify.com/themes/liquid-documentation/basics) 转换文本，通过使用 [GitHub Pages](https://pages.github.com/)作为引擎服务，免费的来发布个人的blog网站。

选用jekyll的原因[参考下文](http://www.pkuwwt.tk/linux/2013-11-29-build-a-github-website/)。

>jekyll的优点在于，它和Github是有关系的，Github的Page功能就是它提供的。因此，你根本不需要将生成的静态传到Github上去。

### 4. [markdown](http://wowubuntu.com/markdown/)

该格式非常方便书写，并可以通过在线编辑器(例如[MaHua](http://mahua.jser.me/))或者离线编辑器(例如[HarooPad](http://pad.haroopress.com/user.html#download))支持该格式的网站框架(例如github,CSDN)转化为html格式或者pdf格式。）


###4. [Cygwin](https://cygwin.com/)
Cygwin可理解为windows系统上的Linux系统的虚拟机，或者说把Linux系统上的一些重要软件移植到Windows系统下。总体而言，在windows下装jekyll实在太麻烦了，所以采用此方法。


##III 搭建过程
###0. 参考链接
- [Github Page说明文档](https://pages.github.com/)
-

###1. 在[github](https://github.com/)上创建主页仓库
主要步骤是：

1. [创建github账户](https://github.com/join)，例如账户名为YourName
2. 创建名为`YourName.github.com`或者`your-user-name.github.io`的仓库（repository），[网页上创建仓库步骤](https://help.github.com/articles/create-a-repo/)

[主要参考下文](http://www.pkuwwt.tk/linux/2013-11-29-build-a-github-website/)：

>注册一个Github帐号pkuwwt，因此，我的github帐号是
[github.com/pkuwwt](https://github.com/pkuwwt)。

>然后，我可以建立任意的github仓库(repository)。使用过git的人应该不会陌生。

>Github提供了网页托管服务...[[more]](http://www.pkuwwt.tk/linux/2013-11-29-build-a-github-website/)

###2. 安装[jekyll](http://jekyllcn.com/)
####Linux,Unix,Max OS X系统下的安装步骤
对于Linux,Unix,Max OS X用户而言，安装十分简单，参考[http://jekyllcn.com/docs/installation/](http://jekyllcn.com/docs/installation/)

####Windows系统下的安装步骤
#####方案1. 便携版（直接忽略）

`注：本文只测试了安装版本的jekyll安装，便携版（绿色版）的jekyll未经测试。`

**[便携版下载地址](https://github.com/madhur/PortableJekyll)**
下载页面download ZIP

**[便携版安装方法](https://github.com/madhur/PortableJekyll/wiki)**：解压并直接用管理员权限运行`setpath.cmd`即可

缺点是太大了，801M，还不如用Cygwin了，遂放弃。

#####方案2. 安装版(忽略)
参考链接：[http://jekyll-windows.juthilo.com/](http://jekyll-windows.juthilo.com/)

`注：安装过于繁杂在此略过，链接中步骤很清楚。就不补充了。已测试并成功安装`

#####方案3. 在Cygwin中安装（推荐）
还是用[Cygwin](https://cygwin.com/)方便啊！

`注：建议用x86版本的Cygwin，X64版本的可能会出现问题`

参考：[http://www.cnblogs.com/dreamFromHere/archive/2014/01/08/3506366.html](http://www.cnblogs.com/dreamFromHere/archive/2014/01/08/3506366.html)

需要注意的是相关软件如ruby，make，gcc要装齐全。同时jekyll的路径最好加入到环境变量$PATH中。

## IV 页面风格

### 1. 页面风格参考

基于jekyll的网站和源码：[https://github.com/jekyll/jekyll/wiki/Sites](https://github.com/jekyll/jekyll/wiki/Sites)


差不多十天的时间里面，参考了各种各样的模板，并且花了不少心思改进。结果最后发现一个模板满足了自己所有想要的需求。而且只需要改一下`_config.yml`即可。不需要太多心思设置。感觉自己白忙活了一阵子TT。这里对作者表示感谢！

有以下几个优点值得推荐：

* 利用`redcarpet`渲染引擎对markdown格式支持比较好，支持大部分高级的markdown语法规则。
* 页面整洁，非常好看，边栏内容丰富。
* 对手机登陆界面支持较好，加载速度很快。
* 有站内search功能。
* 仅需该几个设置即可。

主题可以从 <https://github.com/javachen/javachen-blog-theme> 下载，感谢原作者！[http:/blog.javachen.com](http://blog.javachen.com)。

博主关于搭建博客的文章也很好，值得参考！

<http://blog.javachen.com/2013/08/31/my-jekyll-config/>

<http://blog.javachen.com/2014/01/21/all-things-about-jekyll/>

## V 补充

### data url

网页中如果加载小图标，可以将图标制作成data url。这样就能直接在网页中嵌入自己的个性图标（最好是小图片），例如将favicon.ico制作成data-url嵌入

在线制作data url：[http://dataurl.net/#dataurlmaker](http://dataurl.net/#dataurlmaker)

### markdown引擎的选择

这个折腾了很长时间，一言难尽。


## VI TODO

加入对Latex语法的支持。

修改部分页面风格。

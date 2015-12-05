---
layout: post
title: "Cygwin 配置小结"
description: "我的cygwin安装配置"
keywords: cygwin, jekyll, vim, oh-my-zsh
category: tools
tags: [cygwin]
---

## 前言
此文关于我的cygwin安装配置，起因是重新安装新系统和cygwin后发现以前安装的软件又得重新安装，而且还不是那么容易安装，
故此文总结安装过程，以免每次安装又得重新查资料。

## 基础功能安装
### 1. 下载[cygwin]及初步配置

下载链接： [setup-x86](http://www.cygiwn.com/setup-x86.exe)

初步配置：

以最少量安装为原则，除了**base packages**中的软件包外，有以下软件包值得安装:

``` bash
# Base
## devel-package
gcc-core
gcc-g++
make
cmake
git
git-completion
bash-completion # 这两个自动补全还是需要滴

## web and net
curl
ssh
wget

## editor
vim

## utils
tree

# Option
## shell zsh (if use oh-my-zsh)
zsh

## vim-silver-searcher (if use vim-ag)
automake # 只安装一个版本的automake，否则所有版本都默认安装了。。
pkg-config
libpcre-devel
liblzma-dev

## astyle for code Formatter
astyle # 我通常会在vim中用，并设置快捷键

## ctags
ctags # 代码查看

## cscope
cscope # 同ctags，不过更强，我的vim中包含了cscope的配置。

## lua for vim-neocomplete
如果需要用到vim-neocomplete，则需要对lua提供支持

## jekyll (if test jekyll blog on cygwin)
ruby
ruby-gems
```

### 2. 配置ssh-keys for github
因经常用github，所以先配置好ssh-key，步骤参见[github-Generating-SSH-keys](help.github.com/articles/generating-ssh-keys)

简单来说就是：

```bash
$ ssh-keygen -t rsa -b 4096 -C "szhang.hust@gmail.com"
$ eval $(ssh-agent -s)
$ ssh-add ~/.ssh/id_rsa
$ clip < ~/.ssh/id_rsa.pub
```

然后在github网站中点击 **settings** -> **SSH keys** -> **Add SSH Key**
然后将clip中的内容粘贴（<kbd>ctrl</kbd> + <kbd>V</kbd>）上去即可。

### 3. 配置git global

```bash
$ git config --global user.email szhang.hust@gmail.com
$ git config --global user.email "Sen Zhang"
```

### 4. 配置 vim
直接下载安装我的vimrc配置即可

```bash
$ git clone git@github.com:SenZhangAI/vimrc .vim
$ cd .vim/
$ ./install.sh
```

### 5. 配置bash
主要就是添加部分注释掉的alias,以及添加`alias vi='vim'`

### 6. 环境变量配置
将Cygwin中bin文件夹，例如：`D:\Cygwin\bin`加入windows系统的**PATH**环境变量中去。

### 6. 各种ln
通过软链接定义各种快捷方式，例如将windows的Desktop、Documents、Downloads等链接过来。
也可以建立软件的快捷方式，这里假设在Cygwin中使用Windows下的Sublime Test

```bash
$ ln -s /cygdrive/c/Users/Sen/Desktop ~/Desktop
$ ln -s /cygdrive/c/Users/Sen/Documents ~/Documents
$ ln -s /cygdrive/c/Users/Sen/Downloads ~/Downloads
$ ln -s /cygdrive/c/YourPathToSoftware/Sublime_Text.exe /usr/bin/sublime_text
```

## 进阶功能可选安装
### 1. oh-my-zsh
如果习惯了bash，当然不需要zsh，但是用过oh-my-zsh之后，基本对bash无爱了。
所以虽说可选安装，但还是强烈建议安装。

#### 下载并自动安装

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

#### 设置默认shell为zsh
对于cygwin比较麻烦的是没有chsh命令，对此，需要修改`/etc/passwd`，
然而新安装的cygwin极有可能并没有`/etc/passwd`。
这时候，生成一个即可：

```bash
mkpasswd -l > /etc/passwd
```

然后修改该文件，将其中用户默认的bash改为zsh即可。

#### customization
下载我的oh-my-zsh配置，并安装即可：

```bash
$ git clone git@github.com:SenZhangAI/oh-my-zsh-sen ~/tmp/oh-my-zsh-sen
$ cd ~/tmp/oh-my-zsh-sen
$ bash ~/tmp/oh-my-zsh-sen/install.sh
$ source ~/.zshrc
```
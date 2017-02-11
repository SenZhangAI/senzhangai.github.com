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
gdb
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
zsh # 优点是兼容bash，功能更强，配合oh-my-zsh暴爽

## clang
clang # 推荐安装，类似于gcc的c-family编译器，更好
libclang # 如果用vim的补全插件 clang_complete
libclang-devel

## vim-silver-searcher (if use vim-ag)
### 以下插件都是为了在cygwin上安装silver-searcher(ag)需要的
automake
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
lua # 如果需要用到vim-neocomplete，则需要对lua提供支持

## jekyll (if test jekyll blog on cygwin)
ruby
ruby-devel
rubygems
libffi
libffi-devel
```

### 2. 配置ssh-keys for github
因经常用github，所以先配置好ssh-key，步骤参见[github-Generating-SSH-keys](https://help.github.com/articles/connecting-to-github-with-ssh/)

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
$ git config --global user.name "Sen Zhang"
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
实际上如果用zsh，这部分也可以不配置，但配置了更好。
最终`.bashrc`文件中的配置如下：

```bash
# Default to human readable figures
alias df='df -h'
alias du='du -h'
#
# Misc :)
alias less='less -r'                          # raw control characters
# alias whence='type -a'                        # where, of a sort
alias grep='grep --color'                     # show differences in colour
alias egrep='egrep --color=auto'              # show differences in colour
alias fgrep='fgrep --color=auto'              # show differences in colour
#
# Some shortcuts for different directory listings
alias ls='ls -hF --color=tty'                 # classify files in colour
alias dir='ls --color=auto --format=vertical'
alias vdir='ls --color=auto --format=long'
alias ll='ls -al'                              # long list
alias la='ls -A'                              # all but . and ..
alias l='ls -CF'                              #
alias vi='vim'
```

### 6. 环境变量配置
将Cygwin中bin文件夹，例如：`D:\Cygwin\bin`加入windows系统的**PATH**环境变量中去。

### 6. 各种ln
通过软链接定义各种快捷方式，例如将windows的Desktop、Documents、Downloads等链接过来。
也可以建立软件的快捷方式，这里假设在Cygwin中使用Windows下的Sublime Text

```bash
$ ln -s /cygdrive/c/Users/Sen/Desktop ~/Desktop
$ ln -s /cygdrive/c/Users/Sen/Documents ~/Documents
$ ln -s /cygdrive/c/Users/Sen/Downloads ~/Downloads
$ ln -s /cygdrive/c/YourPathToSoftware/Sublime_Text.exe /usr/bin/sublime_text
```

## 进阶功能可选安装
### 1. [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)
如果习惯了bash，当然不需要zsh，但是用过oh-my-zsh之后，基本对bash无爱了。
所以虽说可选安装，但还是强烈建议安装。

zsh的优点是几乎完全兼容bash的命令，且定制化极强，其方便程度，比如命令或者路径的智能提示，不是bash能比的。

然而它的缺点一方面是linux系统并不会默认安装，另一个方面是配置很麻烦，于是就有人写了非常强大的zsh的配置文件`oh-my-zsh`
所以安装zsh同时再安装oh-my-zsh配置文件即可。

#### 下载并自动安装oh-my-zsh

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
$ ./install.sh
# modify the plugins config in ~/.zshrc
$ source ~/.zshrc
```

需要注意的是，添加对其他插件的支持不能在custom中直接设置，这是因为`oh-my-zsh.sh`
中首先`source all plugins` 然后才`source all custom config`。
因此只能在`.zshrc`中修改，例如：

```bash
# plugins=(git) # the original config
plugins=(git z vi-mode zsh-syntax-highlighting)
```

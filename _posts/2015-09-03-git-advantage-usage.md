---
layout: post
title: "git 高级用法remote add"
description: "git的用法实践"
keywords: git
category: tools
tags: [git]
---

## 缘由
看到github上的一个vim的代码自动生成片段[snippets]，很想加入到我的vim配置中，只是考虑到有以下需求：

1. 我得能够自己定义代码片段
2. 如果原作者更新了新的片段，我希望能够加入到我的代码中

这样的话，直接clone作者的仓库或者自己fork仓库都不行，而是用`git remote add 远程仓库`的方式。

## 步骤
以vim-snippets仓库为例
#### step 1: fork远程仓库
从<https://github.com/honza/vim-snippets>中fork远程仓库到自己的github
#### step 2: 将fork来的仓库clone到本地
由于我的父文件夹`.vim/`文件中已经是一个仓库，所以需要用`git submoudle`增加一个子仓库：

```bash
~/.vim/bundle $ git submoudle add https://github.com/YourGithubName/vim-snippets
```

注意`submoudle`的删除比较麻烦，参见<http://stackoverflow.com/questions/1260748/how-do-i-remove-a-git-submodule>
有时候用`subtree`更好，`subtree`将子项目和主项目融合在一起，而submoudle可以理解为完全分开。因为snippets仓库可能经常会更新的缘故，担心会影响到主仓库vimrc的整洁，所以没用subtree，用`subtree`和`submoudle`视情况而定。

#### step 3: git remote set-url
这一步骤是为了避免每次`git push`都要输入帐户名以及密码的麻烦：

```
~/.vim/bundle $ git remote set-url origin git@github.com:YourGithubName/vim-snippets.git
```

或者直接在步骤2中采用：

```bash
~/.vim/bundle $ git submoudle add git@github.com:YourGithubName/vim-snippets.git
```

#### step 4：将原仓库加入为远程仓库

```bash
~/.vim/bundle $ git remote add upstream https://github.com/honza/vim-snippets
```

其中`upstream`只是一个标签，可以取任意其他名字。

这样当执行`git remote -v`时将看到有两个远程仓库，如下：

```
origin  git@github.com:SenZhangAI/vim-snippets (fetch)
origin  git@github.com:SenZhangAI/vim-snippets (push)
upstream        https:/github.com/honza/vim-snippets (fetch)
upstream        https:/github.com/honza/vim-snippets (push)
```

## 同步原始仓库
参见： <http://git-scm.com/docs/git-remote>

如果要同步原始的仓库，可以用以下命令：

```
~/.vim/bundle $ git fetch upstream
```

当然，由于也改变代码，这样和可能会出现冲突，需要在rebase的时候合并。

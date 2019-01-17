---
title:      git基础
author:     liuyan
catalog:    true
tags:
  - git
  - linux
  - 工具
date:       2017-10-03 20:18:44
urlname:
categories: linux
---

###  综述
下面这张图是git工作区域的一个简单介绍

![](git.jgp)

几个专用名词的译名如下。

    Workspace：工作区
    Index / Stage：暂存区
    Repository：仓库区（或本地仓库）
    Remote：远程仓库

<!-- more -->

### ~/.gitconfig文件

```shell
[user]
    email = liuyan30@meituan.com
    name = Liu Yan
[alias]
  br = branch
  ci = commit
  co = checkout
  st = status
  a = add
  d = diff
```

### git常用命令

```shell
git log --pretty=oneline                #查看版本号
git remote add origin git@***.git       #让本地仓库和远程仓库进行关联
git branch                              #查看本地分支
git branch -a                           #查看远程分支
git diff (--stat)                       #查看修改的部分
git chockout -b liuyan                  #新建本地分支liuyan并进入新分支
git add .                               #添加当前目录的所有文件到暂存区
git commit -a -m "modify mimir options" #commit代码到新分支
git fetch                               #fetch所有分支
git checkout -b xxx origin/xxx          #将远程分支拉到本地
git pull origin xxx                     #pull xxx分支
git checkout ann.cc                     #把ann.cc回退到上一次提交的版本
git checkout xxx                        #切换到xxx分支
git merge xxx                           #将xxx分支merge到当前分支
git branch -d xxx                       #删除本地xxx分支
git status                              #显示变更的文件
git push origin xxx                     #push代码到远程分支xxx，要是不存在会新建分支xxx
git push origin --delete xxx            #删除远程分支xxx
git branch -m xxx1 xxx2                 #重命名本地分支
```

### git常用命令速查表

![](git常用命令速查表.jpg)

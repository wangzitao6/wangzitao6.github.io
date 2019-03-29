---
title:     "git常用命令"  # 文章标题
subtitle:    "git"  # 文章标题
description: "Git的一些命令总结"
date:         2016-11-03T23:10:30+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["git"]
url:    "/2017-11-03-git常用命令/"
categories:  ["build"]
---
## 1. 分支操作

**1.创建，删除本地分支**
git branch 分支名  创建名称为branchname的本地分支

  **`git branch branchName`**

git branch -d 分支名  删除名称为branchname的本地分支

  **`git branch -d branchname`**


**2.切换本地分支**
git checkout 分支名，例如从master切换到dev分支：

  **`git checkout dev`**

**3.远程分支就是本地分支push到服务器上。比如master就是一个最典型的远程分支（默认）**。

 **`git push origin dev`**

**4.远程分支和本地分支需要区分好，所以，在从服务器上拉取特定分支的时候，需要指定远程分支的名字。**

 **`git checkout --track origin/dev`**

注意该命令由于带有--track参数，所以要求git1.6.4以上！这样git会自动切换到分支。

**5.提交分支数据到远程服务器**
git push origin <local_branch_name>:<remote_branch_name>
例如：
**`git push origin dev:dev`**

一般当前如果不在该分支时，使用这种方式提交。如果当前在 dev 分支下，也可以直接提交

**`git push`**


**6.删除远程分支**

  **`git push origin :develop`**  

**7.拉取项目分支**

 **`git clone --branch branchname  repoPath`**

**8.查看所有的分支**
查看所有的分支  高亮的是当前分支:
**`git branch`**


## 2. 文件操作
  ```git
  git status   查看这次修改的东西

  git diff filepath  查看此次修改差异

  git add filepath 把某一个文件加入暂存区

  git  add .   把所有的修改文件放入暂存区

  git commit -m " 修改的信息"   提交修改并显示修改的信息

  git pull  origin/master 拉去远程的master分支（git pull  origin master ） 放在add commit 之后

  git checkout filepath   回滚某一个文件的修改

  git checkout .  回滚所有的修改

  git reset  撤销所有暂存区文件

  git reset filepath  撤销暂存区中的某一个文件

  git push  origin/master   推送到远程的master分支  （git push  origin master ）

  git log 查看项目的commit版本

  git log -p  filepath 查看某一个文件的所有commit版本

  git show commit版本号  显示此次commit所修改的内容
  ```

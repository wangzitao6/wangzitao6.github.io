+++
title = "Git常用命令"  # 文章标题
date = 2019-02-28T23:53:19+08:00  # 自动添加日期信息
draft = false  # 设为false可被编译为HTML，true供本地修改
tags = ["linux,git"]  # 文章标签，可设置多个，用逗号隔开。Hugo会自动生成标签的子URL
comments = true  # 是否开启Disqus评论功能
share = true  # 是否开启分享
+++

克隆bscode分支
git clone --branch bscode git@ingsys.cn:hscode  

git clone --branch sfcode git@ingsys.cn:ruifeng

git clone --branch hxcode git@ingsys.cn:hongxing

git clone git@ingsys.cn:hongxing

git clone  git@github.com:mform/myform.git  克隆仓库

git status   查看这次修改的东西

git  add .   把所有的修改文件放入暂存区

git add filepath 把某一个文件加入暂存区

git reset  撤销所有暂存区文件

git reset filepath  撤销暂存区中的某一个文件

git checkout .  回滚所有的修改

git diff filepath  查看此次修改差异

git checkout filepath   回滚某一个文件的修改

git commit -m " 修改的信息"   提交修改并显示修改的信息

git push  origin/master   推送到远程的master分支  （git push  origin master ）

git pull  origin/master 拉去远程的master分支（git pull  origin master ） 放在add commit 之后

git log 查看项目的commit版本

git log -p  filepath 查看某一个文件的所有commit版本


git show commit版本号  显示此次commit所修改的内容

git  branch 查看所有的分支  高亮的是当前分支

git branch <branchname>，创建新的分支branchname

git branch -d <branchname> ，删除名称为branchname的分支


git checkout -b test  创建并切换到本地test分支

git checkout -b test  创建并切换到 本地test分支

git push origin test  把创建的分支推送到远端


git checkout master   切换到master分支





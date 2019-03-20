---
title:     "利用Travis CI将Hugo自動部署到Github Pages"  # 文章标题
subtitle:    "利用Travis CI将Hugo自動部署到Github Pages"  # 文章标题
description: ""
date:         2018-03-10T00:16:42+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["blog","其他"]
url:    "/2018-03-10-自动部署/"
categories:  ["other"]
showtoc: true
---
## 1.简介
Hugo 有着 Go 语言的几大优点：跨平台、执行快、部署简单、无需安装依赖，同时作为一款静态博客系统，它主题多、拓展性好、操作简单。是用来搭建个人博客的绝佳选择。但是用hugo一段时间后,发现新增博客时部署比较麻烦,经过几天的折腾后发现了Travis CI。

## 2.Travis CI是什么
一个持续化集成平台，类似Jenkins。功能强大，和GitHub的集成尤其好，我们用它部署个人博客算大材小用。它有两个版本:

- [https://travis-ci.org/](https://travis-ci.org/) 免费版本，可以集成GitHub的public项目

- [https://travis-ci.com/](https://travis-ci.com/) 商业版本，可以集成GitHub的private项目

我们使用第一个，免费版本。

## 3.配置Travis CI
### 3.1 为Travis CI生成GitHub Token

打开GitHub。路径: “Settings”->“Developer settings”->“Personal access tokens”->“Generate new token”。[直达车](https://github.com/settings/tokens/new)

因为是public项目，而且Travis CI是用来push代码，所以只需勾选 public_repo, repo:status, repo_deployment 三项。

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/04/007.png)

</br>
然后点击Generate Token 生成token.

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/04/008.png)

Token一会儿就会隐藏，不能找回，所以拷贝好，进入下一步。

### 3.2 配置Travis CI构建选项
> 1.用GitHub方式登录[Travis CI](https://travis-ci.org/)

> 2.“Settings”-“General” 勾选”Build only if .travis.yml is present”和”Build pushed branches”两项。

> 3.“Settings”-“Environment Variables” 添加”GITHUBTOKEN“，值是上一步得到的Token

### 3.3 添加配置文件

**在git根目录下添加 .travis.yml 文件**

>注意我的项目背景,项目有两个git分支
>- hugo: 存放博客源码
>- master: 存放Hugo生成的静态站点文件

  ```yml
  language: go
  # Specify which branches to build using a safelist
  # 分支白名单限制: 只有hugo分支的提交才会触发构建
  branches:
    only:
      - hugo
  git:
    depth: 1
  install:
  # 安装最新的hugo
    - go get github.com/spf13/hugo
  script:
    # 运行hugo命令
    - hugo
  deploy:
    provider: pages  # 重要，指定这是一份github pages的部署配置
    skip-cleanup: true  # 重要，不能省略
    # token is set in travis-ci.org dashboard
    github_token: $GITHUB_API_KEY  # 重要，$GITHUB_API_KEY 是变量，需要在GitHub上申请、再到配置到Travis

    local-dir: public # 静态站点文件所在目录
    repo: wangzitao6/wangzitao6.github.io # github用户名/博客仓库名称
    target_branch: master # 要将静态站点文件发布到哪个分支
    keep-history: true # 是否保持target-branch分支的提交记录
    on:
      branch: hugo # 博客源码的分支

  ```

  把 .travis.yml 放到hugo分支，push到GitHub。
  
## 4.自动部署

上述操作完成后，自动部署就生效了。我们写完一篇博客，只需提交push到GitHub的hugo分支，Travis CI会自动触发后续的构建、在master分支生成静态文件,然后部署

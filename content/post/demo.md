+++
title = "Demo"  # 文章标题
date = 2019-03-01T13:05:37+08:00  # 自动添加日期信息
draft = false  # 设为false可被编译为HTML，true供本地修改
tags = [""]  # 文章标签，可设置多个，用逗号隔开。Hugo会自动生成标签的子URL
share = true  # 是否开启分享
+++

测试 Travis CI

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
  repo: wangzitao6/wangzitao6.github.io
  target_branch: master # 要将静态站点文件发布到哪个分支
  keep-history: true # 是否保持target-branch分支的提交记录
  on:
    branch: hugo # 博客源码的分支

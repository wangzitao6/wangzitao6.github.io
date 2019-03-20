---
title:     "Hugo集成gitment评论系统"  # 文章标题
subtitle:    "Hugo集成gitment评论系统"  # 文章标题
description: ""
date:         2018-03-20T11:06:11+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["blog"]
url:    "/2018-03-20-集成gitment评论系统"
categories:  ["other"]
---
## 引言
Gitment 是作者实现的一款基于 GitHub Issues 的评论系统。支持在前端直接引入，不需要任何后端代码。可以在页面进行登录、查看、评论、点赞等操作，同时有完整的 Markdown / GFM 和代码高亮支持。尤为适合各种基于 GitHub Pages 的静态博客或项目页面。

本博客评论系统已迁移至 Gitment。虽然 Gitment 只能使用 GitHub 账号进行评论，但考虑到博客受众，这是可以接受的。

[项目地址](https://github.com/imsun/gitment)
[示例页面](https://imsun.github.io/gitment/)

## 1. 注册 OAuth Application
点击此处 来注册一个新的 OAuth Application,[申请地址](https://github.com/settings/developers)

其他内容可以随意填写，但要确保填入正确的Authorization callback URL（一般是评论页面对应的域名，
如 `https://wangzitao6.github.io`）注意域名后不要带斜杠

Homepage URL也可以填写域名`https://wangzitao6.github.io`。

你会得到一个 client ID 和一个 client secret，这个将被用于之后的用户登录。

## 2. 引入 Gitment
将下面的代码添加到你的页面：
  ```
  <div id="container"></div>
  <link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
  <script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
  <script>
  var gitment = new Gitment({
    id: '页面 ID', // 可选。默认为 location.href
    owner: '你的 GitHub ID',
    repo: '存储评论的 repo',
    oauth: {
      client_id: '你的 client ID',
      client_secret: '你的 client secret',
    },
  })
  gitment.render('container')
  </script>
  ```
注意，上述代码引用的 Gitment 将会随着开发变动。如果你希望始终使用最新的界面与特性即可引入上述代码。

如果你希望引用确定版本的 Gitment，则应该使用 npm 进行安装。

 `$ npm install --save gitment`
关于构造函数中的更多可用参数

- owner:是你的github登录名,
- repo:是你的github里面的项目名,初始化的评论会放在项目的issues中
- id:页面评论在issues的唯一标示,不宜过长,否则会报错

我的示例:

  ```
  <script>
    var gitment = new Gitment({
      id: '{{ .File.BaseFileName }}',
      owner: 'wangzitao6',
      repo: 'wangzitao6.github.io',
      oauth: {
        client_id: 'a09xxxxxxxxxxxxxxx',
        client_secret: 'ec6b34b6xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
      }
    })
    gitment.render('git-comments')
  </script>

  ```

## 3. 初始化评论
页面发布后，你需要访问页面并使用你的 GitHub 账号登录（请确保你的账号是第二步所填 repo 的 owner），点击初始化按钮。
![](
https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/03/002.png)

之后其他用户即可在该页面发表评论。

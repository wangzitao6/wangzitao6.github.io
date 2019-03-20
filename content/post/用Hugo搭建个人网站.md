---
title:     "用hugo搭建个人网站"  # 文章标题
subtitle:    "hugo"  # 文章标题
description: "用Hugo搭建个人网站"
date:         2017-08-28T23:41:16+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["blog","其他"]
url:    "/2017-08-28-用hugo搭建个人网站/"
categories:  ["other"]
---


- 摘要：
  本文将介绍什么是Hugo，怎么用Hugo搭建个人网站，如何本地测试及如何发布到Github并生成Github pages。

## 什么是Hugo
-------
Hugo是一种静态网站生成器。适用于搭建个人博客、小型公司主页等网站，是一种小型的CMS系统。

静态站点的好处就是快速、安全、易于部署，最主要是可以通过版本控制来进行管理。

静态网站生成器有很多种，Github上有总结，知名的有Jekyll，Middle Man App，等等。

我之所以选择Hugo，首先是因为它支持Windows系统，并且安装很简单。其次是对markdown语法的支持，这对我来说很方便。然后是主题、文档支持等等各方面都比较完善。

## 如何用Hugo搭建个人网站
=============
### 1.下载和安装Hugo
-----------

Hugo是用Go语言写的，早期版本还要下载Go，目前版本是v0.18.1，直接下载，不再需要额外的依赖了。

win64x对应的是hugo_0.18.1_Windows-64bit.zip，下载后创建安装目录，例如D:\Hugo，之下建两个子目录bin和Sites，然后解压，例如解压到D:\Hugo\bin，把解压的hugo_0.18.1_windows_amd64.exe文件重命名为hugo.exe，然后加到环境变量Path里，方便在命令行里使用。

添加成功后打开cmd或者PowerShell，运行命令hugo version，如果安装成功，会输出Hugo Static Site Generator v0.54.0-B1A82C61 windows/amd64 BuildDate: 2017-08-01T09:42:02Z。

### 2.搭建个人网站
--------

首先要确定自己要搭建什么网站，我要建的是托管到Github的用户网站，按照Github Pages规则，网站名应该是<username.github.io>，所以我第一步创建网站用以下命令：

    cd D:\Hugo\Sites
    hugo new site brent-li.github.io

之后在Site目录下多了一个brent-li.github.io文件夹，进入文件夹可以看到目录结构如下：

    |-- archetypes
    |-- config.toml
    |-- content
    |-- data
    |-- layouts
    `-- static

archetypes目录里可以放一些原型，用于hugo新建内容的配置属性。config.toml是网站的配置属性文件。content文件夹里放你网站的内容，例如你发布的博客文章。data目录是Hugo使用的配置文件存放的地方。layout目录存放布局内容。static目录存放静态资源如图片、css等。

接下来我们先在content里放点东西。命令如下：

    cd brent-li.github.io
    hugo new post/first.md

Hugo会在content目录下创建post目录，在post目录下创建 first.md文件。之后打开md文件，里面已经有些内容

    +++
    date = "2017-10-08T22:07:46+08:00"
    title = "first"
    draft = true
    +++

+++包起来的内容是TOML配置信息，叫作扉页(front matter)，默认这3项，可以再添加一些，其中draft是true时Hugo不会真正发布它，我修改后的扉页如下：

    +++
    date = "2017-02-08T22:07:46+08:00"
    title = "Scala学习笔记之模式匹配"
    draft = false
    tags = ["scala","pattern matching","模式匹配"]
    share = true
    +++

然后再把我的博客内容复制进md文件，一篇博客完成了。接下来该给网站添加主题来装饰一下了。

Hugo主题网站提供了很多主题，选择自己喜欢的下载，我选择了casper，在自己网站目录下创建themes目录，然后下载主题：

    cd themes
    git clone git clone https://github.com/zhaohuabing/hugo-theme-cleanwhite.git white

然后配置config.toml文件如下：
  ```
  baseurl = "https://wangzitao6.github.io" #一定要用https，github强制安全措施
  title = "王子滔的博客"
  theme = "white"
  languageCode = "zh_CN"
  # Enable comments by entering your Disqus shortname
  DisqusShortname = "WangZiTao"
  googleAnalytics = ""
  preserveTaxonomyNames = true
  paginate = 5 #frontpage pagination
  hasCJKLanguage = true
  [outputs]
  home = ["HTML", "Algolia"]

  [params]
    header_image = "images/1.jpg"
    description = "王子滔, 美食爱好者，生活探险家 | 这里是 王子滔 的博客，与你一起发现更大的世界。"
    keyword = "王子滔, wangzitao, WangZiTao, 王子滔的网络日志, 王子滔的博客, 博客, 个人网站, 互联网, Web, 云原生, PaaS, Istio, Kubernetes, 微服务, Microservice"
    slogan = "~路漫漫其修远兮~"

    image_404 = "images/404-bg.jpg"
    title_404 = "你来到了没有知识的荒原 :("

    # leancloud storage for page view counter
    page_view_conter = false
    leancloud_app_id = ""
    leancloud_app_key = ""

    # algolia site search
    algolia_search = true
    algolia_appId = "WM4BEY1UDN"
    algolia_indexName = "blog"
    algolia_apiKey = "090c4a77b8bd4b8d2f2c1262afbc4be2"


    # Sidebar settings
    sidebar_about_description = "搬砖工程师"
    #sidebar_avatar = "images/avatar-1.jpg"      # use absolute URL, seeing it's used in both `/` and `/about/`
    sidebar_avatar = "images/me.jpg"      # use absolute URL, seeing it's used in both `/` and `/about/`

    featured_tags = false
    featured_condition_size = 1

    # Baidu Analytics
    ba_track_id = "28c952e35b6e860c096a2cdf52785c6f"

    # We need a proxy to access Disqus api in China
    # Follow https://github.com/zhaohuabing/disqus-php-api to set up your own disqus proxy
    disqus_proxy = ""
    disqus_site = ""

    friends = false
    bookmarks = false
    about_me = true

    [params.social]
    rss            = false
    email = "wang_zitao@foxmail.com" #邮箱
    #facebook      = "full profile url in facebook"
    #googleplus    = "full profile url in googleplus"
    #twitter       = "full profile url in twitter"
    #linkedin       = "https://www.linkedin.com/in/yourlinkedinid"
    #stackoverflow  = "https://stackoverflow.com/users/yourstackoverflowid"
    #instagram     = "full profile url in instagram"
    github = "https://github.com/wangzitao6" #github用户名
    wechat= "images/qrcode.jpg"  # Replace with your wechat qrcode image
    #medium         = "full profile url in medium"
    #pinterest     = "full profile url in pinterest"


  [outputFormats.Algolia]
  baseName = "algolia"
  isPlainText = true
  mediaType = "application/json"
  notAlternative = true

  [params.algolia]
  vars = ["title", "summary", "date", "publishdate", "expirydate", "permalink"]
  params = ["categories", "tags"]
  ```

> 配置完不要忘了把封面、头像图片都拷贝到static\images目录下。

### 3.本地测试


Hugo自带服务器，可以用命令行启动：

    hugo server -t white

服务器启动后访问http://localhost:1313访问网站，发现问题可以及时修改。

### 4.发布到github
-----------

本地测试网站没有问题后，就可以准备发布了。执行以下命令

    hugo -t casper

Hugo将编译所有文件并输出到public目录，你需要在github上创建repository，名字就是<你的用户名>.github.io，创建完后，返回你本地命令行，进入public目录，执行以下命令：

    git init
    git add .
    git commit -m "Initial commit."
    git remote add origin git@github.com:Brent-Li/brent-li.github.io.git
    git push -u origin master

稍等片刻后，打开<你的用户名>.github.io网址，就可以看到你的个人网站了。

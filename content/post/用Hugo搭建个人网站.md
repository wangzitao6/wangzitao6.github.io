+++
title = "用Hugo搭建个人网站"  # 文章标题
date = 2017-10-01T22:59:30+08:00  # 自动添加日期信息
draft = false  # 设为false可被编译为HTML，true供本地修改
tags = ["其他"]  # 文章标签，可设置多个，用逗号隔开。Hugo会自动生成标签的子URL
comments = true  # 是否开启Disqus评论功能
share = true  # 是否开启分享
+++

摘要：
本文将介绍什么是Hugo，怎么用Hugo搭建个人网站，如何本地测试及如何发布到Github并生成Github pages。

什么是Hugo
-------
Hugo是一种静态网站生成器。适用于搭建个人博客、小型公司主页等网站，是一种小型的CMS系统。

静态站点的好处就是快速、安全、易于部署，最主要是可以通过版本控制来进行管理。

静态网站生成器有很多种，Github上有总结，知名的有Jekyll，Middle Man App，等等。

我之所以选择Hugo，首先是因为它支持Windows系统，并且安装很简单。其次是对markdown语法的支持，这对我来说很方便。然后是主题、文档支持等等各方面都比较完善。

如何用Hugo搭建个人网站
=============
一 下载和安装Hugo
-----------

Hugo是用Go语言写的，早期版本还要下载Go，目前版本是v0.18.1，直接下载，不再需要额外的依赖了。

win64x对应的是hugo_0.18.1_Windows-64bit.zip，下载后创建安装目录，例如D:\Hugo，之下建两个子目录bin和Sites，然后解压，例如解压到D:\Hugo\bin，把解压的hugo_0.18.1_windows_amd64.exe文件重命名为hugo.exe，然后加到环境变量Path里，方便在命令行里使用。

添加成功后打开cmd或者PowerShell，运行命令hugo version，如果安装成功，会输出Hugo Static Site Generator v0.18.1 BuildDate: 2017-02-08T21:36:59+08:00。

二 搭建个人网站
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
    hugo new post/scala-learning-pattern-matching.md

Hugo会在content目录下创建post目录，在post目录下创建scala-learning-pattern-matching.md文件。之后打开md文件，里面已经有些内容

    +++
    date = "2017-02-08T22:07:46+08:00"
    title = "scala learning pattern matching"
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
    git clone https://github.com/vjeantet/hugo-theme-casper casper

然后配置config.toml文件如下：

    languageCode = "zh_CN" #语言
    title = "李子峰的Github Page" #博客标题
    baseurl = "https://brent-li.github.io/" #一定要用https，github强制安全措施
    paginate = 5 #分页每页记录数
    DisqusShortname = "李子峰" #评论时显示的名字
    Copyright = "All rights reserved - 2017" #版权
    canonifyurls = true

    [params]
      description = "知其雄，守其雌，为天下溪。" #加段提升逼格的副标题
      cover = "images/cover.jpg" #自己找的博客封面，要够大够酷
      author = "李子峰" #文章作者
      authorlocation = "Beijing, China" #作者位置
      authorwebsite = "https://brent-li.github.io/" #作者站点
      bio= "京东|高级软件工程师" #作者简介
      logo = "images/Einstan.jpg" #作者头像
      #googleAnalyticsUserID = "UA-79101-12"
      # Optional RSS-Link, if not provided it defaults to the standard index.xml
      #RSSLink = "http://feeds.feedburner.com/..."
      githubName = "Brent-Li" #github用户名
      #twitterName = "vjeantet"
      # facebookName = ""
      linkedinName = "zifeng" #LinkedIn用户名
      # set true if you are not proud of using Hugo (true will hide the footer note "Proudly published with HUGO.....")
      hideHUGOSupport = false #是否显示Hugo水印
      [params.social]
        linkedin = "https://cn.linkedin.com/in/zifeng"

    [[menu.main]] #页面菜单参数
      name = "李子峰的博客"
      weight = -120
      identifier = "blog"
      url = "/"

    [[menu.main]]
      name = "About me"
      weight = -110
      identifier = "about"
      url = "/about"

配置完不要忘了把封面、头像图片都拷贝到static\images目录下。

三 本地测试
------

Hugo自带服务器，可以用命令行启动：

    hugo server -t casper

服务器启动后访问http://localhost:1313访问网站，发现问题可以及时修改。

四 发布到github
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

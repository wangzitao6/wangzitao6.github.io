---
title:     "gitment登录失败"  # 文章标题
subtitle:    "gitment登录失败"  # 文章标题
description: "博客评论插件gitment登录失败修复"
date:         2018-06-12T09:31:57+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["blog","其他"]
url:    "/2018-06-12-gitment登录失败/"
categories:  ["other"]
---
## gitment登录失败
搭建个人博客之后, 最终还是选择了gitment. 最近突然发现gitment登录失败,报错 [object ProgressEvent],去官网看下发现是域名https://gh-oauth.imsun.net证书过期了,
地址:`https://github.com/imsun/gitment/issues/170`
里面有几种解决方法
## 解决方法
### 1.本地解决
由于引入的 gitment.js 中有这样的一段代码：
  ```
   _utils.http.post('https://gh-oauth.imsun.net', {
      code: code,
      client_id: client_id,
      client_secret: client_secret
    }, '').then(function (data) {
      _this.accessToken = data.access_token;
      _this.update();
    }).catch(function (e) {
      _this.state.user.isLoggingIn = false;
      alert(e);
    });

  ```
请求了一个服务接口，由于这个服务接口是作者自己搭建的，已经停止了。
这里可以直接改为请求 github 认证的接口
 ```
 _utils.http.post('https://github.com/login/oauth/access_token', {...}
 ```
就可以了,不用经过作者的服务

### 2.借用别人的服务
由于作者貌似已经弃坑,无法登陆的问题一直没有得到修复,许多大牛已经自己搭建了服务,可以在征得他们同意后,修改配置

![](
https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/18/06/001.png)

把Install那一步换成这两个：
   ```
   <link rel="stylesheet" href="https://jjeejj.github.io/css/gitment.css">
   <script src="https://www.wenjunjiang.win/js/gitment.js"></script>
   ```
### 3.自己搭建服务

这个我没有试过,有兴趣的朋友可以参考这里:

  **[参考链接](https://smalbox.club/2018/10/24/an-zhuang-gitment-ji-chang-jian-wen-ti-jie-jue/)**

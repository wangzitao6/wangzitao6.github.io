---
title:     "Markdown进阶语法"  # 文章标题
subtitle:    "Markdown进阶语法"  # 文章标题
description: "Markdown"
date:         2019-08-02T13:59:37+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   "https://img.zhaohuabing.com/post-bg-2015.jpg"
tags:        ["其他"]
url:    "/2018-08-02-markdown进阶语法"
categories:  ["other"]
showtoc: true   # 是否显示目录
---

## 字体、大小、颜色
```
<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color="#00dd00">我是浅绿色</font><br />
<font color=Blue>我是蓝色</font>
<font size=8>我是尺寸</font>
<font face="黑体" color=red size=8>我是黑体，红色，尺寸为8</font>
```

效果：

<font face="黑体">我是黑体字</font>
<font face="微软雅黑">我是微软雅黑</font>
<font face="STCAIYUN">我是华文彩云</font>
<font color=red>我是红色</font>
<font color=#008000>我是绿色</font>
<font color="#00dd00">我是浅绿色</font><br />
<font color=Blue>我是蓝色</font>
<font size=8>我是尺寸</font>
<font face="黑体" color=red size=8>我是黑体，红色，尺寸为8</font>

## 空格

一个汉字占两个空格大小，所以使用四个空格就可以达到首行缩进两个汉字的效果。有如下几种方法：

* 一个空格大小的表示：**&ensp**;或 **&#8194**;，此时只要在相应需要缩进的段落前加上 4个 如上的标记即可，**注意要带上分号**。

* 两个空格的大小表示：**&emsp**;或 **&#8195**;，同理，使用2个即可缩进2个汉字，**推荐使用该方式**。

* 不换行空格：**&nbsp**;或 **&#160**;，使用4个 **&#160**;即可。

## 文字背景

```
<table><tr><td bgcolor=red>红色背景</td></tr></table>
```
效果：
<table><tr><td bgcolor=red>红色背景</td></tr></table>

参考：[Markdown: Basics （快速入门）](https://www.appinn.com/markdown/basic.html)

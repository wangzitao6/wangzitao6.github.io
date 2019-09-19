---
title:     "了解ARP协议"  # 文章标题
subtitle:    "了解ARP协议"  # 文章标题
description: ""
date:         2019-04-19T13:50:49+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["网络"]
url:    "/2019-04-19-了解arp协议/"
categories:  ["net"]
showtoc: true   # 是否显示目录
---

## MAC地址

什么是MAC地址？MAC地址与IP有什么区别？
![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/6.png)

大概了解后禁不住会问，那么有了IP地址为什么还要一个MAC地址呢？
知乎回答:[https://www.zhihu.com/question/21546408](https://www.zhihu.com/question/21546408)

互联网是范围概念；以太网是技术概念。

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/7.png)

　Ethernet物理地址长度为48位，通常表示为12个16进制数，每2个16进制数之间用冒号隔开，如：08:00:20:0A:8C:6D就是一个MAC地址，其中前6位16进制数08:00:20代表网络硬件制造商的编号，它由IEEE（电气与电子工程师协会）分配。理论上存在物理地址为2^48个，但是第32位为组播标识位，必须为0，因此允许分配的物理地址为2^47个，即140737488355328个！
　　所以2^48=281474976710656是错误的！
　　标准为2^47=140737488355328个。
![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/8.png)

## ARP：地址解析协议

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/9.png)

### 在同一局域网中

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/10.png)

用ping说明ARP工作的原理
假设我们的计算机IP地址是192.168.1.1，要执行这个命令：ping192.168.1.2。该命令会通过ICMP协议发送ICMP(以太网控制报文协议)数据包
该过程需要经过下面的步骤：

```java　　
1> 应用程序构造数据包，该示例是产生ICMP包，被提交给内核（网络驱动程序）； 　　
2> 内核检查是否能够转化该IP地址为MAC地址，也就是在本地的ARP缓存中查看IP-MAC对应表;
3> 如果存在该IP-MAC对应关系，那么跳到步骤<7；如果不存在该IP-MAC对应关系，那么接续下面的步骤;
4> 内核进行ARP广播，目的MAC地址是FF-FF-FF-FF-FF-FF，ARP命令类型为REQUEST（1），其中包含有自己的MAC地址； 　　
5> 当192.168.1.2主机接收到该ARP请求后，就发送一个ARP的REPLY（2）命令，其中包含自己的MAC地址； 　　
6> 本地获得192.168.1.2主机的IP-MAC地址对应关系，并保存到ARP缓存中； 　　
7> 内核将把IP转化为MAC地址，然后封装在以太网头结构中，再把数据发送出去；
```

### 在不同局域网中

寻址：从一个LAN路由至另一个LAN

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/11.png)

step 1:

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/12.png)

step 2:
![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/13.png)

如果左侧是内网，右侧是互联网的话，可能要更改源IP地址；</br>
如果左侧是互联网，右侧是内网，可能要更改目的IP地址；

step 3:
![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/19/14.png)

如果所要找的目标设备和源主机不在同一个局域网上。

```java
1>此时主机A就无法解析出主机B的硬件地址（实际上主机A也不需要知道远程主机B的硬件地址）;
2>此时主机A需要的是将路由器R1的IP地址解析出来，然后将该IP数据报发送给路由器R1.
3>R1从路由表中找出下一跳路由器R2，同时使用ARP解析出R2的硬件地址。于是IP数据报按照路由器R2的硬件地址转发到路由器R2。
4>路由器R2在转发这个IP数据报时用类似方法解析出目的主机B的硬件地址，使IP数据报最终交付给主机B.
```

说明：</br>
如果你的数据包是发送到不同网段的目的地，那么就一定存在一条网关的IP-MAC地址对应的记录。知道了ARP协议的作用，就能够很清楚地知道，数据包的向外传输很依靠ARP协议，当然，也就是依赖ARP缓存。要知道，ARP协议的所有操作都是内核自动完成的，同其他的应用程序没有任何关系。同时需要注意的是，ARP协议只使用于本网络。


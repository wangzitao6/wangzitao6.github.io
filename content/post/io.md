---
title:     "Io"  # 文章标题
subtitle:    "Io"  # 文章标题
description: ""
date:         2021-04-15T15:35:35+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["io"]
url:    "/2021-04-15-异步io/"
categories:  ["other"]
showtoc: true   # 是否显示目录
---


IO  ->网络通信IO socket ->BIO NIO 多路复用  ->Netty

BIO 每线程，每连接
优势:可以接收很多的连接
缺点:线程内存浪费 ，CPU调度消耗
根源 BLOCKING  阻塞 accept recv
解决方案 NONBLOCKING

NIO  一个是java New IO   一个是操作系统的nonblocking

java New IO 中会有对OS NONBLOCKING 的使用

ss.accept()时不会被卡住，会有返回(没有返回空，有就返回客户端)


NIO
优势 规避多线程的问题
弊端：当有一万个连接，但是只有一个连接发送消息的时候，每循环一次，会向内核发送一万次recv的请求调用，有9999此时无意义的 浪费和消耗资源


多路复用  多路复用器只是读取的状态，哪几个连接可以获取数据，由程序自己去发送recv请求调用获取数据

select 最多接收1024个链接
poll   不限制链接数

select、poll多路复用器
优势： 用过一次调用， 把fds传递给内核，有内核去遍历，返回可读的链接，这种遍历减少了系统调用的次数
弊端：  1，重复传递fd(文件标示符)    解决方案，内核开辟空间保留fd
2,每次select、poll 都要重新遍历全量的fd   解决方案（计算机组成原理的深度知识，中断，callback,增强）


epoll


同步IO模型： 如果程序自己读取IO,无论是BIO、NIO、多路复用器(select、poll、epoll)，统一称为同步IO模型
异步IO模型：windows IOCP 内核有线程，将数据拷贝的程序的内存空间





=======================================================================================================================================================


在java中，NIO(New IO)有三个核心部分组成，分别是Buffer(缓冲区)，Channel(管道),以及Selector(选择器)
IO处理客户端请求的最小单位是线程
而NIO使用了比线程还小一级的单位：通道（Channel）
可以说，NIO中只需要一个线程就能完成所有接收，读，写等操作

可以简单的理解为
Buffer:是需要存储数据的地方。
Channel:是运输数据的载体。
Selector:用于检查多个Selector变更的状况。


												 Channel <-----(data)----> Channel 
[client] ----(data)----> Buffer ----(data)---> 										----(data)----> Buffer ----(data)---> [Server]

ByteBuffer:

Buffer中对应的Position， Mark， Capacity，Limit都啥？
capacity：缓冲区容量的大小，就是里面包含的数据大小。
limit：对buffer缓冲区使用的一个限制，从这个index开始就不能读取数据了。
position：代表着数组中可以开始读写的index， 不能大于limit。
mark：是类似路标的东西，在某个position的时候，设置一下mark，此时就可以设置一个标记



所有的数据只会存在于缓存区中，无论你是写或是读，必然是缓存区通过通道到达磁盘文件，或是磁盘文件通过通道到达缓存区
即缓存区是数据的「起点」，也是「终点」
// 要使用NIO，有了Channel，就必然要有Buffer，Buffer是与数据打交道的呢，为ByteBuffer分配空间
ByteBuffer buffer = ByteBuffer.allocate(1024);

//向Buffer写入数据
1).数据从Channel到Buffer：channel.read(byteBuffer);
2).数据从Client到Buffer： byteBuffer.put(...);

//从Buffer读出数据
1).数据从Buffer到Channel：channel.write(byteBuffer);
2).数据从Buffer到Server： byteBuffer.get(...);


Selector
可使一个单独的线程管理多个 Channel， open 方法可创建 Selector， register 方法向多路复用器器注册通道，
可以监听的事件类型：读、写、连接、 accept。注册事件后会产生一个 SelectionKey：
它表示 SelectableChannel 和 Selector 之间的注册关系， wakeup 方法：使尚未返回的第一个选择操作立即返回，
唤醒的原因是：注册了新的 channel 或者事件； channel 关闭，取消注册；优先级更高的事件触发（如定时器事件），希望及时处理
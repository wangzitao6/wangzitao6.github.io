---
title:     "jvm调优"  # 文章标题
subtitle:    "jvm调优"  # 文章标题
description: ""
date:         2021-07-12T15:35:35+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["jvm"]
url:    "/2021-07-12-jvm调优/"
categories:  ["jvm"]
showtoc: true   # 是否显示目录
---

jps：查看当前java进程id，java本身就是一个进程

jinfo -flags 6198  查看jvm参数

java -XX 查看更详细的jvm参数

java -XX:+PrintFlagsInitial  查看jvm设置初始值的
java -XX:+PrintFlagsFinal   jvm参数的的最终值



查看fullGC情况：
jstat -gc 12538 5000

即会每5查看fullGc次数秒一次显示进程号为12538的java进成的GC情况

查看堆情况
jmap -heap 10900

jmap -heap pid：输出堆内存设置和使用情况(JDK11使用jhsdb jmap --heap --pid pid)
jmap -histo pid：输出heap的直方图，包括类名，对象数量，对象占用大小
jmap -histo:live pid：同上，只输出存活对象信息
jmap -clstats pid：输出加载类信息
jmap -help：jmap命令帮助信息



jconsole pid
是jdk自带的一个内存分析工具，它提供了图形界面。可以查看到被监控的jvm的内存信息，线程信息，类加载信息，MBean信息

jvisualvm
监控内存泄露，跟踪垃圾回收，执行时内存、cpu分析，线程分析，可以安装VisualGC插件


打印dump日志
jstack 14 > goods_20200328.dump


查看端口占用统计
watch 'netstat -anp | grep 1521 | wc -l'
netstat -anp | grep 749 | wc -l
netstat -anp | grep 25590 | grep java | wc -l


查线程占用
top -Hp 23344可以查看该进程下各个线程的cpu使用情况


Xloggc
-Xloggc:gc-%t.log
-Xloggc:E:/data/gc.log


-XX:+PrintGCDetails  -XX:+PrintGCDateStamps  
-Xloggc:/var/log/hbase/gc-regionserver-hbase.log   
-XX:+UseGCLogFileRotation  
-XX:NumberOfGCLogFiles=10  -XX:GCLogFileSize=512k

-XX:+PrintGCDetails
输出GC的详细日志

-XX:+PrintGCDateStamps
输出GC的日期戳

-Xloggc:/var/log/hbase/gc-regionserver-hbase.log
GC日志输出的路径

-XX:+UseGCLogFileRotation
打开GC日志滚动记录功能

-XX:NumberOfGCLogFiles
设置滚动日志文件的个数，必须大于等于1
日志文件命名策略是，.0, .1, …, .n-1，其中n是该参数的值



=======================================================================================================================================

https://blog.csdn.net/aiwaston/article/details/104936473

监控服务器上的tomcat
tomcat的配置文件catalina.sh中增加

JAVA_OPTS="-Dcom.sun.management.jmxremote.port=9998
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false
-Djava.rmi.server.hostname=192.168.58.164"


指定了JMX启动的代理端口，这个端口就是visualvm要连接的端口（9998端口不能被别的程序使用netstat -an|gerp 9998）
Dcom.sun.management.jmxremote.port=9998

指定了JMX是否启用ssl
Dcom.sun.management.jmxremote.authenticate=false

指定了JMX是否启用鉴权（需要用户名，密码鉴权）
Dcom.sun.management.jmxremote.authenticate=false

指定了服务器主机名
Djava.rmi.server.hostname=192.168.58.164



fullGC 调优
出现fullGc后，先看fullGC后能不能有效的把内存回收掉，同时把堆dump下来分析下原因
需要找到问题，对症下药

如果fullGC没有效果的话，可能发生了内存泄露，需要优化代码

如果fullGC有效果，堆内存配置不合理，导致老年代很快被塞满了。

还有一种可能是业务量上来了，服务器配置扛不住了，需要扩容

根据具体情况来分析是需要改代码、调整堆大小、扩容等

或者修改下垃圾回收器，把CMS升级成G1


查看fullGC情况：
jstat -gc 12538 5000


打印dump日志
jmap 14 > goods_20200328.dump

arthas  dashboard


cpu过高

死循环，死锁


OOM案例
导入excel问题
workbook导致的
后面再代码中设置最大100条，其余的暂时写入临时文件中
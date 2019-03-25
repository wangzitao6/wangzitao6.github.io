---
title:     "HotSpot选项笔记"  # 文章标题
subtitle:    "HotSpot选项笔记"  # 文章标题
description: ""
date:       2018-06-05T11:05:33+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["jvm"]
url:    "/2018-06-05-虚拟机hotspot选项笔记"
categories:  ["jvm"]
showtoc: true   # 是否显示目录
---
## 语法
> **java [ options ] class [ argument ... ]**

> **java [ options ] -jar file.jar [ argument ... ]**

## 标准选项


**-d32|-d64**
</br>
**-client|-server**
</br>
 以客户端模式还是服务器模式执行虚拟机。</br>
 服务器模式的特点是启动速度比较慢，但运行时性能和内存管理效率很高，适用于生产环境。</br>
 对于64位虚拟机而言，服务器模式是默认值，也是唯一支持的模式。

**-agentlib:库名[=启动选项]** </br>
**-agentpath:绝对路径[=启动选项]** </br>
 从指定的路径(必须是绝对路径)/库名称中加载JVMT Tool Interface agent库(还可传递启动选项)。</br>
例如：
```
-agentlib:foo=opt1,opt2
-agentpath:/opt/lib/foo.so=opt1,opt2
```

**-classpath 路径列表** </br>
**-cp 路径列表** </br>
 指定一个搜索classic文件的目录/JAR/ZIP列表，列表项之间用分号(;)分隔。 </br>
 这里指定的值将会覆盖CLASSPATH环境变量的设置。 </br>
 如果既没有指定 -classpath 也没有设置CLASSPATH环境变量，那么其默认值将是用户的当前目录(.) </br>
 可以用星号(\*)作为通配符，表示目录下的所有.jar文件。单独一个星号表示用户的当前目录下的所有.jar文件。</br>

**-D属性=值** </br>
 将系统"属性"设为"值"。如果"值"是一个包含空格的字符串，那么必须用双引号进行界定。

**-enableassertions[:包名"..." | :类名 ]** </br>
**-ea[:包名"..." | :类名 ]** </br>
**-disableassertions[:包名"..." | :类名 ]** </br>
**-da[:包名"..." | :类名 ]** </br>
 为指定的包或者类启用/禁止断言，默认值为禁止。

**-enablesystemassertions**</br>
**-esa**</br>
**-disablesystemassertions**</br>
**-dsa**</br>
仅对所有的系统类启用/禁止断言</br>

**-jar file.jar**</br>
 执行封装在file.jar文件内的程序。</br>
 使用此选项后，file.jar文件内必须包含所有的用户class，并且所有-classpath选项都将被忽略。</br>


**-javaagent:jar文件[=选项]**</br>
 指定jvm启动时装入java语言代理。参见：`java.lang.instrument`</br>

**-jre-restrict-search**</br>
**-no-jre-restrict-search**</br>
 在版本搜索的时候，包含/排除用户私人的JRE

**-verbose**</br>
**-verbose:class**</br>
输出jvm载入类的相关信息，当jvm报告说找不到类或者类冲突时可此进行诊断。

**-verbose:gc**</br>
输出每次垃圾回收的相关情况。

**-verbose:jni**</br>
输出native方法调用的相关情况，一般用于诊断jni调用错误信息。

**-version**</br>
输出java的版本信息

**-version:版本信息**</br>
指定class或者jar运行时需要的jdk版本信息；若指定版本未找到，则以能找到的系统默认jdk版本执行；</br>
一般情况下，对于jar文件，可以在manifest文件中指定需要的版本信息，而不是在命令行。</br>
"版本信息"中可以指定单个版本，也可以指定一个列表，中间用空格隔开，且支持复杂组合，例如：-version:"1.6.0_13 1.6* & 1.6.0_10+"</br>

**-showversion**</br>
输出java版本信息(与-version相同)之后，继续输出java的标准参数列表及其描述。

**-?**</br>
**-help**</br>
输出java标准参数列表及其描述。

**-X**</br>
输出非标准的参数列表及其描述。

## 非标准选项

**-Xint**</br>
仅以解释模式运行虚拟机，全部字节码都以解释方式执行

**-Xcomp**</br>
仅以编译模式运行虚拟机，优先采用编译方式执行字节码，仅在无法编译的情况下解释器才介入执行

**-Xmixed**</br>
以混合模式运行虚拟机，对热点代码使用编译器执行，其他代码使用解释器执行。这是默认值。

**-Xbatch**</br>
禁止后台编译。</br>
默认情况下，对于尚未编译完成的热点代码，虚拟机会先使用解释方式执行，同时将编译任务放到后台。</br>
此选项将会把编译任务放到前台进行，直到编译完成后再继续执行。</br>

**-Xbootclasspath:启动类路径**</br>
**-Xbootclasspath/a:路径**</br>
**-Xbootclasspath/p:路径**</br>
指定一个分号(;)分隔的目录/JAR/ZIP列表，在其中搜索boot class</br>
此选项用于改变虚拟机自带的系统运行包"rt.jar"，除非你自己能写一个运行时，否则不会用到该参数。</br>
/a:在默认搜索路径后加上path中的搜索路径。</br>
/p:在默认搜索路径前加上path中的搜索路径。</br>

**-Xcheck:jni**</br>
调用JNI(Java Native Interface)函数时进行额外的检查，特别地，虚拟机将校验传递给JNI函数参数的合法性。</br>
在本地代码中遇到非法数据时，虚拟机将报一个致命错误而终止。使用该参数后将造成性能下降。

**-Xfuture**</br>
对类文件进行严格的格式检查，以保证类代码严格符合新规范。</br>
为保持向后兼容1.1.x版本的JDK，虚拟机缺省不进行严格的格式检查。</br>
建议明确开启此参数，特别是在开发阶段。</br>

**-Xnoclassgc**</br>
关闭虚拟机对class的垃圾回收功能。很可能会导致OutOfMemoryError异常。

**-Xincgc**</br>
启动增量垃圾收集器(默认关闭)。</br>
增量垃圾收集器能减少偶然发生的长时间垃圾回收造成的暂停时间。</br>
但增量垃圾收集器和应用程序并发执行，因此会抢占部分应用程序的CPU资源。</br>

**-Xloggc:文件**</br>
将虚拟机每次垃圾回收的信息写到"文件"中，内容和 -verbose:gc 的输出内容相同，但增加了时间戳。</br>
务必指定一个本机文件，以避免因为网络拥堵造成虚拟机停顿。</br>
当文件系统被填满后，日志文件会被自动从头截掉(类似日志滚动)，并继续记录。</br>
如果同时使用了 -verbose:gc 和本选项，那么 -verbose:gc 将会被忽略。</br>

**-Xms字节数**</br>
设置初始Java heap的大小，可以使用K或M或G后缀。建议和-Xmx设为相同。

**-Xmx字节数**</br>
设置最大Java heap的大小，可以使用K或M或G后缀。建议和-Xms设为相同。

**-Xss字节数**</br>
设置单个java线程的stack大小，可以使用K或M后缀。</br>
在64bit平台上默认是1024K，在32bit平台上默认是512K。</br>
建议不要设的太大(一般设为256K)，若无特殊情况，512K已经绰绰有余了。</br>

**-Xprof**</br>
对运行中的程序进行性能分析(profile)，并将结果发送到标准输出。仅用于开发目的，切勿用于生产环境！

**-Xrs**</br>
Shutdown Hook功能允许应用程序在终止前(可能是虚拟机自身被终止所致)执行一些诸如关闭数据库连接之类的清理操作。</br>
JVM会注册一个控制台控制句柄(console control handler)，用于捕获要求虚拟机进程终止的控制信号。</br>
该句柄会触发shutdown-hook进程，同时对 SIGHUP, SIGINT, SIGTERM, CTRL_LOGOFF_EVENT 信号返回TURE。</br>
如果JVM是作为service运行(例如Tomcat)，它可以接受CTRL_LOGOFF_EVENT信号但却不应该终止整个进程。</br>
使用该选项之后，JVM将不会注册控制台控制句柄，因此也就不会对上述信号做出响应。</br>
使用此选项会造成如下两个后果：</br>
(1)Ctrl-Break线程转储功能将不可用</br>
(2)应用程序代码必须自己负责触发Shutdown Hook，比如在JVM终止前调用System.exit()方法。

## 不稳定选项

这些"不稳定"选项的意思是很容易在没有通知的情况下改变。语法格式如下：</br>

> -XX:+OPT    开启OPT选项 </br>
> -XX:-OPT    关闭OPT选项 </br>
> -XX:OPT=VAL 将OPT选项的值设为VAL </br>

说明："="后面的内容表示的是此选项的默认值。

### 内存管理参数
JVM最大总内存 = 线程数*(-Xss) + (-Xmx) + MaxPermSize + MaxDirectMemorySize + ReservedCodeCacheSize　</br>


**DisableExplicitGC = false**</br>
忽略程序中System.gc()方法触发的垃圾收集动作

**MaxDirectMemorySize = -1**</br>
最大直接内存大小，可以使用K或M或G后缀。默认值"-1"表示使用-Xmx的值。

**PermSize = [依赖于物理内存大小]**</br>
永久代区域的初始值，可以使用K或M或G后缀。

**MaxPermSize = [依赖于物理内存大小]**</br>
永久代区域的最大值，可以使用K或M或G后缀。</br>
一些JSP页面可能需要很大的永久代去存放动态生成与加载的类。

**NewRatio = 2**</br>
老年代对新生代的比例。默认值2表示老年代是新生代大小的2倍。</br>
如果想要更精细的调整新生代的大小，应该使用下面两个参数：
**NewSize = [依赖于-Xms,-Xmx的设置]**</br>
新生代大小的初始值(最小值)，可以把这个参数设成与MaxNewSize相同来固定新生代的大小

**MaxNewSize = 无穷大**</br>
新生代大小的最大值(上限)，可以把这个参数设成与NewSize相同来固定新生代的大小

**SurvivorRatio = 8**</br>
新生代中Eden区域与Survivor区域容量的比值。</br>
默认值8表示Eden区的容量是Survivor区的8倍。</br>
因为有两个Survivor区，所以Eden:Survivor1:Survivor2=8:1:1

**PretenureSizeThreshold = 0**</br>
直接晋升到老年代的对象大小阈值(无默认值)，大于这个值的对象将直接在老年代内分配

**MaxTenuringThreshold = 15**</br>
晋升到老年代的对象的年龄阈值。</br>
对象每坚持过一次Minor GC之后，年龄加1，当超过这个阈值之后，该对象将进入老年代

**UseAdaptiveSizePolicy = true**</br>
允许JVM动态调整java堆中老年代、新生代区域的大小以及进入老年代的年龄

**UseG1GC = false**</br>
使用G1(Garbage First)垃圾收集器

**GCTimeRatio = 9**</br>
垃圾回收时间占总运行时间的比率=1/(1+GCTimeRatio)。默认值9表示最大允许1/(1+9)=10%的时间用于GC

**MaxGCPauseMillis = 200**</br>
设置GC最大停顿时间，也就是在垃圾回收时允许正常工作进程最大允许停顿多长时间，单位是毫秒

**GCPauseIntervalMillis = 201**</br>
设置两次GC之间的时间间隔，也就是每隔多长时间可以接受一次GC停顿 ，单位是毫秒

**UseGCOverheadLimit = true**</br>
如果有过多的时间花费在垃圾收集上(超过98%)，就抛OutOfMemoryError异常。</br>
这个功能是用来防止堆太小导致程序长时间无法正常工作而设计的。

**UseCompressedOops = false**</br>
指针压缩功能

### 即时编译参数

**BackgroundCompilation = true**</br>
使用后台编译。也就是在将方法编译为本地代码完成之前，先解释执行，而不是暂停执行

**TieredCompilation = false**</br>
启用分层编译策略：C0为解释执行，C1进行可靠优化编译，C2进行激进优化编译。</br>
分层编译可以逐步优化代码，以使最终运行速度更快，且降低C2编译时对执行速度的影响，建议开启。

**InitialCodeCacheSize = 4M**</br>
即时编译器编译的代码缓存的初始值，可以使用K或M或G后缀。

**ReservedCodeCacheSize = 48M**</br>
即时编译器编译的代码缓存的最大值，可以使用K或M或G后缀。

**CompileThreshold = 10000**</br>
触发即时编译的阈值。</br>
当调用计数器(用于函数)或者回边计数器(用于代码块)超过此值时，将被编译为机器码

**InterpreterProfilePercentage = 33**</br>
**OnStackReplacePercentage = 140**</br>
用于计算回边计数器阈值的两个参数，当计算结果大于CompileThreshold时，代码块所在的函数将被编译为机器码。

回边计数器阈值=CompileThreshold*(OnStackReplacePercentage-**InterpreterProfilePercentage)/100**</br>
**UseCounterDecay = true**</br>
对方法调用计数器使用热度衰减算法。</br>
建议关闭热度衰减算法，这样，只要运行时间足够长，绝大部分方法都将被编译成本地代码

### 类型加载参数

**UseSplitVerifier = true**</br>
使用新的Class类型校验器(依赖StackMapTable信息的类型检查)，以加快字节码校验速度，建议启用。

**FailOverToOldVerifier = true**</br>
当类型校验失败时，回到老的类型推导校验方式进行校验。建议关闭。

**RelaxAccessControlCheck = false**</br>
在校验阶段放松对类型访问性的限制。建议关闭。</br>

### 多线程相关参数

**UseBiasedLocking = true**</br>
使用偏向锁，它通过消除资源无竞争情况下的同步原语，进一步提高了程序的运行性能。

**UseSpinning = false**</br>
使用自旋锁以避免线程频繁的挂起和唤醒，建议开启

**PreBlockSpin = 10**</br>
在挂起线程前，使用自旋锁时的最大自旋次数，超过这个次数后，线程将被挂起

**UseThreadPriorities = true**</br>
使用线程优先级

**UseFastAccessorMethods = true**</br>
当频繁反射执行某个方法时，编译成字节码来加快反射的执行速度

### 性能参数

**AggressiveOpts = false**</br>
使用比较激进的优化特性，这些特性具有双重影响，并不一定能够加速应用的执行

**UseLargePages = false**</br>
使用大的内存页，需要操作系统内核的支持。在大内存的机器上建议开启

**LargePageSizeInBytes = 0**</br>
设置堆内存的内存页大小，一般是4M，需要操作系统内核的支持

### 调试参数

**PrintCommandLineFlags = false**</br>
打印当前启用的非稳态选项

**PrintFlagsFinal = false**</br>
输出所有不稳定选项的名称及其当前值

**PrintGCTimeStamps = false**</br>
打印每次回收开始时间的时间戳，对于查看垃圾回收频率非常有用。

**ErrorFile =**</br>
如果JVM崩溃，则将错误日志输出到指定的文件路径。

**HeapDumpPath =**</br>
当java进程因OutOfMemory或crash被OS强制终止后，生成一个Heap Profling格式的堆内存快照文件。

转载自:
[金步国作品集](http://www.jinbuguo.com/)

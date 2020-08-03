---
title:     "AbstractQueuedSynchronizer超详细原理解析"  # 文章标题
subtitle:    "AbstractQueuedSynchronizer超详细原理解析"  # 文章标题
description: ""
date:         2020-07-09T16:40:03+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["JUC"]
url:    "/2020-07-09-abstractqueuedsynchronizer超详细原理解析"
categories:  ["多线程"]
showtoc: true   # 是否显示目录
---

今天我们来研究学习一下<font color ='red'> **AbstractQueuedSynchronizer**</font>类的相关原理，<font color ='red'> **java.util.concurrent**</font>包中很多类都依赖于这个类所提供队列式同步器，比如说常用的<font color ='red'> **ReentranLock**</font>，<font color ='red'> **Semaphore**</font>和<font color ='red'> **CountDownLatch**</font>等。
 为了方便理解，我们以一段使用 <font color ='red'> **ReentranLock**</font>的代码为例，讲解<font color ='red'> **ReentranLock**</font>每个方法中有关AQS的使用。

 ![bb](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/20/07/09/lock%E7%9A%84%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.png)

 ![bb](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/20/07/09/%E5%A4%9A%E4%B8%AA%E5%9B%BE%E7%BB%84%E5%90%88.png)


# 1.0 ReentranLock示例

我们都知道<font color ='red'> **ReentranLock**</font>的加锁行为和<font color ='red'> **Synchronized**</font>类似，都是可重入的锁，但是二者的实现方式确实完全不同的，我们之后也会讲解<font color ='red'> **Synchronized**</font>的原理。**除此之外，Synchronized的阻塞无法被中断，而<font color ='red'> **ReentrantLock**</font>则提供了可中断的阻塞**。下面的代码是<font color ='red'> **ReentranLock**</font>的函数，我们就以此为顺序，依次讲解这些函数背后的实现原理。


    ReentrantLock lock = new ReentrantLock();
    lock.lock();
    lock.unlock();

# 2.0 公平锁和非公平锁

<font color ='red'> **ReentranLock**</font>分为公平锁和非公平锁，二者的区别就在获取锁机会是否和排队顺序相关。我们都知道，如果锁被另一个线程持有，那么申请锁的其他线程会被挂起等待，加入等待队列。理论上，先调用<font color ='red'> **lock**</font>函数被挂起等待的线程应该排在等待队列的前端，后调用的就排在后边。如果此时，锁被释放，需要通知等待线程再次尝试获取锁，公平锁会让最先进入队列的线程获得锁。而非公平锁则会唤醒所有线程，让它们再次尝试获取锁，所以可能会导致后来的线程先获得了锁，则就是非公平。

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

我们会发现<font color ='red'> **FairSync**</font>和<font color ='red'> **NonfairSync**</font>都继承了<font color ='red'> **Sync**</font>类，而<font color ='red'> **Sync**</font>的父类就是<font color ='red'> **AbstractQueuedSynchronizer**</font>(后续简称AQS)。但是<font color ='red'> **AQS**</font>的构造函数是空的,并没有任何操作。
 之后的源码分析，如果没有特别说明，就是指公平锁。

# 3.0 Lock操作
<font color ='red'> **ReentranLock的lock**</font>函数如下所示，直接调用了<font color ='red'> **sync**</font> 的 <font color ='red'> **lock**</font>函数。也就是调用了<font color ='red'> **FairSync的lock**</font>函数。


    //ReentranLock
    public void lock() {
        sync.lock();
    }
    //FairSync
    final void lock() {
        //调用了AQS的acquire函数,这是关键函数之一
        acquire(1);
    }

我们接下来就正式开始 <font color ='red'> **AQS**</font>相关的源码分析了， <font color ='red'> **acquire**</font>函数的作用是获取同一时间段内只能被一个线程获取的量，这个量就是抽象化的锁概念。我们先分析代码，你慢慢就会明白其中的含义。


    public final void acquire(int arg) {
        // tryAcquire先尝试获取"锁",获取了就不进入后续流程
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            //addWaiter是给当前线程创建一个节点,并将其加入等待队列
            //acquireQueued是当线程已经加入等待队列之后继续尝试获取锁.
            selfInterrupt();
    }

 <font color ='red'> **tryAcquire**</font>， <font color ='red'> **addWaiter**</font>和 <font color ='red'> **acquireQueued**</font>都是十分重要的函数，我们接下来依次学习一下这些函数，理解它们的作用。
 

    //AQS类中的变量.
    private volatile int state;
    //这是FairSync的实现,AQS中未实现,子类按照自己的需要实现该函数
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        //获取AQS中的state变量,代表抽象概念的锁.
        int c = getState();
        if (c == 0) { //值为0,那么当前独占性变量还未被线程占有
            //如果当前阻塞队列上没有先来的线程在等待,UnfairSync这里的实现就不一致
            if (!hasQueuedPredecessors() && 
                compareAndSetState(0, acquires)) {
                //成功cas,那么代表当前线程获得该变量的所有权,也就是说成功获得锁
                setExclusiveOwnerThread(current);
                // setExclusiveOwnerThread将本线程设置为独占性变量所有者线程
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            //如果该线程已经获取了独占性变量的所有权,那么根据重入性
            //原理,将state值进行加1,表示多次lock
            //由于已经获得锁,该段代码只会被一个线程同时执行,所以不需要
            //进行任何并行处理
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        //上述情况都不符合,说明获取锁失败
        return false;
    }

由上述代码我们可以发现，<font color ='red'> **tryAcquire**</font>就是尝试获取那个线程独占的变量<font color ='red'> **state**</font>。<font color ='red'> **state**</font>的值表示其状态：如果是0，那么当前还没有线程独占此变量；否在就是已经有线程独占了这个变量，也就是代表已经有线程获得了锁。但是这个时候要再进行一次判断，看是否是当前线程自己获得的这个锁，如果是，就增加<font color ='red'> **state**</font>的值。

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/20/07/09/%E6%B5%81%E7%A8%8B.webp)

ReentranLock获得锁

这里有几点需要说明一下，首先是<font color ='red'> **compareAndSetState**</font>函数，这是使用<font color ='red'> **CAS**</font>操作来设置<font color ='red'> **state**</font>的值，而且<font color ='red'> **state**</font>值设置了<font color ='red'> **volatile**</font>修饰符，通过这两点来确保修改<font color ='red'> **state**</font>的值不会出现多线程问题。然后是公平锁和非公平锁的区别问题，在<font color ='red'> **UnfairSync**</font>的<font color ='red'> **nonfairTryAcquire**</font>函数中不会在相同的位置上调用<font color ='red'> **hasQueuedPredecessors**</font>来判断当前是否已经有线程在排队等待获得锁。

如果<font color ='red'> **tryAcquire**</font>返回true，那么就是获取锁成功；如果返回false，那么就是未获得锁，需要加入阻塞等待队列。我们下面就来看一下<font color ='red'> **addWaiter**</font>的相关操作。

# 4.0  等待锁的阻塞队列

将保存当前线程信息的节点加入到等待队列的相关函数中涉及到了无锁队列的相关算法，由于在<font color ='red'> **AQS**</font>中只是将节点添加到队尾，使用到的无锁算法也相对简单。真正的无锁队列的算法我们等到分析<font color ='red'> **ConcurrentSkippedListMap**</font>时在进行讲解。

    private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        //先使用快速入列法来尝试一下,如果失败,则进行更加完备的入列算法.
        //只有在必要的情况下才会使用更加复杂耗时的算法，也就是乐观的态度
        Node pred = tail; //列尾指针
        if (pred != null) {
            node.prev = pred; //步骤1:该节点的前趋指针指向tail
            if (compareAndSetTail(pred, node)){ //步骤二:cas将尾指针指向该节点
                pred.next = node;//步骤三:如果成果,让旧列尾节点的next指针指向该节点.
                return node;
            }
        }
        //cas失败,或在pred == null时调用enq
        enq(node);
        return node;
    }
    private Node enq(final Node node) {
        for (;;) { //cas无锁算法的标准for循环,不停的尝试
            Node t = tail;
            if (t == null) { //初始化
                if (compareAndSetHead(new Node())) 
                //需要注意的是head是一个哨兵的作用,并不代表某个要获取锁的线程节点
                    tail = head;
            } else {
                //和addWaiter中一致,不过有了外侧的无限循环,不停的尝试,自旋锁
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

通过调用<font color ='red'> **addWaiter**</font>函数，<font color ='red'> **AQS**</font>将当前线程加入到了等待队列，但是还没有阻塞当前线程的执行，接下来我们就来分析一下<font color ='red'> **acquireQueued**</font>函数。

# 5.0  等待队列节点的操作
由于进入阻塞状态的操作会降低执行效率，所以，<font color ='red'> **AQS**</font>会尽力避免试图获取独占性变量的线程进入阻塞状态。所以，当线程加入等待队列之后，<font color ='red'> **acquireQueued**</font>会执行一个for循环，每次都判断当前节点是否应该获得这个变量(在队首了)。如果不应该获取或在再次尝试获取失败，那么就调用<font color ='red'> **shouldParkAfterFailedAcquire**</font>判断是否应该进入阻塞状态。如果当前节点之前的节点已经进入阻塞状态了，那么就可以判定当前节点不可能获取到锁，为了防止CPU不停的执行for循环，消耗CPU资源，调用<font color ='red'> **parkAndCheckInterrupt**</font>函数来进入阻塞状态。

    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) { //一直执行,直到获取锁,返回.
                final Node p = node.predecessor(); 
                //node的前驱是head,就说明,node是将要获取锁的下一个节点.
                if (p == head && tryAcquire(arg)) { //所以再次尝试获取独占性变量
                    setHead(node); //如果成果,那么就将自己设置为head
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                    //此时,还没有进入阻塞状态,所以直接返回false,表示不需要中断调用selfInterrupt函数
                }
                //判断是否要进入阻塞状态.如果`shouldParkAfterFailedAcquire`
                //返回true,表示需要进入阻塞
                //调用parkAndCheckInterrupt；否则表示还可以再次尝试获取锁,继续进行for循环
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    //调用parkAndCheckInterrupt进行阻塞,然后返回是否为中断状态
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL) //前一个节点在等待独占性变量释放的通知,所以,当前节点可以阻塞
            return true;
        if (ws > 0) { //前一个节点处于取消获取独占性变量的状态,所以,可以跳过去
            //返回false
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            //将上一个节点的状态设置为signal,返回false,
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this); //将AQS对象自己传入
        return Thread.interrupted();
    }

# 6.0  阻塞和中断

由上述分析，我们知道了<font color ='red'> **AQS**</font>通过调用<font color ='red'> **LockSupport**</font>的<font color ='red'> **park**</font>方法来执行阻塞当前进程的操作。其实，这里的阻塞就是线程不再执行的含义，通过调用这个函数，线程进入阻塞状态，上述的<font color ='red'> **lock**</font>操作也就阻塞了，等待中断或在独占性变量被释放。

    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);//设置阻塞对象,用来记录线程被谁阻塞的,用于线程监控和分析工具来定位
        UNSAFE.park(false, 0L);//让当前线程不再被线程调度,就是当前线程不再执行.
        setBlocker(t, null);
    }

关于中断的相关知识，我们以后再说，就继续沿着AQS的主线，看一下释放独占性变量的相关操作吧。

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/20/07/09/%E6%B5%81%E7%A8%8B2.webp)

ReentrantLock未获得阻塞,加入队列

# 7.0  unlock操作

与<font color ='red'>**lock**</font>操作类似，<font color ='red'> **unlock**</font>操作调用了<font color ='red'> **AQS**</font>的relase方法，参数和调用<font color ='red'> **acquire**</font>时一样，都是1。

    public final boolean release(int arg) {
        if (tryRelease(arg)) { 
        //释放独占性变量,起始就是将status的值减1,因为acquire时是加1
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);//唤醒head的后继节点
            return true;
        }
        return false;
    }

由上述代码可知，release就是先调用<font color ='red'> **tryRelease**</font>来释放独占性变量。如果成功，那么就看一下是否有等待锁的阻塞线程，如果有，就调用<font color ='red'> **unparkSuccessor**</font>来唤醒他们。

    protected final boolean tryRelease(int releases) {
        //由于只有一个线程可以获得独占先变量,所以,所有操作不需要考虑多线程
        int c = getState() - releases; 
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) { //如果等于0,那么说明锁应该被释放了,否则表示当前线程有多次lock操作.
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

我们可以看到<font color ='red'> **tryRelease**</font>中的逻辑也体现了可重入锁的概念，只有等到<font color ='red'> **state**</font>的值为0时，才代表锁真正被释放了。所以独占性变量<font color ='red'> **state**</font>的值就代表锁的有无。当<font color ='red'> **state**</font>=0时，表示锁未被占有，否在表示当前锁已经被占有。

    private void unparkSuccessor(Node node) {
        .....
        //一般来说,需要唤醒的线程就是head的下一个节点,但是如果它获取锁的操作被取消,或在节点为null时
        //就直接继续往后遍历,找到第一个未取消的后继节点.
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }

调用了<font color ='red'> **unpark**</font>方法后，进行<font color ='red'> **lock**</font>操作被阻塞的线程就恢复到运行状态,就会再次执行<font color ='red'> **acquireQueued**</font>中的无限for循环中的操作，再次尝试获取锁。

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/20/07/09/%E6%B5%81%E7%A8%8B3.webp)

ReentrantLock释放锁并通知阻塞线程恢复执行

# 8.0  后记

有关<font color ='red'> **AQS**</font>和<font color ='red'> **ReentrantLock**</font>的分析就差不多结束了。不得不说，我第一次看到<font color ='red'> **AQS**</font>的实现时真是震惊，以前都认为<font color ='red'> **Synchronized**</font>和<font color ='red'> **ReentrantLock**</font>的实现原理是一致的，都是依靠java虚拟机的功能实现的。没有想到还有<font color ='red'> **AQS**</font>这样一个背后大Boss在提供帮助啊。学习了这个类的原理，我们对JUC的很多类的分析就简单了很多。此外，<font color ='red'> **AQS**</font>涉及的<font color ='red'> **CAS**</font>操作和无锁队列的算法也为我们学习其他无锁算法提供了基础。


转载自： [https://www.cnblogs.com/sweetorangezzz/p/13184325.html](https://www.cnblogs.com/sweetorangezzz/p/13184325.html)
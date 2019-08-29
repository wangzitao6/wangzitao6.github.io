---
title:     "HashMap实现原理"  # 文章标题
subtitle:    "HashMap实现原理"  # 文章标题
description: ""
date:         2018-09-12T11:49:31+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["java","基础"]
url:    "/2019-09-12-hashmap实现原理/"
categories:  ["java"]
showtoc: true   # 是否显示目录
---
## 什么是哈希表

哈希表（hash table）也叫散列表，是一种非常重要的数据结构，应用场景及其丰富，许多缓存技术（比如memcached）的核心其实就是在内存中维护一张大的哈希表，而HashMap的实现原理也常常出现在各类的面试题中，重要性可见一斑。本文会对java集合框架中的对应实现HashMap的实现原理进行讲解，然后会对JDK7的HashMap源码进行分析。

* **数组**：采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)；通过给定值进行查找，需要遍历数组，逐一比对给定关键字和数组元素，时间复杂度为O(n)，当然，对于有序数组，则可采用二分查找，插值查找，斐波那契查找等方式，可将查找复杂度提高为O(logn)；对于一般的插入删除操作，涉及到数组元素的移动，其平均复杂度也为O(n)

* **线性链表**：对于链表的新增，删除等操作（在找到指定操作位置后），仅需处理结点间的引用即可，时间复杂度为O(1)，而查找操作需要遍历链表逐一进行比对，复杂度为O(n)

* **二叉树**：对一棵相对平衡的有序二叉树，对其进行插入，查找，删除等操作，平均复杂度均为O(logn)。

* **哈希表**：相比上述几种数据结构，在哈希表中进行添加，删除，查找等操作，性能十分之高，不考虑哈希冲突的情况下，仅需一次定位即可完成，时间复杂度为O(1)，接下来我们就来看看哈希表是如何实现达到惊艳的常数阶O(1)的。

我们知道，数据结构的物理存储结构只有两种：**<font color = "red">顺序存储结构</font>**和**<font color = "red">链式存储结构</font>**（像栈，队列，树，图等是从逻辑结构去抽象的，映射到内存中，也这两种物理组织形式），而在上面我们提到过，在数组中根据下标查找某个元素，一次定位就可以达到，哈希表利用了这种特性，哈希表的主干就是数组。

　　比如我们要新增或查找某个元素，我们通过把当前元素的关键字 通过某个函数映射到数组中的某个位置，通过数组下标一次定位就可完成操作。

　　　　　　　　**<font color = "red">存储位置 = f(关键字)</font>**

　　其中，这个函数f一般称为**哈希函数**，这个函数的设计好坏会直接影响到哈希表的优劣。举个例子，比如我们要在哈希表中执行插入操作：
![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/08/29/29-1.png)

查找操作同理，先通过哈希函数计算出实际存储地址，然后从数组中对应地址取出即可。

### 哈希冲突

　然而万事无完美，如果两个不同的元素，通过哈希函数得出的实际存储地址相同怎么办？也就是说，**当我们对某个元素进行哈希运算，得到一个存储地址，然后要进行插入的时候，发现已经被其他元素占用了，其实这就是所谓的哈希冲突，也叫哈希碰撞**。前面我们提到过，哈希函数的设计至关重要，好的哈希函数会尽可能地保证 计算简单和散列地址分布均匀,但是，我们需要清楚的是，数组是一块连续的固定长度的内存空间，再好的哈希函数也不能保证得到的存储地址绝对不发生冲突。那么哈希冲突如何解决呢？哈希冲突的解决方案有多种:**开放定址法**（发生冲突，继续寻找下一块未被占用的存储地址），**再散列函数法**，**链地址法**，而HashMap即是采用了链地址法，也就是**数组+链表**的方式，

## HashMap实现原理

HashMap的主干是一个Node数组。Node是HashMap的基本组成单元，每一个Node包含一个key-value键值对。

Node是HashMap中的一个静态内部类。代码如下
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash; //对key的hashcode值进行hash运算后得到的值，存储在Node，避免重复计算
    final K key;
    V value;
    Node<K,V> next;//存储指向下一个Node的引用，单链表结构

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
     /**省略此处代码**/
}
```

所以，HashMap的整体结构如下:

![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/08/29/29-7.png)

**简单来说，HashMap由数组+链表组成的，<font color = "red">数组是HashMap的主体</font>，<font color = "red">链表则是主要为了解决哈希冲突而存在的</font>，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，HashMap中的链表出现越少，性能才会越好**。(<font color = "red">jdk 8 之前，其内部是由数组+链表来实现的，而 jdk 8 对于链表长度超过 8 的链表将转储为红黑树</font>，本文讨论的是jdk8格式的。)

其他几个重要字段

```java
//默认的容量，即默认的数组长度 16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

//最大的容量，即数组可定义的最大长度 
static final int MAXIMUM_CAPACITY = 1 << 30;
```
这就是上述提到的数组，数组的元素都是 Node 类型，数组中的每个 Node 元素都是一个链表的头结点，通过它可以访问连接在其后面的所有结点。其实你也应该发现，上述的容量指的就是这个数组的长度。

```java
//HashMap的主干数组，可以看到就是一个Node数组，初始值为空数组{}
transient Node<K,V>[] table;

//实际存储的键值对个数
transient int size;

//用于迭代防止结构性破坏的标量，由于HashMap非线程安全，
//在对HashMap进行迭代时，如果期间其他线程的参与导致HashMap的结构发生变化了（比如put，remove等操作），
//需要抛出异常ConcurrentModificationException
transient int modCount;
```
下面这几个属性是相关的，threshold 代表的是一个阈值。伴随着元素不断的被添加进数组，一旦数组中的元素数量达到这个阈值，那么表明数组应该被扩容而不应该继续任由元素加入。而这个阈值的具体值则由负载因子（loadFactor）和数组容量来决定。
公式：``threshold = capacity * loadFactor``。
```java
//map中实际存储的key-value键值对的个数
transient int size;

//阈值，当table == {}时，该值为初始容量（初始容量默认为16）；
//当table被填充了，也就是为table分配内存空间后，threshold一般为 capacity*loadFactory。
//HashMap在进行扩容时需要参考threshold，后面会详细谈到
int threshold;

//负载因子，代表了table的填充度有多少，默认是0.75
final float loadFactor;

//HashMap 中默认负载因子为 0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```
好了，有关 HashMap 的基本属性大致介绍如上。下面我们看看它的几个重载的构造函数。HashMap有4个构造器，其他构造器如果用户没有传入``initialCapacity`` 和``loadFactor``这两个参数，会使用默认值

**``initialCapacity``默认为16，``loadFactory``默认为0.75**

```java
public HashMap(int initialCapacity, float loadFactor) {
    　//此处对传入的初始容量进行校验，最大范围 0 < MAXIMUM_CAPACITY <= 1<<30
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```
这是一个最基本的构造函数，需要调用方传入两个参数，``initialCapacity`` 和 ``loadFactor``。程序的大部分代码在判断传入参数的合法性，``initialCapacity`` 小于零将抛出异常，大于 ``MAXIMUM_CAPACITY`` 将被限定为 ``MAXIMUM_CAPACITY``。``loadFactor`` 如果小于等于零或者非数字类型也会抛出异常。

整个构造函数的核心在对 ``threshold``的初始化操作：

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```
由此可以看到，当在实例化HashMap实例时，如果给定了``initialCapacity``，由于HashMap的``capacity``都是2的幂，因此这个方法用于找到大于等于``initialCapacity``的最小的2的幂（``initialCapacity``如果就是2的幂，则返回的还是这个数）。 

下面分析这个算法： 
首先，为什么要对cap做减1操作。```int n = cap - 1;```
这是为了防止cap已经是2的幂。**如果cap已经是2的幂， 又没有执行这个减1操作，则执行完后面的几条无符号右移操作之后，返回的capacity将是这个cap的2倍**。如果不懂，要看完后面的几个无符号右移之后再回来看看。 
下面看看这几个无符号右移操作： 
如果n这时为0了（经过了cap-1之后），则经过后面的几次无符号右移依然是0，最后返回的capacity是1（最后有个n+1的操作）。 
这里只讨论n不等于0的情况。

第一次右移
```java
n |= n >>> 1;(相当于: n=n|n >>> 1)
```
由于n不等于0，则n的二进制表示中总会有一bit为1，这时考虑最高位的1。通过无符号右移1位，则将最高位的1右移了1位，再做或操作，使得n的二进制表示中与最高位的1紧邻的右边一位也为1，如:

```
n=000...01xxxxxxx
右移一位后为 n >>> 1 = 000...001xxxxxx  = 000...011xxxxxx
那么n|n >>> 1 = 000...01xxxxxxx|000...001xxxxxx  = 000...011xxxxxx
```

第二次右移

```java
n |= n >>> 2;
```
注意，这个n已经经过了n |= n >>> 1; 操作。假设此时n=16,二进制为000011xxxxxx ，则n无符号右移两位，会将最高位两个连续的1右移两位，然后再与原来的n做或操作，这样n的二进制表示的高位中会有4个连续的1。如000...01111xxxx 。 
第三次右移

```java
n |= n >>> 4;
```
这次把已经有的高位中的连续的4个1，右移4位，再做或操作，这样n的二进制表示的高位中会有8个连续的1。如000...011111111 。 

所以从宏观上看，传入的容量无论是处于任何范围，最终都会被打造成比该值大并且比最近的一个 2 的 n 次幂小一的值。为什么这么做？因为 2 的 n 次幂小一的值在二进制角度看全为 1，将有利于 HashMap 中的元素搜索，这一点我们后续将介绍。

**以此类推**
注意，容量最大也就是32bit的正数，因此最后``n |= n >>> 16;`` ，最多也就32个1，但是这时已经大于了``MAXIMUM_CAPACITY`` ，所以取值到``MAXIMUM_CAPACITY`` 。 
举一个例子说明下吧。 
![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/08/29/29-9.png)

这个算法着实牛逼啊！

那么通过该方法，我们将获得一个 2 的整数次幂的容量的值，此处存放至``threshold``，实际上我们获取的是一个有关数组容量的值，不应该存放至``阈值 threshold`` 中，但在后续实际初始化数组的时候并不会受到影响，这里可能是写 jdk 的大神偷了一次懒吧。

注意，得到的这个``capacity``却被赋值给了``阈值 threshold``。

```java
this.threshold = tableSizeFor(initialCapacity);
```
开始以为这个是个Bug，感觉应该这么写：

```java
this.threshold = tableSizeFor(initialCapacity) * this.loadFactor;
```
这样才符合threshold的意思（当HashMap的size到达threshold这个阈值时会扩容）。 
但是，请注意，在构造方法中，并没有对table这个成员变量进行初始化，table的初始化被推迟到了put方法中，在put方法中会对threshold重新计算

## put 方法的具体实现

put 方法的源码分析是本篇的一个重点，因为通过该方法我们可以窥探到 HashMap 在内部是如何进行数据存储的，所谓的数组+链表+红黑树的存储结构是如何形成的，又是在何种情况下将链表转换成红黑树来优化性能的。带着一系列的疑问，我们看这个 put 方法：

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

添加一个元素只需要传入一个键和一个值即可，putVal 方法是关键，我已经在该方法中进行了基本的注释，具体的细节稍后详细说明，先从这些注释中大体上建立一个直观的感受。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    //Node<K,V>[] tab, Node<K,V>数组
    //Node<K,V> p,   单链表中存储指向下一个Node的引用
    //n,主干数组的长度
    //i, index索引
    Node<K,V>[] tab; Node<K,V> p; int n, i;

    //如果 table 还未被初始化，那么初始化它
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;

    //根据键的 hash 值找到该键对应到数组中存储的索引,i = (n - 1) & hash
    //如果 tab[i]为 null，那么说明此索引位置并没有被占用,创建节点
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);

    //tab[i]不为 null，说明此处已经被占用，只需要将构建一个节点插入到这个链表的尾部即可
    else {
        Node<K,V> e; K k;

        //当前结点和将要插入的结点的 hash 和 key 相同，说明这是一次修改操作
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;

        //如果 p 这个头结点是红黑树结点的话，以红黑树的插入形式进行插入
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        
        // p 这个头结点属于链表，遍历此条链表，将构建一个节点插入到该链表的尾部
        else {
            for (int binCount = 0; ; ++binCount) {

                //在链表的最后插入
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);

                    //如果插入后链表长度大于等于 8 ，将链表裂变成红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }

                //遍历的过程中，如果发现与某个结点的 hash和key，这依然是一次修改操作 
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //e 不是 null，说明当前的 put 操作是一次修改操作并且e指向的就是需要被修改的结点
        if (e != null) { 
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //如果添加后，数组容量达到阈值，进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

从整体上来看，该方法的大致处理逻辑已如上述注释说明，下面我们针对其中的细节进行详细的解释。

首先，我们看 resize 这个方法是如何对 table 进行初始化的，代码比较多，分两部分进行解析：

```java
//第一部分
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;

        //拿到旧数组的长度，为0时初始化，其他时扩容
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        
        //说明旧数组已经被初始化完成了，此处需要给旧数组扩容
        if (oldCap > 0) {
            //极限的限定，达到容量限定的极限将不再扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //未达到极限，将数组容量扩大两倍，阈值也扩大两倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; 
        }
        //数组未初始化，但阈值不为 0，为什么不为 0 ？
        //上述提到 jdk 大神偷懒的事情就指的这，构造函数根据传入的容量打造了一个合适的数组容量暂存在阈值中
        //这里直接使用
        else if (oldThr > 0) 
            newCap = oldThr;
        //数组未初始化并且阈值也为0，说明一切都以默认值进行构造
        else {
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //这里也是在他偷懒的后续弥补
        //newCap = oldThr 之后并没有计算阈值，所以 newThr = 0
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
****************后续代码......****************
```
这一部分代码结束后，无论是初始化数组还是扩容，总之，必需的数组容量和阈值都已经计算完成了。下面看后续的代码：

```java
****************第二部分代码.....****************
//根据新的容量初始化一个数组
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
table = newTab;
//旧数组不为 null，这次的 resize 是一次扩容行为
if (oldTab != null) {
    //将旧数组中的每个节点位置相对静止地拷贝值新数组中
    for (int j = 0; j < oldCap; ++j) {
        Node<K,V> e;
        //获取头结点
        if ((e = oldTab[j]) != null) {
            oldTab[j] = null;
            //说明链表或者红黑树只有一个头结点，转移至新表
            if (e.next == null)
                newTab[e.hash & (newCap - 1)] = e;
            //如果 e 是红黑树结点，红黑树分裂，转移至新表
            else if (e instanceof TreeNode)
                ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
            //这部分是将链表中的各个节点原序地转移至新表中，我们后续会详细说明
            else { 
                Node<K,V> loHead = null, loTail = null;
                Node<K,V> hiHead = null, hiTail = null;
                Node<K,V> next;
                do {
                    next = e.next;
                    if ((e.hash & oldCap) == 0) {
                        if (loTail == null)
                            loHead = e;
                        else
                            loTail.next = e;
                        loTail = e;
                    }
                    else {
                        if (hiTail == null)
                            hiHead = e;
                        else
                            hiTail.next = e;
                        hiTail = e;
                    }
                } while ((e = next) != null);
                if (loTail != null) {
                    loTail.next = null;
                    newTab[j] = loHead;
                }
                if (hiTail != null) {
                    hiTail.next = null;
                newTab[j + oldCap] = hiHead;
                }
            }
        }
    }
}
//不论你是扩容还是初始化，都可以返回 newTab
return newTab;
```

对于第二部分的代码段来说，主要完成的是将旧链表中的各个节点按照原序地复制到新数组中。关于头结点是红黑树的情况我们暂时不去涉及，下面重点介绍下链表的拷贝和优化代码块，这部分代码不再重复贴出，此处直接进行分析，有需要的可以参照上述列出的代码块或者自己的 jdk 进行理解。

这部分其实是一个优化操作，将当前链表上的一些结点移出来向刚扩容的另一半存储空间放。

一般我们有如下公式：``index = e.hash & (oldCap - 1)``

```
例如：
e.hash = 010101110010101000101
oldCap = 　10000 
index = 01010111001010100 0101　& 1111 = 0101

newCap = 1000 00
newIndex = 0101011100101010 00101　& 11111 = 00101
```

随便举个例子，此时的 e 在容量扩大两倍以后的索引值没有变化，所以这部分结点是不需要移动的，那么程序如何判断扩容前后的 index 是否相等呢？

```java
//oldCap 一定是 100...000 的形式
if ((e.hash & oldCap) == 0)
```
如果原 oldCap 为 10000 的话，那么扩容后的 newCap 则为 100000，会比原来多出一位。所以我们只要知道原索引值的前一位是 0 还是 1 即可，如果是 0，那么它和新容量与后还是 0 并不改变索引的值，如果是 1 的话，那么索引值会增加 oldCap。

这样就分两步拆分当前链表，一条链表是不需要移动的，依然保存在当前索引值的结点上，另一条则需要变动到 index + oldCap 的索引位置上。

这里我们只介绍了普通链表的分裂情况，至于红黑树的裂变其实是类似的，依然分出一些结点到 index + oldCap 的索引位置上，只不过遍历的方式不同而已。

这样，我们对于 resize 这个扩容的方法已经解析完成了，下面接着看 putVal 方法，篇幅比较长，该方法的源码已经在介绍 resize 之前贴出，建议读者根据自己的 jdk 对照着理解。

上面我们说到，如果在 put 一个元素的时候判断内部的 table 数组还未初始化，那么调用 resize 根据相应的参数信息初始化数组。接下来的这个判断语句就很简单了：

```java
if ((p = tab[i = (n - 1) & hash]) == null)
   tab[i] = newNode(hash, key, value, null);
```
根据键的 hash 值找到对应的索引位置，如果该位置为 null，说明还没有头结点，于是 newNode 并存储在该位置上。

否则的话说明该位置已经有头结点了，或者说已经存在一个链表或红黑树了，那么我们要做的只是新建一个节点添加到链表或者红黑树的最后位置即可。

第一步，

```java
if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
      e = p;
```
p 指向当前节点，如果我们要插入的节点的键以及键所对应的 hash 值和 p 节点完全一样的话，那么说明这次 put 是一次修改操作，新建一个引用指向这个需要修改的节点。

第二步，

```java
else if (p instanceof TreeNode)
     e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
```
如果当前 p 节点是红黑树结点，那么需要调用不同于链表的的添加节点的方法来添加一个节点到红黑树中。（主要是维持平衡，建议读者去了解下红黑树，此处没有深谈是限于它的复杂度和文章篇幅）。

第三步，

```java
else {
     for (int binCount = 0; ; ++binCount) {
     if ((e = p.next) == null) {
         p.next = newNode(hash, key, value, null);
         if (binCount >= TREEIFY_THRESHOLD - 1) 
             treeifyBin(tab, hash);
         break;
     }
    if (e.hash == hash &&((k = e.key) == key || (key != null && key.equals(k))))
         break;
    p = e;
    }
}
```
这里主要处理的是向普通链表的末尾添加一个新的结点，e 不断地往后移动，如果发现 e 为 null，那么说明已经到链表的末尾了，那么新建一个节点添加到链表的末尾即可，因为 p 是 e 的父节点，所以直接让 p.next 指向新节点即可。添加之后，如果发现链表长度超过 8，那么将链表转储成红黑树。

在遍历的过程中，如果发现 e 所指向的当前结点和我们即将插入的节点信息完全匹配，那么也说明这是一次修改操作，由于 e 已经指向了该需要被修改的结点，所以直接 break 即可。

那么最终，无论是第一步中找到的头节点即需要被修改的节点，还是第三步在遍历中找到的需要被修改的节点，它们的引用都是 e，此时我们只需要用传入的 Value 值替换 e 指向的节点的 value 即可。正如这段代码一样：

```java
if (e != null) { // existing mapping for key
     V oldValue = e.value;
     if (!onlyIfAbsent || oldValue == null)
          e.value = value;
     afterNodeAccess(e);
     return oldValue;
 }
 ```
如果 e 为 null，那更简单了，说明此次 put 是添加新元素并且新元素也已经在上述代码中被添加到 HashMap 中了，我们只需要关心下，新加入一个元素后是否达到数组的阈值，如果是则调用 resize 方法扩大数组容量。该方法已经详细阐述过，此处不再赘述。

所以，这个 put 方法是集添加与修改一体的一个方法，如果执行的是添加操作则会返回 null，是修改操作则会返回旧结点的 value 值。

那么至此，我们对添加操作的内部实现想必已经了解的不错了，接下来看看删除操作的内部实现。

## remove 方法的具体实现

删除操作就是一个查找+删除的过程，相对于添加操作其实容易一些，但那是你基于上述添加方法理解的不错的前提下。

```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
```
根据键值删除指定节点，这是一个最常见的操作了。显然，removeNode 方法是核心。

```java
final Node<K,V> removeNode(int hash, Object key, Object value,boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||(value != null && value.equals(v)))) {
            if (node instanceof TreeNode)                                                                     ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```
删除操作需要保证在表不为空的情况下进行，并且 p 节点根据键的 hash 值对应到数组的索引，在该索引处必定有节点，如果为 null ，那么间接说明此键所对应的结点并不存在于整个 HashMap 中，这是不合法的，所以首先要在这两个大前提下才能进行删除结点的操作。

第一步，

```java
if (p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k))))
     node = p;
```
需要删除的结点就是这个头节点，让 node 引用指向它。否则说明待删除的结点在当前 p 所指向的头节点的链表或红黑树中，于是需要我们遍历查找。

第二步，

```java
else if ((e = p.next) != null) {
     if (p instanceof TreeNode)
          node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
     else {
         do {
              if (e.hash == hash &&((k = e.key) == key ||(key != null && key.equals(k)))) {
                     node = e;
              break;
         }
         p = e;
         } while ((e = e.next) != null);
     }
}
```
如果头节点是红黑树结点，那么调用红黑树自己的遍历方法去得到这个待删结点。否则就是普通链表，我们使用 do while 循环去遍历找到待删结点。找到节点之后，接下来就是删除操作了。

第三步，

```java
if (node != null && (!matchValue || (v = node.value) == value ||(value != null && value.equals(v)))) {
       if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
       else if (node == p)
            tab[index] = node.next;
       else
            p.next = node.next;
       ++modCount;
       --size;
       afterNodeRemoval(node);
       return node;
 }
 ```
删除操作也很简单，如果是红黑树结点的删除，直接调用红黑树的删除方法进行删除即可，如果是待删结点就是一个头节点，那么用它的 next 结点顶替它作为头节点存放在 table[index] 中，如果删除的是普通链表中的一个节点，用该结点的前一个节点直接跳过该待删结点指向它的 next 结点即可。

最后，如果 removeNode 方法删除成功将返回被删结点，否则返回 null。

这样，相对复杂的 put 和 remove 方法的内部实现，我们已经完成解析了。下面看看其他常用的方法实现，它们或多或少都于这两个方法有所关联。

## 其他常用的方法介绍

除了常用的 put 和 remove 两个方法外，HashMap 中还有一些好用的方法，下面我们简单的学习下它们。

### clear

```java
public void clear() {
    Node<K,V>[] tab;
    modCount++;
    if ((tab = table) != null && size > 0) {
        size = 0;
        for (int i = 0; i < tab.length; ++i)
            tab[i] = null;
    }
}
```
该方法调用结束后将清除 HashMap 中存储的所有元素。

### keySet

```java
//实例属性 keySet
transient volatile Set<K>        keySet;

public Set<K> keySet() {
    Set<K> ks;
    return (ks = keySet) == null ? (keySet = new KeySet()) : ks;
}
final class KeySet extends AbstractSet<K> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    public final Iterator<K> iterator()     { return new KeyIterator(); }
    public final boolean contains(Object o) { return containsKey(o); }
    public final boolean remove(Object key) {
        return removeNode(hash(key), key, null, false, true) != null;
    }
    public final Spliterator<K> spliterator() {
        return new KeySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
}
```
HashMap 中定义了一个 keySet 的实例属性，它保存的是整个 HashMap 中所有键的集合。上述所列出的 KeySet 类是 Set 的一个实现类，它负责为我们提供有关 HashMap 中所有对键的操作。

可以看到，KeySet 中的所有的实例方法都依赖当前的 HashMap 实例，也就是说，我们对返回的 keySet 集中的任意一个操作都会直接映射到当前 HashMap 实例中，例如你执行删除一个键的操作，那么 HashMap 中将会少一个节点。

### values

```java
public Collection<V> values() {
    Collection<V> vs;
    return (vs = values) == null ? (values = new Values()) : vs;
}
```
values 方法其实和 keySet 方法类似，它返回了所有节点的 value 属性所构成的 Collection 集合，此处不再赘述。

### entrySet

```java
public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
}
```
它返回的是所有节点的集合，或者说是所有的键值对集合。

### get

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```
get 方法的内部实现其实是我们介绍过的 put 方法中的一部分，所以此处也不再赘述。

参考文档：
[HashMap源码注解 之 静态工具方法hash()、tableSizeFor()](https://blog.csdn.net/fan2012huan/article/details/51097331)

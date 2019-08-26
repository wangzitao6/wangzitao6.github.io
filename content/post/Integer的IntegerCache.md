---
title:     "Integer的IntegerCache"  # 文章标题
subtitle:    "Integer的IntegerCache"  # 文章标题
description: ""
date:         2019-08-26T10:23:46+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["java","基础"]
url:    "/2019-08-26-integer的integerCache/"
categories:  ["java"]
showtoc: false   # 是否显示目录
---

首先我们来看这样一个例子:
```java
int m = 10;
int n = 10;
System.out.println(m == n);

int j = 128;
int k = 128;
System.out.println(j == k);

```

输出结果：

```java
true
true
```
出现这样的一个结果大家都意外。


下面我们再看一个例子：

```java
Integer a = 10;
Integer b = 10;
System.out.println(a == b);

Integer c = 128;
Integer d = 128;
System.out.println(c == d);

```

输出结果：

```java
true
false
```

这是因为，Integer是包装类型，是Object对象，因此==比较的是Integer指向的内存地址。然而-128~127直接的Integer数据直接缓存进入常量池（IntegerCache），所以这个区间的比较返回true，其他区间返回false。当然，new的Integer对象不适用。

看下Integer源码：

```java
public static Integer valueOf(String s) throws NumberFormatException {
    return Integer.valueOf(parseInt(s, 10));
}
```

Java 编译器把原始类型自动转换为封装类的过程称为自动装箱（autoboxing），这相当于调用 valueOf 方法。</br>
我们
Integer a = 10 或者 Integer c = 200 都进行了一次自动装箱。

```java
Integer a = 10;
//等价于：
Integer a = Integer.valueOf(10);

Integer c = 128
//等价于：
Integer a = Integer.valueOf(128);

```

Integer在赋值时自动装箱，调用valueOf(),其中valueOf源码如下：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

可以看到如果这个数值在cache数组的范围内（low和high之间），就返回cache缓存数组的中的数据，否则产生一个新的Integer值。其IntegerCache源码如下：

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}

```

通常这个范围是-128-127，然而这个范围的最大值是可变的，可以通过-XX:AutoBoxCacheMax=<size>参数去修改这个值，在JVM初始化的时候，这个值被写入sun.misc.VM class系统私有配置文件中，并加载。

所以这就是为什么在阿里巴巴Java开发手册的OOP规约中强制使用equals比较两个Integer的值

>4.7【强制】所有的相同类型的包装类对象之间值的比较，全部使用 equals 方法比较。
说明： 对于 Integer var=?在-128 至 127 之间的赋值， Integer 对象是在
IntegerCache.cache 产生，会复用已有对象，这个区间内的 Integer 值可以直接使用==进行
判断。
但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑，
推荐使用 equals 方法进行判断


这种缓存行为不仅适用于Integer对象。我们针对所有整数类型的类都有类似的缓存机制。
有 ByteCache 用于缓存 Byte 对象
有 ShortCache 用于缓存 Short 对象
有 LongCache 用于缓存 Long 对象
有 CharacterCache 用于缓存 Character 对象
Byte，Short，Long 有固定范围: -128 到 127。对于 Character, 范围是 0 到 127。除了 Integer 可以通过参数改变范围外，其它的都不行。





---
title:     "Java中静态代码块、代码块、main()方法和构造函数加载顺序"  # 文章标题
subtitle:    "Java中静态代码块、代码块、main()方法和构造函数加载顺序"  # 文章标题
description: ""
date:         2018-07-02T10:38:28+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["java","jvm"]
url:    "/2018-07-02-java中静态代码块、代码块、main()方法和构造函数加载顺序/"
categories:  ["jvm"]
showtoc: true   # 是否显示目录
---

>静态代码块：用staitc声明，jvm加载类时执行，仅执行一次。</br>
>构造代码块：类中直接用{}定义，每一次创建对象时执行。</br>
>同一个类中，执行顺序优先级：**静态代码块 > main() > 构造代码块 > 构造函数**。
   
## 构造函数 

```java
//构造函数
public ClassLoadDemo(){
    System.out.println("这里是构造函数");
}
```
注意：</br>
1. 对象一建立，就会调用与之相应的构造函数，也就是说，不建立对象，构造函数时不会运行的。</br>
2. **构造函数的作用是用于给对象进行初始化**。</br>
3. 一个对象建立，构造函数只运行一次，而一般方法可以被该对象调用多次。

## 构造代码块

```java
//构造代码块
{
    System.out.println("这里是构造代码块");
}
```

注意：</br>
1. **构造代码块的作用是给对象进行初始化**。</br>
2. **对象一建立就运行构造代码块了，而且优先于构造函数执行**。这里要强调一下，有对象建立，才会运行构造代码块，类不能调用构造代码块的。</br>
3. 构造代码块与构造函数的区别是：**构造代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化**，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，**构造代码块中定义的是不同对象共性的初始化内容，所以构造代码块优先于构造函数执行**。

## 静态代码块

```java
//静态代码块
public ClassLoadDemo(){
    System.out.println("这里是构造函数");
}
```
注意：</br>
1. 它是随着类的加载而执行，只执行一次，并优先于主函数。具体说，静态代码块是由类调用的。类调用时，先执行静态代码块，然后才执行主函数的。</br>
2. 静态代码块其实就是给类初始化的，而构造代码块是给对象初始化的。</br>
3. 静态代码块中的变量是局部变量，与普通函数中的局部变量性质没有区别。</br>
4. 一个类中可以有多个静态代码块。</br>
   
```java
public class ClassLoadDemo {
    private static int num = 1;

    //静态代码块
    static{
        num = num + 6;
    }
    //main()
    public static void main(String[] args) {
        System.out.println("num: "+ num);
    }
    //静态代码块
    static{
        num = num - 5;
    }
}

输出结果： num: 2
```

## 类初始化顺序

demo1:

```java
public class ClassA {

    //构造函数
    public ClassA(){
        System.out.println("这里是ClassA构造函数");
    }
    //静态代码块
    static{
        System.out.println("这里是ClassA静态代码块");
    }
    //构造代码块
    {
        System.out.println("这里是ClassA构造代码块");
    }
    //main()
    public static void main(String[] args) {
        
    }
}

运行结果: 这里是ClassA静态代码块
```

demo2:对ClassA对象进行初始化。
```java
public class ClassA {

    //构造函数
    public ClassA(){
        System.out.println("这里是ClassA构造函数");
    }
    //静态代码块
    static{
        System.out.println("这里是ClassA静态代码块");
    }
    //构造代码块
    {
        System.out.println("这里是ClassA构造代码块");
    }
    //main()
    public static void main(String[] args) {
        ClassA a = new ClassA();
    }
}


运行结果: 
这里是ClassA静态代码块
这里是ClassA构造代码块
这里是ClassA构造函数
```

demo3:对ClassA对象进行初始化两次。
```java
public class ClassA {

    //构造函数
    public ClassA(){
        System.out.println("这里是ClassA构造函数");
    }
    //静态代码块
    static{
        System.out.println("这里是ClassA静态代码块");
    }
    //构造代码块
    {
        System.out.println("这里是ClassA构造代码块");
    }
    //main()
    public static void main(String[] args) {
        ClassA a = new ClassA();
        ClassA b = new ClassA();
    }
}

运行结果：
这里是ClassA静态代码块
这里是ClassA构造代码块
这里是ClassA构造函数
这里是ClassA构造代码块
这里是ClassA构造函数
```

>对于一个类而言，按照如下顺序执行：

>1. 执行静态代码块
>2. 执行构造代码块
>3. 执行构造函数

>对于静态变量、静态初始化块、变量、初始化块、构造器，它们的初始化顺序依次是（静态变量、静态初始化块）>（变量、初始化块）>构造器。


demo4: 加入静态变量、静态对ClassA进行初始化
```java
public class ClassA {
    /* 静态变量 */
    public static String staticField = "静态变量";
    /* 变量 */
    public String field = "变量";

    //构造函数
    public ClassA() {
        System.out.println("这里是ClassA构造函数");
    }

    //静态代码块
    static {
        System.out.println( staticField );
        System.out.println("这里是ClassA静态代码块");
    }

    //构造代码块
    {
        System.out.println( field);
        System.out.println("这里是ClassA构造代码块");
    }

    //main()
    public static void main(String[] args) {
        ClassA a = new ClassA();
    }
}

运行结果:

静态变量
这里是ClassA静态代码块
变量
这里是ClassA构造代码块
这里是ClassA构造函数
```

## 继承下的类初始化顺序

上面我们说的是单独Class下的加载顺序，下面我们讨论下继承下的加载顺序。

demo5:
```java
public class ClassA {

    //构造函数
    public ClassA() {
        System.out.println("这里是ClassA构造函数");
    }
    //静态代码块
    static {
        System.out.println("这里是ClassA静态代码块");
    }
    //构造代码块
    {
        System.out.println("这里是ClassA构造代码块");
    }
}

public class ClassB extends ClassA{

    //构造函数
    public ClassB(){
        System.out.println("这里是ClassB构造函数");
    }
    //静态代码块
    static{
        System.out.println("这里是ClassB静态代码块");
    }
    //构造代码块
    {
        System.out.println("这里是ClassB代码块");
    }
    //main()
    public static void main(String[] args) {
        ClassB b = new ClassB();
    }
}

运行ClassB结果：

这里是ClassA静态代码块
这里是ClassB静态代码块
这里是ClassA构造代码块
这里是ClassA构造函数
这里是ClassB代码块
这里是ClassB构造函数
```
注意：</br>
当涉及到继承时，按照如下顺序执行：</br>
1. 执行父类的静态代码块，并初始化父类静态成员变量</br>
2. 执行子类的静态代码块，并初始化子类静态成员变量</br>
3. 执行父类的构造代码块，执行父类的构造函数，并初始化父类普通成员变量</br>
4. 执行子类的构造代码块， 执行子类的构造函数，并初始化子类普通成员变量

Java初始化顺序如图：
![](https://wangzitao-blog.oss-cn-hangzhou.aliyuncs.com/19/09/01/2.png)

```java
public class ClassA {
    /* 静态变量 */
    public static String p_StaticField = "父类--静态变量";
    /* 变量 */
    public String    p_Field = "父类--变量";
    protected int    i    = 9;
    protected int    j    = 0;

    //构造函数
    public ClassA() {
        System.out.println("这里是父类构造函数");
        System.out.println( "i=" + i + ", j=" + j );
        j = 20;
    }
    //静态代码块
    static {
        System.out.println( p_StaticField );
        System.out.println("这里是父类静态代码块");
    }
    //构造代码块
    {
        System.out.println( p_Field);
        System.out.println("这里是父类构造代码块");
    }
}

public class ClassB extends ClassA{
    /* 静态变量 */
    public static String s_StaticField = "子类--静态变量";
    /* 变量 */
    public String s_Field = "子类--变量";

    //构造函数
    public ClassB(){
        System.out.println("这里是子类构造函数");
        System.out.println( "i=" + i + ",j=" + j );
    }
    //静态代码块
    static{
        System.out.println( s_StaticField );
        System.out.println("这里是子类静态代码块");
    }
    //构造代码块
    {
        System.out.println( s_Field );
        System.out.println("这里是子类代码块");
    }
    //main()
    public static void main(String[] args) {
        System.out.println( "子类main方法" );
        ClassB b = new ClassB();
    }
}

运行结果：
父类--静态变量
这里是父类静态代码块
子类--静态变量
这里是子类静态代码块
子类main方法
父类--变量
这里是父类构造代码块
这里是父类构造函数
i=9, j=0
子类--变量
这里是子类代码块
这里是子类构造函数
i=9,j=20
```

子类的静态变量和静态初始化块的初始化是在父类的变量、初始化块和构造器初始化之前就完成了。静态变量、静态初始化块，变量、初始化块初始化了顺序取决于它们在类中出现的先后顺序。

### 分析
1. 运行ClassB.main(),(这是一个static方法)，于是装载器就会为你寻找已经编译的ClassB类的代码（也就是ClassB.class文件）。在装载的过程中，装载器注意到它有一个基类（也就是extends所要表示的意思），于是它再装载基类。不管你创不创建基类对象，这个过程总会发生。如果基类还有基类，那么第二个基类也会被装载，依此类推。

2. 执行根基类的static初始化，然后是下一个派生类的static初始化，依此类推。这个顺序非常重要，因为派生类的“static初始化”有可能要依赖基类成员的正确初始化。

3. 当所有必要的类都已经装载结束，开始执行main()方法体，并用new ClassB() 创建对象。

4. 类ClassB存在父类，则调用父类的构造函数，你可以使用super来指定调用哪个构造函数。基类的构造过程以及构造顺序，同派生类的相同。首先基类中各个变量按照字面顺序进行初始化，然后执行基类的构造函数的其余部分。

5. 对子类成员数据按照它们声明的顺序初始化，执行子类构造函数的其余部分。




   

+++
layout= "post"
title = "Markdown常用语法"  # 文章标题
subtitle=  "" # 副标题
date = 2017-10-28T22:59:30+08:00  # 自动添加日期信息
tags = [" 其他"]  # 文章标签，可设置多个，用逗号隔开。Hugo会自动生成标签的子URL
author = "WangZiTao"
URL= "/2017/10/28/"
image=  "https://img.zhaohuabing.com/post-bg-2015.jpg"  # 单独图片
+++
#内容目录
在段落中填写 `[TOC]` 以显示全文内容的目录结构。
[TOC]

## 1.标题
```
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```
---
## 2.字体

- **加粗**
要加粗的文字左右分别用两个*号包起来
- **斜体**
要倾斜的文字左右分别用一个*号包起来
- **斜体加粗**
要倾斜和加粗的文字左右分别用三个*号包起来
- **删除线**
要加删除线的文字左右分别用两个~~号包起来

示例：
```
**这是加粗的文字**
*这是倾斜的文字*`
***这是斜体加粗的文字***
~~这是加删除线的文字~~
```
效果:

**这是加粗的文字**
*这是倾斜的文字*`
***这是斜体加粗的文字***
~~这是加删除线的文字~~
******
## 3.引用
在引用的文字前加>即可。引用也可以嵌套，如加两个>>三个>>>
```
>这是引用的内容
>>这是引用的内容
>>>>>>>>>>这是引用的内容
```
效果:
>这是引用的内容
>>这是引用的内容
>>>>>>>>>>这是引用的内容
---

## 4.分割线
三个或者三个以上的 - 或者 *
示例：
```
---
----
***
*****
```
效果如下：

---
----
***
*****



## 5.图片
```
网络图片
![bb](https://pic3.zhimg.com/50/v2-d570c35ffedc064cc12f6aa87275f5f7_hd.jpg)
```

效果:
![bb](https://pic3.zhimg.com/50/v2-d570c35ffedc064cc12f6aa87275f5f7_hd.jpg)


------

## 6.超链接
```
[超链接名](超链接地址 "超链接title")
title可加可不加

https://wangzitao6.github.io/ - 自动生成！
[博客](https://wangzitao6.github.io/,"超链接title")
```
效果:
https://wangzitao6.github.io/ - 自动生成！
[博客](https://wangzitao6.github.io/,"超链接title")

## 7.列表


### 7.1 无序列表


语法：
无序列表用 - + * 任意一种
```
- 列表内容
+ 列表内容
* 列表内容
```

注意：- + * 跟内容之间都要有一个空格

效果如下：
- 列表内容
+ 列表内容
* 列表内容



### 7.2 有序列表


语法：
数字加点
```
1. 列表内容
2. 列表内容
3. 列表内容
```

注意：序号跟内容之间要有空格

效果如下：
1. 列表内容
2. 列表内容
3. 列表内容


### 7.3 列表嵌套

下一级比上一级多敲两个空格

```

- 一级无序列表内容
  - 二级有序列表内容
    - 三级有序列表内容
        - 四级有序列表内容
            - 五级有序列表内容

```



- 一级无序列表内容
  - 二级有序列表内容
    - 三级有序列表内容
        - 四级有序列表内容
            - 五级有序列表内容




```
1. 一级有序列表内容

    2. 二级无序列表内容

       3. 二级无序列表内容

          4. 二级无序列表内容
```

1. 一级有序列表内容

    2. 二级无序列表内容

       3. 三级无序列表内容

          4. 四级无序列表内容





----

## 8.语法

```
无序列表用 - + * 任何一种都可以
- 列表内容
+ 列表内容
* 列表内容
注意：- + * 跟内容之间都要有一个空格
```
效果:

- 列表内容
+ 列表内容
* 列表内容

语法：
数字加点

```
1.列表内容
2.列表内容
3.列表内容

注意：序号跟内容之间要有空格
```
1.列表内容
2.列表内容
3.列表内容

```
表头|表头|表头
---|:--:|---:
内容|内容|内容
内容|内容|内容

第二行分割表头和内容。
- 有一个就行，为了对齐，多加了几个
文字默认居左
-两边加：表示文字居中
-右边加：表示文字居右
注：原生的语法两边都要用 | 包起来。此处省略
```
---
<br>
### 9. 行内代码块

使用 \`代码` 表示行内代码块。

示例：

让我们聊聊 `html`。

----

## 10.代码块
使用 \```代码``` 表示行内代码块。

### 10.1 代码块
```
    ```
    public static void main(String[] args) {
        System.out.println("hello");
        System.out.println("world");
        System.out.println("!");
    }
    ```
```
效果：

```
public static void main(String[] args) {
    System.out.println("hello");
    System.out.println("world");
    System.out.println("!");
}
```
### 10.2 代码块语言高亮
在代码块标识'''后面添加语言名java
```
    ```java
    public static void main(String[] args) {
        System.out.println("hello");
        System.out.println("world");
        System.out.println("!");
    }
    ```
```
效果：

```java
public static void main(String[] args) {
    System.out.println("hello");
    System.out.println("world");
    System.out.println("!");
}
```

### 10.3 代码块显示行数
在代码块标识'''java后面添加{.line-numbers}
```
    ```java {.line-numbers}
    public static void main(String[] args) {
        System.out.println("hello");
        System.out.println("world");
        System.out.println("!");
    }
    ```
```
效果：
```java {.line-numbers}
public static void main(String[] args) {
    System.out.println("hello");
    System.out.println("world");
    System.out.println("!");
}
```
### 10.4 代码块高亮代码
在代码块标识'''java后面添加{.line-numbers highlight=2-3}
<!-- 代码块内语法高亮  显示代码行数,高亮代码-->
```
    ```java {.line-numbers highlight=2-3}
    public static void main(String[] args) {
        System.out.println("hello");
        System.out.println("world");
        System.out.println("!");
    }
    ```
```
效果：
```java {.line-numbers highlight=2-3}
public static void main(String[] args) {
    System.out.println("hello");
    System.out.println("world");
    System.out.println("!");
}
```
----
<br>
## 11.表格

```
姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
```
效果:

姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
----
<br>

## 12.流程图

```
  流程图
  ```flow
  st=>start: 开始
  op=>operation: My Operation
  cond=>condition: Yes or No?
  e=>end
  st->op->cond
  cond(yes)->e
  cond(no)->op
  &```
```
```flow
st=>start: Start:>https://www.zybuluo.com
io=>inputoutput: verification
op=>operation: Your Operation
cond=>condition: Yes or No?
sub=>subroutine: Your Subroutine
e=>end

st->io->op->cond
cond(yes)->e
cond(no)->sub->io
```

-------

## 13. 待办事宜 Todo 列表
```
- [ ] ** Todo 列表**
       - [ ] one
       - [ ] two
       - [x] three
       - [x] four
           - [x] Todo事件
           - [x] Todo事件
   - [ ] **Todo 列表2**
       - [ ] Todo事件
       - [ ] Todo事件
       - [x] Todo事件
```

- [ ] ** Todo 列表**
  - [ ] one
  - [ ] two
  - [x] three
  - [x] four
- [ ] **Todo 列表2**
   - [ ] Todo事件
   - [ ] Todo事件
   - [x] Todo事件

----
## 14. Html 标签

本站支持在 Markdown 语法中嵌套 Html 标签，譬如，你可以用 Html 写一个纵跨两行的表格：
```
    <table>
        <tr>
            <th rowspan="2">值班人员</th>
            <th>星期一</th>
            <th>星期二</th>
            <th>星期三</th>
        </tr>
        <tr>
            <td>李强</td>
            <td>张明</td>
            <td>王平</td>
        </tr>
    </table>
```
效果：

<table>
    <tr>
        <th rowspan="2">值班人员</th>
        <th>星期一</th>
        <th>星期二</th>
        <th>星期三</th>
    </tr>
    <tr>
        <td>李强</td>
        <td>张明</td>
        <td>王平</td>
    </tr>
</table>

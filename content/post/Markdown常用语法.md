---
title:       "markdown常用语法"
subtitle:    "Markdown"
description: "Markdown的一些简单语法笔记"
date:         2017-10-28T22:59:30+08:00
author:   "WangZiTao"
image:   "https://img.zhaohuabing.com/post-bg-2015.jpg"
tags:        ["其他"]
url:    "/2017-10-28-markdown常用语法/"
categories:  ["other" ]
---

## 1.内容目录
在段落中填写 `[TOC]` 以显示全文内容的目录结构。

## 2.标题
```
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```
---
## 3.字体

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

## 4.引用

在引用的文字前加>即可。引用也可以嵌套，如加两个>>三个>>>

  ```
  >这是引用的内容
  >>这是引用的内容
  >>>>>>>>>>这是引用的内容
  ```

效果:

  >这是引用的内容

  >>这是引用的内容

  >>>>这是引用的内容

----

## 5.分割线
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



## 6.图片
```
网络图片
![bb](https://edu-image.nosdn.127.net/C0124E0336721FF65563B76A16A8143F.png?imageView&thumbnail=190y28&quality=100)
```

效果:
![bb](https://edu-image.nosdn.127.net/C0124E0336721FF65563B76A16A8143F.png?imageView&thumbnail=190y28&quality=100)


------

## 7.超链接
```
[超链接名](超链接地址 "超链接title")
title可加可不加

https://wangzitao6.github.io/ - 自动生成！
[博客](https://wangzitao6.github.io/,"超链接title")
```
效果:
https://wangzitao6.github.io/ - 自动生成！

[博客](https://wangzitao6.github.io/,"超链接title")

## 8.列表


### 8.1 无序列表


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



### 8.2 有序列表


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


### 8.3 列表嵌套

下一级比上一级多敲两个空格

  ```

  - 一级无序列表内容
    - 二级有序列表内容
      - 三级有序列表内容
          - 四级有序列表内容
  ```



  - 一级无序列表内容
    - 二级有序列表内容
      - 三级有序列表内容
          - 四级有序列表内容




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

## 9.语法

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
## 10. 行内代码块

使用 \`代码` 表示行内代码块。

示例：

让我们聊聊 `html`。

----

##  11.代码块

使用 \```代码``` 表示行内代码块。

### 11.1 代码块语言高亮
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

### 11.2 代码块显示行数
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
### 11.3 代码块高亮代码
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

## 12.表格

### 12.1 单元格和表头
使用 | 来分隔不同的单元格，使用 - 来分隔表头和其他行：

 示例:

  ```
  name | age
  ---- | ---
  LearnShare | 12
  Mike |  32
  ```

效果:

name | age
---- | ---
LearnShare | 12
Mike |  32

为了美观，可以使用空格对齐不同行的单元格，并在左右两侧都使用 | 来标记单元格边界：
  ```
  |    name    | age |
  | ---------- | --- |
  | LearnShare |  12 |
  | Mike       |  32 |
  ```

|    name    | age |
| ---------- | --- |
| LearnShare |  12 |
| Mike       |  32 |

> 为了使 Markdown 更清晰，| 和 - 两侧需要至少有一个空格（最左侧和最右侧的 | 外就不需要了）。

### 12.2 对齐
在表头下方的分隔线标记中加入 :，即可标记下方单元格内容的对齐方式：

- **:---** 代表左对齐
- **:--:** 代表居中对齐
- **---:**  代表右对齐

  ```
  | left | center | right |
  | :--- | :----: | ----: |
  | aaaa | bbbbbb | ccccc |
  | a    | b      | c     |

  ```
  效果:
  | left | center | right |
  | :--- | :----: | ----: |
  | aaaa | bbbbbb | ccccc |
  | a    | b      | c     |

  > 如果不使用对齐标记，单元格中的内容默认左对齐；表头单元格中的内容会一直居中对齐（不同的实现可能会有不同表现）。

### 12.3 插入其他内容
表格中可以插入其他 Markdown 中的行内标记：

  ```
  |     name     | age |             blog                |
  | ------------ | --- | ------------------------------- |
  | _LearnShare_ |  12 | [LearnShare](http://xianbai.me) |
  | __Mike__     |  32 | [Mike](http://mike.me)          |

  ```
  效果:
|     name     | age |             blog                |
| ------------ | --- | ------------------------------- |
| _LearnShare_ |  12 | [LearnShare](http://xianbai.me) |
| __Mike__     |  32 | [Mike](http://mike.me)          |

----

<br>

## 13.流程图

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

## 14. 待办事宜 Todo 列表
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
## 15. Html 标签

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

## 16.注释
### 16.1 html语法

既然支持html语法，那也支持html注释。

  ```
  <!--哈哈我是注释，不会在浏览器中显示。-->

  <!--
  哈哈我是多段
  注释，
  不会在浏览器中显示。
  -->
  ```

### 16.2 hack方法

hack方法就是利用markdown的解析原理来实现注释的。

一般有的markdown解析器不支持上面的注释方法，这个时候就可以用hack方法。

hack方法比上面2种方法稳定得多，但是语义化太差。

    [comment]: <> (哈哈我是注释，不会在浏览器中显示。)
    [comment]: <> (哈哈我是注释，不会在浏览器中显示。)
    [comment]: <> (哈哈我是注释，不会在浏览器中显示。)
    [//]: <> (哈哈我是注释，不会在浏览器中显示。)
    [//]: # (哈哈我是注释，不会在浏览器中显示。)


其中，这种方法最稳定，适用性最强：

```[//]: # (哈哈我是注释，不会在浏览器中显示。)```


这种方法最可爱，超级无敌萌啊：

```[^_^]: # (哈哈我是注释，不会在浏览器中显示。)```

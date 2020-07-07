---
title:     "Mysql自定义变量和结束分隔符"  # 文章标题
subtitle:    "Mysql自定义变量和结束分隔符"  # 文章标题
description: ""
date:         2020-07-07T09:23:19+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["MySQL"]
url:    "/2020-07-06-mysql自定义变量和结束分隔符"
categories:  ["sql"]
showtoc: true   # 是否显示目录
---

# 1. 自定义变量

## 1-1. SET方式赋值
自定义变量前边必须加一个 <font color = "red">@</font> 符号，环境变量赋值用<font color = "red">SET</font>,查询变量时用<font color = "red"> SELECT,不过仍然需要在变量名称前加一个@符号</font>。
```sql
mysql> set @a = '1';
Query OK, 0 rows affected (0.00 sec)

mysql> select @a;
+------+
| @a   |
+------+
| 1    |
+------+
1 row in set (0.00 sec)
```
## 1-2. 环境变量之间相互赋值
环境变量也可以相互之间赋值,比如把a的值赋值给b。
```sql
mysql> set @b = @a;
Query OK, 0 rows affected (0.00 sec)

mysql> select @b;
+------+
| @b   |
+------+
| 1    |
+------+
1 row in set (0.00 sec)
```

## 1-3. 根据查询结果赋值
我们还可以将某个查询的结果赋值给一个变量，<font color = "red">前提是这个查询的结果只有一个值：</font>
```sql
mysql> set @a =(select age from a where id =1);
Query OK, 0 rows affected (0.00 sec)

mysql> select @a;
+------+
| @a   |
+------+
|   11 |
+------+
1 row in set (0.00 sec)
```
否则会报错：
```sql
mysql> set @a =(select age from a where age ='11');
ERROR 1242 (21000):
```
## 1-4. 使用INTO赋值
还可以用另一种形式的语句来将查询的结果赋值给一个变量,为查询语句加上<font color = "red">limit</font> 可以确保只有一个返回结果。
```sql
mysql> select age from a where age = '11' limit 1 into @b;
Query OK, 1 row affected (0.00 sec)

mysql> select @b;
+------+
| @b   |
+------+
|   11 |
+------+
1 row in set (0.00 sec)
```

# 2. 语句结束分隔符

在MySQL客户端的交互界面处，当我们完成键盘输入并按下回车键时，MySQL客户端会检测我们输入的内容中是否包含 <font color = "red"> **;**</font> 、<font color = "red"> **\g** </font>或者<font color = "red"> **\G**</font>这三个符号之一，如果有的话，会把我们输入的内容发送到服务器。这样一来，如果我们想一次性给服务器发送多条的话，就需要把这些语句写到一行中，比如这样：

```sql
mysql> select age from a; select id from a;
+-----+
| age |
+-----+
|  11 |
|  15 |
|  11 |
+-----+
3 rows in set (0.00 sec)

+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.00 sec)


--------------------------------------
mysql>  select age from a\g select id from a \g
+-----+
| age |
+-----+
|  11 |
|  15 |
|  11 |
+-----+
3 rows in set (0.00 sec)

+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.00 sec)
```


<font color = "red">delimiter $ </font>命令意味着修改语句结束分隔符为<font color = "red"> $</font>，也就是说之后MySQL客户端检测用户语句输入结束的符号为<font color = "red"> $ </font>。上边例子中我们虽然连续输入了3个以分号;结尾的查询语句并且按了回车键，但是输入的内容并没有被提交，直到敲下<font color = "red"> $ </font>符号并回车，MySQL客户端才会将我们输入的内容提交到服务器，此时我们输入的内容里已经包含了2个独立的查询语句了，所以返回了2个结果集。

```sql
mysql> delimiter $
mysql> select age from a ;
    -> select id from a ;
    -> $
+-----+
| age |
+-----+
|  11 |
|  15 |
|  11 |
+-----+
3 rows in set (0.00 sec)

+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.00 sec)
```

我们也可以将语句结束分隔符重新 <font color = "red">  **定义为$以外的其他包含单个或多个字符的字符串**</font>，比方说这样 把<font color = "red"> $ </font>改为<font color = "red"> end</font> ：

```sql
mysql> delimiter end
mysql> select id from a ;
    -> select age from a ;
    -> end
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
+----+
3 rows in set (0.00 sec)

+-----+
| age |
+-----+
|  11 |
|  15 |
|  11 |
+-----+
3 rows in set (0.00 sec)
```
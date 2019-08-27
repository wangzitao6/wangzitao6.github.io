---
title:     "Mysql内置条件判断函数"  # 文章标题
subtitle:    "Mysql内置条件判断函数"  # 文章标题
description: ""
date:         2019-03-16T11:33:44+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["MySQL"]
url:    "/2018-03-16-mysql内置条件判断函数"
categories:  ["sql"]
showtoc: true   # 是否显示目录
---

## 常见的条件判断函数

|Name| Description|
|----| ----|
|CASE|	Case operator|
|IF()|	If/else construct|
|IFNULL()|	Null if/else construct|
|NULLIF(expr1,expr2)|	Return NULL if expr1 = expr2|



### CASE

为了后面容易举例子，我们先创建一张表并插入数据

```SQL
CREATE TABLE demo ( 
id INT,
NAME VARCHAR ( 20 ),
age INT
) ENGINE = INNODB;

insert into demo values (1,"小明",13), (1,"小红",28), (1,"老李",61);
```

case 包含两种表达格式：

 **<font color = "red">CASE value WHEN [compare_value] THEN result [WHEN [compare_value] THEN result ...] [ELSE result] END</font>**</br>
 规则：当查询的结果当value = compare_value时返回result。
 
 ```sql
 SELECT 
	NAME,
CASE
	NAME 
		WHEN '小明' THEN '少年' 
		WHEN '小红' THEN '中年' 
		WHEN '老李' THEN '老年' 
	END 'Type' 
FROM
	demo;
 ```

 结果：

| NAME|Type|
|-----|----|
|小明|少年|
|小红|中年|
|老李|老年|


 **<font color = "red">CASE WHEN [condition] THEN result [WHEN [condition] THEN result ...] [ELSE result] END</font>**</br>
规则：当condition为TRUE时返回result,当结果不符合时，返回else后结果，当没有else部分时，为NULL。

```sql
SELECT 
	NAME,
CASE 
		WHEN age < 18 THEN '少年' 
		WHEN age >= 25 AND age < 50 THEN '中年' 
		WHEN age >= 50 THEN '老年' 
		ELSE '其他'
	END 'Type' 
FROM
	demo;
```
 结果：

| NAME|Type|
|-----|----|
|小明|少年|
|小红|中年|
|老李|老年|

### IF()

IF(expr1,expr2,expr3), 当expr1为TRUE (expr1 <> 0 and expr1 <> NULL)时, IF() 返回 expr2. 否则, 结果为 expr3.

```sql
mysql> SELECT IF(1>2,2,3);
        -> 3
mysql> SELECT IF(1<2,'yes','no');
        -> 'yes'
```

### IFNULL()

IFNULL(expr1,expr2),如果 expr1 L不为空时, IFNULL() 返回 expr1; 否则返回 expr2.

```sql
mysql> SELECT IFNULL(1,0);
        -> 1
mysql> SELECT IFNULL(NULL,10);
        -> 10
mysql> SELECT IFNULL(1/0,10);
        -> 10
mysql> SELECT IFNULL(8%5,'yes');
        -> 3
```

### NULLIF()
NULLIF(expr1,expr2), 当 expr1 = expr2 时，返回NULL, 否则返回 expr1. 这个语句相当于</br> 
**CASE WHEN expr1 = expr2 THEN NULL ELSE expr1 END**.

```sql
mysql> SELECT NULLIF(1,1);
        -> NULL
mysql> SELECT NULLIF(1,2);
        -> 1
```








## 参考文档

[MySQL官方文档](https://dev.mysql.com/doc/refman/5.7/en/control-flow-functions.html#function_if)
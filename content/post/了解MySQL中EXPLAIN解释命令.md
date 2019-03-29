---
title:     "了解MySQL中EXPLAIN解释命令"  # 文章标题
subtitle:    "了解MySQL中EXPLAIN解释命令"  # 文章标题
description: "EXPLAIN会向我们提供一些MySQL是执行sql的信息"
date:         2018-08-13T11:49:13+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["MySQL"]
url:    "/2018-08-13-了解explain解释命令"
categories:  ["sql"]
showtoc: true   # 是否显示目录
---
## EXPLAIN优化查询

EXPLAIN会向我们提供一些MySQL是执行sql的信息：

1. EXPLAIN可以解释说明 SELECT, DELETE, INSERT, REPLACE, and UPDATE 等语句.

2. 当EXPLAIN与可解释的语句一起使用时，mysql会显示一些来自于优化器的关于sql执行计划的信息。即mysql解释它是如何处理这些语句的，和表之间是如何连接的。想获取更多关于EXPLAIN如何获取执行计划信息的。

3. 当EXPLAIN后面是一个会话的connection_id 而不是一个可执行的语句时，它会展示会话的信息。

4. 对于SELECT语句，EXPLAIN会产生额外的执行计划信息，这些信息可以用SHOW WARNINGS显示出来。

5. EXPLAIN对于检查设计分区表的查询时非常有用。

6. FORMAT选项可以用于选择输出格式，如果没有配置FORMAT选项，默认已表格形式输出。JSON 选项让信息已json格式展示。


EXPLAIN为SELECT中设计到的每张表返回一行信息。然后按照MySQL在处理语句时读取它们的顺序列出这些表，MySQL使用嵌套循环连接方法解析所有的连接。这意味着MySQL从第一个表中读取一行，然后在第二个表，第三个表中找到匹配的行，依此类推。处理完所有表后，MySQL将通过列表输出所选列和回溯，直到找到有更多匹配行的表。从该表中读取下一行，并继续下一个表。


EXPLAIN输出包括分区信息。此外，对于SELECT语句，EXPLAIN生成的扩展信息可以使用EXPLAIN后的SHOW WARNINGS来显示

EXPLAIN 输出列信息

EXPLAIN输出的字段信息：
第一列:列名, 第二列:FORMAT = JSON时输出中显示的等效属性名称 ,第三列：字段含义

|    Column   | JSON Name     |     Meaning     |
| ----------  | ---           |-----            |
| id          |select_id      |select标识号          
|select_type  |None            |select类型      
|table        |table_name     |这一行数据是关于哪张表的
|partitions   |partitions     |匹配的分区，对于未分区表，该值为空
|type         |access_type    |使用的连接类别,有无使用索引
|possible_keys|possible_keys  |MySQL能使用哪个索引在该表中找到行
|key          |key            |MySQL实际决定使用的键（索引）
|key_len      |key_length     |MySQL决定使用的键长度。如果键是NULL，长度为NULL
|ref          |ref            |与索引关联的列
|rows	        |rows           |mysql认为执行sql时必须被校验的行数
|filtered     |filtered       |表示此查询条件所过滤的数据的百分比
|Extra        |None           |附加信息

1. id: SELECT标识符。SELECT在查询中的序列号，可以为空。

2. select_type：SELECT类型，所有类型在下表中展示，JSON格式的EXPLAIN将SELECT类型公开为query_block的属性，除非它是SIMPLE或PRIMARY。 JSON名称(不适用为None)也显示在表中。

    |    select_type Value  | 	JSON Name               |     Meaning                          |
    | ----------            | ---                       |-----                                  |
    |SIMPLE                 |None                       |简单SELECT(不使用UNION或子查询等)          
    |PRIMARY                |None                       |嵌套查询时最外层的查询      
    |UNION                  |None                       |UNION中的第二个或后面的SELECT语句
    |DEPENDENT UNION        |dependent (true)           |UNION中的第二个或以后的SELECT语句，取决于外部查询
    |UNION RESULT           |	union_result              |UNION的结果。
    |SUBQUERY               |None                       |子查询中的第一个选择
    |DEPENDENT SUBQUERY     |dependent (true)           |子查询中的第一个选择，取决于外部查询
    |DERIVED                |None                       |派生表（子查询中产生的临时表）
    |MATERIALIZED           |materialized_from_subquery |物化子查询
    |UNCACHEABLE SUBQUERY	  |cacheable (false)          |无法缓存结果的子查询，必须对外部查询的每一行进行重新计算
    |UNCACHEABLE UNION      |cacheable (false)          |UNION中属于不可缓存子查询的第二个或以后的选择(请参 UNCACHEABLE SUBQUERY)


      * SIMPLE:简单SELECT(不使用UNION或子查询等)
      ```sql
      mysql> explain select * from t_a where id =1;
      +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
      | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
      +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
      |  1 | SIMPLE      | t_a   | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL  |
      +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
      1 row in set, 1 warning (0.03 sec)
      ```

      * PRIMARY:嵌套查询时最外层的查询
      (未完待续。。。)


参考：
1. [MySQL5.7 EXPLAIN Output Format](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

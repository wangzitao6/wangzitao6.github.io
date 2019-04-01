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
## 1 EXPLAIN概念

EXPLAIN会向我们提供一些MySQL是执行sql的信息：

1. EXPLAIN可以解释说明 SELECT, DELETE, INSERT, REPLACE, and UPDATE 等语句.

2. 当EXPLAIN与可解释的语句一起使用时，mysql会显示一些来自于优化器的关于sql执行计划的信息。即mysql解释它是如何处理这些语句的，和表之间是如何连接的。想获取更多关于EXPLAIN如何获取执行计划信息的。

3. 当EXPLAIN后面是一个会话的connection_id 而不是一个可执行的语句时，它会展示会话的信息。

4. 对于SELECT语句，EXPLAIN会产生额外的执行计划信息，这些信息可以用SHOW WARNINGS显示出来。

5. EXPLAIN对于检查设计分区表的查询时非常有用。

6. FORMAT选项可以用于选择输出格式，如果没有配置FORMAT选项，默认已表格形式输出。JSON 选项让信息已json格式展示。

## 2 EXPLAIN 输出列信息

表信息：
```sql
mysql> show create table t_a;
------+
| t_a   | CREATE TABLE `t_a` (
  `id` bigint(20) NOT NULL DEFAULT '0',
  `age` int(20) DEFAULT NULL,
  `code` int(20) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_code` (`code`),
  KEY `age_key` (`age`)
) ENGINE=InnoDB DEFAULT CHARSET=gbk |
+-------+-----------------------------------
------+
1 row in set (0.03 sec)
```

EXPLAIN输出的字段信息
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


### 2.1 id

SELECT标识符。SELECT在查询中的序列号，可以为空。

### 2.2 select_type

SELECT类型，所有类型在下表中展示，JSON格式的EXPLAIN将SELECT类型公开为query_block的属性，除非它是SIMPLE或PRIMARY。 JSON名称(不适用为None)也显示在表中。

|select_type Value      |JSON Name                  |Meaning                                |
| ----------            | ---                       |-----                                  |
|SIMPLE                 |None                       |简单SELECT(不使用UNION或子查询等)          
|PRIMARY                |None                       |嵌套查询时最外层的查询      
|UNION                  |None                       |UNION中的第二个或后面的SELECT语句
|DEPENDENT UNION        |dependent (true)           |UNION中的第二个或以后的SELECT语句，取决于外部查询
|UNION RESULT           |union_result               |UNION的结果
|SUBQUERY               |None                       |子查询中的第一个选择
|DEPENDENT SUBQUERY     |dependent (true)           |子查询中的第一个选择，取决于外部查询
|DERIVED                |None                       |派生表（子查询中产生的临时表）
|MATERIALIZED           |materialized_from_subquery |物化子查询
|UNCACHEABLE SUBQUERY	  |cacheable (false)          |无法缓存结果的子查询，必须对外部查询的每一行进行重新计算
|UNCACHEABLE UNION      |cacheable (false)          |UNION中属于不可缓存子查询的第二个或以后的选择(请参 UNCACHEABLE SUBQUERY)

* SIMPLE：简单SELECT(不使用UNION或子查询等)
  ```sql
  mysql> explain select * from t_a where id =1;
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
  | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
  |  1 | SIMPLE      | t_a   | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL  |
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
  1 row in set, 1 warning (0.03 sec)
  ```
  </br>

* PRIMARY：嵌套查询时最外层的查询
  ```sql
  mysql> explain select * from t_a where num >(select num from t_a where id = 3);
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
  | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra                    |
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
  |  1 | PRIMARY     | t_a   | NULL       | range | num_key       | num_key | 5       | NULL  |    6 |   100.00 | Using where; Using index |
  |  2 | SUBQUERY    | t_a   | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL                     |
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
  2 rows in set, 1 warning (0.03 sec)
  ```
  </br>

* UNION：UNION中的第二个或后面的SELECT语句
  ```sql
  mysql> explain select * from t_a where id =9 union all select * from t_a;
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
  | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
  |  1 | PRIMARY     | t_a   | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL        |
  |  2 | UNION       | t_a   | NULL       | index | NULL          | num_key | 5       | NULL  |    9 |   100.00 | Using index |
  +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------------+
  2 rows in set, 1 warning (0.04 sec)
  ```
  </br>

* DEPENDENT UNION：UNION中的第二个或以后的SELECT语句，取决于外部查询
  ```sql
  mysql> explain select * from t_a where id in (select id from t_a where id >8 union all select id from t_a where id =5);
  +----+--------------------+-------+------------+--------+---------------+---------+---------+-------+------+----------+--------------------------+
  | id | select_type        | table | partitions | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra                    |
  +----+--------------------+-------+------------+--------+---------------+---------+---------+-------+------+----------+--------------------------+
  |  1 | PRIMARY            | t_a   | NULL       | index  | NULL          | num_key | 5       | NULL  |    9 |   100.00 | Using where; Using index |
  |  2 | DEPENDENT SUBQUERY | t_a   | NULL       | eq_ref | PRIMARY       | PRIMARY | 8       | func  |    1 |   100.00 | Using where; Using index |
  |  3 | DEPENDENT UNION    | t_a   | NULL       | const  | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | Using index              |
  +----+--------------------+-------+------------+--------+---------------+---------+---------+-------+------+----------+--------------------------+
  3 rows in set, 1 warning (0.08 sec)
  ```
  </br>

* UNION RESULT：UNION的结果
  ```sql
  mysql> explain select num from t_a where id = 3 union select num from t_a where id =4;
  +----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
  | id | select_type  | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra           |
  +----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
  |  1 | PRIMARY      | t_a        | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL            |
  |  2 | UNION        | t_a        | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL            |
  | NULL | UNION RESULT | <union1,2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  | NULL |     NULL | Using temporary |
  +----+--------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+-----------------+
  3 rows in set, 1 warning (0.03 sec)
  ```
  </br>

* SUBQUERY：子查询中的第一个选择
 ```sql
 mysql> explain select * from t_a where num >(select num from t_a where id = 3);
 +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
 | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra                    |
 +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
 |  1 | PRIMARY     | t_a   | NULL       | range | num_key       | num_key | 5       | NULL  |    6 |   100.00 | Using where; Using index |
 |  2 | SUBQUERY    | t_a   | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL                     |
 +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
 2 rows in set, 1 warning (0.03 sec)
 ```
 </br>

* DEPENDENT SUBQUERY：子查询中的第一个选择，取决于外部查询
  ```sql
  mysql> explain select * from t_a where num in(select num from t_a where id = 3 union select num from t_a where id =4);
  +----+--------------------+------------+------------+-------+-----------------+---------+---------+-------+------+----------+--------------------------+
  | id | select_type        | table      | partitions | type  | possible_keys   | key     | key_len | ref   | rows | filtered | Extra                    |
  +----+--------------------+------------+------------+-------+-----------------+---------+---------+-------+------+----------+--------------------------+
  |  1 | PRIMARY            | t_a        | NULL       | index | NULL            | num_key | 5       | NULL  |    9 |   100.00 | Using where; Using index |
  |  2 | DEPENDENT SUBQUERY | t_a        | NULL       | const | PRIMARY,num_key | PRIMARY | 8       | const |    1 |   100.00 | NULL                     |
  |  3 | DEPENDENT UNION    | t_a        | NULL       | const | PRIMARY,num_key | PRIMARY | 8       | const |    1 |   100.00 | NULL                     |
  | NULL | UNION RESULT       | <union2,3> | NULL       | ALL   | NULL            | NULL    | NULL    | NULL  | NULL |     NULL | Using temporary          |
  +----+--------------------+------------+------------+-------+-----------------+---------+---------+-------+------+----------+--------------------------+
  4 rows in set, 1 warning (0.12 sec)
  ```
  </br>

* DERIVED：派生表（子查询中产生的临时表）
  ```sql
  mysql> explain select a.id from (select id from t_a where id >8 union all select id from t_a where id =5) a;
  +----+-------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
  | id | select_type | table      | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra                    |
  +----+-------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
  |  1 | PRIMARY     | <derived2> | NULL       | ALL   | NULL          | NULL    | NULL    | NULL  |    3 |   100.00 | NULL                     |
  |  2 | DERIVED     | t_a        | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL  |    1 |   100.00 | Using where; Using index |
  |  3 | UNION       | t_a        | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | Using index              |
  +----+-------------+------------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
  3 rows in set, 1 warning (0.12 sec)
  ```

### 2.3 table

显示这一行的数据是关于哪张表的，<font color=red>有时是真实的表名字，有时也可能是以下几种结果</font>

  * \<unionM,N\>: 指id为M,N行结果的并集

  * \<derivedN\>: 该行是指id值为n的行的派生表结果。派生表可能来自例如from子句中的子查询。

  * \<subqueryN\>: 该行是指id值为n的行的物化子查询的结果。

### 2.4 partitions
查询的记录所属于的分区，对于未分区表，该值为NULL。

### 2.5 type

连接使用了哪种类别,有无使用索引,常用的类型有：<font color=red> system,  const, eq_ref, ref, range, index, ALL（从左到右，性能越来越差）</font>，详情查看[ EXPLAIN Join Types](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-join-types)

* NULL：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成

* system：这个表（也可能是查询出来的临时表）只有一行数据 (= system table). 是const中的一个特例

* const：表最多有一个匹配行，它将在查询开始时被读取。因为仅有一行，在这行的列值可被优化器剩余部分认为是常数。const表很快，因为它们只读取一次！const用于查询条件为PRIMARY KEY或UNIQUE索引并与常数值进行比较时的所有部分。</br>
在下面的查询中，tbl_name可以用于const表：

    ```sql
    SELECT * from tbl_name WHERE primary_key=1；
    SELECT * from tbl_name WHERE primary_key_part1=1和 primary_key_part2=2；

    --例子
    mysql> explain select * from t_a where id =1;
    +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
    | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
    +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
    |  1 | SIMPLE      | t_a   | NULL       | const | PRIMARY       | PRIMARY | 8       | const |    1 |   100.00 | NULL  |
    +----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
    1 row in set, 1 warning (0.07 sec)
    ```

* eq_ref：对于前几个表中的每一行组合，从该表中读取一行。除了system和const，这是最好的连接类型。当连接使用索引的所有部分，并且索引是主键或唯一非空索引时，将使用它。eq_ref可以用于使用= 操作符比较的带索引的列。比较值可以为常量或一个使用在该表前面所读取的表的列的表达式。</br>
  在下面的例子中，MySQL可以使用eq_ref联接去处理ref_tables：

    ```sql
    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;

    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
      AND ref_table.key_column_part2=1;

    --例子（t_b为t_a的复制表，表结构相同）
    mysql> explain select * from t_a,t_b where t_a.code=t_b.code;
    +----+-------------+-------+------------+--------+---------------+---------+---------+---------------+------+----------+-------+
    | id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref           | rows | filtered | Extra |
    +----+-------------+-------+------------+--------+---------------+---------+---------+---------------+------+----------+-------+
    |  1 | SIMPLE      | t_a   | NULL       | ALL    | uk_code       | NULL    | NULL    | NULL          |    9 |   100.00 | NULL  |
    |  1 | SIMPLE      | t_b   | NULL       | eq_ref | uk_code       | uk_code | 4       | test.t_a.code |    1 |   100.00 | NULL  |
    +----+-------------+-------+------------+--------+---------------+---------+---------+---------------+------+----------+-------+
    2 rows in set, 1 warning (0.03 sec)
    ```

* ref对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY（换句话说，如果联接不能基于关键字查询结果为单个行的话），则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。ref可以用于使用=或<=>操作符的带索引的列。</br>
  在下面的例子中，MySQL可以使用ref联接来处理ref_tables：

    ```sql
    SELECT * FROM ref_table WHERE key_column=expr;

    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column=other_table.column;

    SELECT * FROM ref_table,other_table
      WHERE ref_table.key_column_part1=other_table.column
      AND ref_table.key_column_part2=1;

    --例子（t_b为t_a的复制表，表结构相同）
    mysql> explain select * from t_a,t_b where t_a.age=t_b.age;
    +----+-------------+-------+------------+------+---------------+---------+---------+--------------+------+----------+-------------+
    | id | select_type | table | partitions | type | possible_keys | key     | key_len | ref          | rows | filtered | Extra       |
    +----+-------------+-------+------------+------+---------------+---------+---------+--------------+------+----------+-------------+
    |  1 | SIMPLE      | t_a   | NULL       | ALL  | age_key       | NULL    | NULL    | NULL         |    9 |   100.00 | Using where |
    |  1 | SIMPLE      | t_b   | NULL       | ref  | age_key       | age_key | 5       | test.t_a.age |    1 |   100.00 | NULL        |
    +----+-------------+-------+------------+------+---------------+---------+---------+--------------+------+----------+-------------+
    2 rows in set, 1 warning (0.03 sec)
    ```

* fulltext：使用FULLTEXT索引执行连接

* ref_or_null：该联接类型ref类似，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。</br>
在下面的例子中，MySQL可以使用ref_or_null联接来处理ref_tables：

    ```sql
    SELECT * FROM ref_table
      WHERE key_column=expr OR key_column IS NULL;

    --例子
    mysql> explain select * from t_a where t_a.age =3 or t_a.age is null;
    +----+-------------+-------+------------+-------------+---------------+---------+---------+-------+------+----------+-----------------------+
    | id | select_type | table | partitions | type        | possible_keys | key     | key_len | ref   | rows | filtered | Extra                 |
    +----+-------------+-------+------------+-------------+---------------+---------+---------+-------+------+----------+-----------------------+
    |  1 | SIMPLE      | t_a   | NULL       | ref_or_null | age_key       | age_key | 5       | const |    2 |   100.00 | Using index condition |
    +----+-------------+-------+------------+-------------+---------------+---------+---------+-------+------+----------+-----------------------+
    1 row in set, 1 warning (0.03 sec)
    ```

* index_merge：该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。

    ```sql
    SELECT * FROM ref_table
      WHERE idx1=expr1 OR idx2 =expr2;

    --例子
    mysql> explain select * from t_a where t_a.code =3 or t_a.age = 3;
    +----+-------------+-------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-------------------------------------------+
    | id | select_type | table | partitions | type        | possible_keys   | key             | key_len | ref  | rows | filtered | Extra                                     |
    +----+-------------+-------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-------------------------------------------+
    |  1 | SIMPLE      | t_a   | NULL       | index_merge | uk_code,age_key | uk_code,age_key | 4,5     | NULL |    2 |   100.00 | Using union(uk_code,age_key); Using where |
    +----+-------------+-------+------------+-------------+-----------------+-----------------+---------+------+------+----------+-------------------------------------------+
    1 row in set, 1 warning (0.03 sec)
    ```

* unique_subquery：该类型替换了下面形式的IN子查询的ref：</br>
`value IN (SELECT primary_key FROM single_table WHERE some_expr)`</br>
unique_subquery是一个索引查找函数，可以完全替换子查询，效率更高。

* index_subquery：该联接类型类似于unique_subquery。可以替换IN子查询，但只适合下列形式的子查询中的非唯一索引：
`value IN (SELECT key_column FROM single_table WHERE some_expr)`

* range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。<font color=red>当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时，可以使用range </font>

    ```sql
    mysql> explain select * from t_a where id > 8;
    +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
    | id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
    +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
    |  1 | SIMPLE      | t_a   | NULL       | range | PRIMARY       | PRIMARY | 8       | NULL |    1 |   100.00 | Using where |
    +----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
    1 row in set, 1 warning (0.03 sec)
    ```

* index：该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。当查询只使用作为单索引一部分的列时，MySQL可以使用该联接类型。

* ALL：对于每个来自于先前的表的行组合，进行完整的表扫描。如果表是第一个没标记const的表，这通常不好，并且通常在它情况下很差。通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或列值被检索出。

### 2.6 possible_keys

possible_keys列指出MySQL能使用哪个索引在该表中找到行。注意，该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。

如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询

### 2.7 key

  key列显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

### 2.8 key_len

  key_len列显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。
  使用的索引的长度。在不损失精确性的情况下，长度越短越好

### 2.9 ref

  ref列显示使用哪个列或常数与key一起从表中选择行。

### 2.10 rows

  rows列显示MySQL认为它执行查询时必须检查的行数。

### 2.11 Extra

  该列包含MySQL解决查询的详细信息,下面详细.

1. Distinct：一旦MYSQL找到了与行相联合匹配的行，就不再搜索了

2. Not exists：MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行，就不再搜索了

3. Range checked for each：没有找到理想的索引，因此对于从前面表中来的每一个行组合，MYSQL检查使用哪个索引，并用它来从表中返回行。这是使用索引的最慢的连接之一

4. Using filesort：看到这个的时候，查询就需要优化了。MYSQL需要进行额外的步骤来发现如何对返回的行排序。它根据连接类型以及存储排序键值和匹配条件的全部行的行指针来排序全部行

5. Using index：列数据是从仅仅使用了索引中的信息而没有读取实际的行动的表返回的，这发生在对表的全部的请求列都是同一个索引的部分的时候

6. Using temporary：看到这个的时候，查询需要优化了。这里，MYSQL需要创建一个临时表来存储结果，这通常发生在对不同的列集进行ORDER BY上，而不是GROUP BY上

7.  Using where：使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。如果不想返回表中的全部行，并且连接类型ALL或index，这就会发生，或者是查询有问题


参考：
1. [MySQL5.7 EXPLAIN Output Format](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

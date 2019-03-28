---
title:     "了解MySQL死锁日志"  # 文章标题
subtitle:    "了解MySQL死锁日志"  # 文章标题
description: "讲解mysql死锁日志里面的一些重要信息"
date:       2018-08-05T13:36:46+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["死锁"]
url:    "/2018-08-05-死锁日志"
categories:  ["sql"]
showtoc: false   # 是否显示目录
---

表格信息：

  ```SQL
  CREATE TABLE `t_bitfly` (
    `id` bigint(20) NOT NULL DEFAULT '0',
    `num` int(20) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `num_key` (`num`)
  ) ENGINE=InnoDB DEFAULT CHARSET=gbk;
  ```

表中数据：

  ```SQL
  mysql> select * from t_bitfly;
  +----+------+
  | id | num  |
  +----+------+
  |  1 |    2 |
  |  3 |    5 |
  |  8 |    7 |
  +----+------+
  3 rows in set (0.04 sec)
  ```

数据库隔离级别为：可重复读（REPEATABLE-READ）

模拟死锁场景：

  ```SQL
  session 1:                                                       session 2:

  begin;                                                           begin;                                                     
  select * from t_bitfly where num = 7 lock in share mode ;
                                                                   select * from t_bitfly where num = 5 lock in share mode ;
  update  t_bitfly set num = 6 where num = 5
                                                                   update  t_bitfly set num = 8 where num = 7
  ```

结果：

  ```SQL
  update  t_bitfly set num = 6 where num = 5
  > 1213 - Deadlock found when trying to get lock; try restarting transaction
  > 时间: 2.392s
  ```

查询日志 ：`show engine innodb status ;`

结果如下
```log

=====================================
2018-08-05 21:20:27 0x7fd40c082700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 4 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 245 srv_active, 0 srv_shutdown, 15331 srv_idle
srv_master_thread log flush and writes: 15567
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 486
OS WAIT ARRAY INFO: signal count 428
RW-shared spins 0, rounds 539, OS waits 271
RW-excl spins 0, rounds 127, OS waits 1
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 539.00 RW-shared, 127.00 RW-excl, 0.00 RW-sx
------------------------
LATEST DETECTED DEADLOCK
------------------------
2018-08-05 21:14:20 0x7fd40c0b3700
*** (1) TRANSACTION:
TRANSACTION 1095001, ACTIVE 11 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1136, 3 row lock(s)
MySQL thread id 16, OS thread handle 140548578129664, query id 2959 183.6.50.229 root Searching rows for update
update  t_bitfly set num = 6 where num = 5
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2514 page no 4 n bits 72 index num_key of table `test`.`t_bitfly` trx id 1095001 lock_mode X waiting
Record lock, heap no 4 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 80000005; asc     ;;
 1: len 8; hex 8000000000000003; asc         ;;

*** (2) TRANSACTION:
TRANSACTION 1095002, ACTIVE 6 sec starting index read
mysql tables in use 1, locked 1
5 lock struct(s), heap size 1136, 3 row lock(s)
MySQL thread id 17, OS thread handle 140548711855872, query id 2963 183.6.50.229 root Searching rows for update
update  t_bitfly set num = 8 where num = 7
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 2514 page no 4 n bits 72 index num_key of table `test`.`t_bitfly` trx id 1095002 lock mode S
Record lock, heap no 4 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 80000005; asc     ;;
 1: len 8; hex 8000000000000003; asc         ;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2514 page no 4 n bits 72 index num_key of table `test`.`t_bitfly` trx id 1095002 lock_mode X waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 80000007; asc     ;;
 1: len 8; hex 8000000000000008; asc         ;;

省略。。。
```
一些注释：

* **LATEST DETECTED DEADLOCK**：标示为最新发生的死锁；

* **(1) TRANSACTION**：此处表示事务1开始 ；

* **TRANSACTION 1095001, ACTIVE 11 sec starting index read**：事务编号为 1095001 ，活跃11秒；

* **mysql tables in use 1, locked 1**：说明当前的事务使用一个表。locked 1 表示表上有一个表锁,对于DML语句为LOCK_IX；

* **MySQL thread id 17, OS thread handle 140548711855872, query id 2963 183.6.50.229 root Searching rows for update**：此处为记录当前数据库线程id；

* **update  t_bitfly set num = 6 where num = 5**：表示事务1在执行的sql ，不过比较悲伤的事情是show engine innodb status 是查看不到完整的事务的sql 的，通常显示当前正在等待锁的sql；

* **(1) WAITING FOR THIS LOCK TO BE GRANTED**：此处表示当前事务1等待获取行锁；

* **(2) TRANSACTION**：此处表示事务2开始 ；
* **update  t_bitfly set num = 8 where num = 7**：表示事务2在执行的sql

* **(2) HOLDS THE LOCK(S)**：此处表示当前事务2持有的行锁；

* **(2) WAITING FOR THIS LOCK TO BE GRANTED**：此处表示当前事务2等待获取行锁；


结合上文中的sql语境和死锁日志，可以看出事务一执行`update  t_bitfly set num = 6 where num = 5`时，需要加上X锁，此条记录已经在事务二中加了S锁，事务一等待事务二释放锁，事务二在执行`update  t_bitfly set num = 8 where num = 7`时，需要加上X锁，此条记录已经在事务一中加了S锁，事务二待事务一释放锁，产生了死锁。

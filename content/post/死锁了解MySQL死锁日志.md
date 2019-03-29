---
title:     "死锁-了解MySQL死锁日志"  # 文章标题
subtitle:    "死锁-了解MySQL死锁日志"  # 文章标题
description: "讲解mysql死锁日志里面的一些重要信息"
date:       2018-08-05T13:36:46+08:00  # 自动添加日期信息
author:   "WangZiTao"
image:   ""
tags:        ["死锁","MySQL"]
url:    "/2018-08-05-死锁日志"
categories:  ["sql"]
showtoc: false   # 是否显示目录
---
锁的种类&概念

* **Shared and Exclusive Locks**

  ```
  * Shared lock: 共享锁,官方描述：permits the transaction that holds the lock to read a row
  eg：select * from xx where a=1 lock in share mode
  * Exclusive Locks：排他锁： permits the transaction that holds the lock to update or delete a row
  eg: select * from xx where a=1 for update
  ```
</br>
* **Intention Locks**

  ```
  1. 这个锁是加在table上的，表示要对下一个层级（记录）进行加锁  
  2. Intention shared (IS）：Transaction T intends to set S locks on individual rows in table t  
  3. Intention exclusive (IX):  Transaction T intends to set X locks on those rows  
  4. 在数据库层看到的结果是这样的：
      TABLE LOCK table `lc_3`.`a` trx id 133588125 lock mode IX
  ```
</br>
* **Record Locks**

  ```
  1. 在数据库层看到的结果是这样的：
      RECORD LOCKS space id 281 page no 3 n bits 72 index PRIMARY of table `lc_3`.`a` trx id 133588125 lock_mode X locks rec but not gap
  2. 该锁是加在索引上的（从上面的index PRIMARY of table `lc_3`.`a` 就能看出来）
  3. 记录锁可以有两种类型：lock_mode X locks rec but not gap  && lock_mode S locks rec but not gap
  ```
</br>
* **Gap Locks**

  ```
  1. 在数据库层看到的结果是这样的：
      RECORD LOCKS space id 281 page no 5 n bits 72 index idx_c of table `lc_3`.`a` trx id 133588125 lock_mode X locks gap before rec  
  2. Gap锁是用来防止insert的  
  3. Gap锁，中文名间隙锁，锁住的不是记录，而是范围,比如：(negative infinity, 10），(10, 11）区间，这里都是开区间哦
  ```
</br>
* **Next-Key Locks**

  ```
  1. 在数据库层看到的结果是这样的：
      RECORD LOCKS space id 281 page no 5 n bits 72 index idx_c of table `lc_3`.`a` trx id 133588125 lock_mode X
  2. Next-Key Locks = Gap Locks + Record Locks 的结合, 不仅仅锁住记录，还会锁住间隙，
     比如： (negative infinity, 10】，(10, 11】区间，这些右边都是闭区间哦
  ```
</br>
*  **Insert Intention Locks**

  ```
  1. 在数据库层看到的结果是这样的：
      RECORD LOCKS space id 279 page no 3 n bits 72 index PRIMARY of table `lc_3`.`t1` trx id 133587907 lock_mode X insert intention waiting
  2. Insert Intention Locks 可以理解为特殊的Gap锁的一种，用以提升并发写入的性能
  ```
</br>
* **AUTO-INC Locks**

  ```
1. 在数据库层看到的结果是这样的：
    TABLE LOCK table xx trx id 7498948 lock mode AUTO-INC waiting
2. 属于表级别的锁  
3. 自增锁的详细情况可以之前的一篇文章:
    http://keithlan.github.io/2017/03/03/auto_increment_lock/
  ```
</br>
* **显示锁 vs 隐示锁**

  ```
* 显示锁(explicit lock)
    显示的加锁，在show engine innoDB status 中能够看到  ，会在内存中产生对象，占用内存  
    eg: select ... for update , select ... lock in share mode   
* 隐示锁(implicit lock)
    implicit lock 是在索引中对记录逻辑的加锁，但是实际上不产生锁对象，不占用内存空间  

* 哪些语句会产生implicit lock 呢？
   eg: insert into xx values(xx)
   eg: update xx set t=t+1 where id = 1 ; 会对辅助索引加implicit lock  
* implicit lock 在什么情况下会转换成 explicit lock
  eg： 只有implicit lock 产生冲突的时候，会自动转换成explicit lock,这样做的好处就是降低锁的开销    
  eg: 比如：我插入了一条记录10，本身这个记录加上implicit lock，如果这时候有人再去更新这条10的记录，那么就会自动转换成explicit lock
* 数据库怎么知道implicit lock的存在呢？如何实现锁的转化呢？
  1. 对于聚集索引上面的记录，有db_trx_id,如果该事务id在活跃事务列表中，那么说明还没有提交，那么implicit则存在  
  2. 对于非聚集索引：由于上面没有事务id，那么可以通过上面的主键id，再通过主键id上面的事务id来判断，不过算法要非常复杂，这里不做介绍
  ```

记录锁，间隙锁，Next-key 锁和插入意向锁。这四种锁对应的死锁如下：

  - <font color=red>记录锁（LOCK_REC_NOT_GAP）: lock_mode X locks rec but not gap </font>

  - <font color=red>间隙锁（LOCK_GAP）: lock_mode X locks gap before rec </font>

  - <font color=red>Next-key 锁（LOCK_ORNIDARY）: lock_mode X </font>

  - <font color=red>插入意向锁（LOCK_INSERT_INTENTION）: lock_mode X locks gap before rec insert intention </font>

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
  session 1:                                    session 2:

  begin;                                        begin;                                                     
  delete from t_bitfly  where num = 5 ;
                                                delete from t_bitfly where num = 7 ;
  insert into t_bitfly values(7,7) ;
                                                insert into t_bitfly values(5,5) ;
  ```

结果：

    ```SQL
    insert into t_bitfly values(5,5)
    > 1213 - Deadlock found when trying to get lock; try restarting transaction
    > 时间: 0.085s
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
srv_master_thread loops: 251 srv_active, 0 srv_shutdown, 22663 srv_idle
srv_master_thread log flush and writes: 22905
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 513
OS WAIT ARRAY INFO: signal count 450
RW-shared spins 0, rounds 569, OS waits 286
RW-excl spins 0, rounds 127, OS waits 1
RW-sx spins 0, rounds 0, OS waits 0
Spin rounds per wait: 569.00 RW-shared, 127.00 RW-excl, 0.00 RW-sx
------------------------
LATEST DETECTED DEADLOCK
------------------------
2018-08-05 21:15:42 0x7fd40c0b3700
*** (1) TRANSACTION:
TRANSACTION 1095010, ACTIVE 21 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 5 lock struct(s), heap size 1136, 4 row lock(s), undo log entries 2
MySQL thread id 16, OS thread handle 140548578129664, query id 3052 183.6.50.229 root update
insert into t_bitfly values(7,7)
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2514 page no 4 n bits 72 index num_key of table `test`.`t_bitfly` trx id 1095010 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 2; compact format; info bits 32
 0: len 4; hex 80000007; asc     ;;
 1: len 8; hex 8000000000000008; asc         ;;

*** (2) TRANSACTION:
TRANSACTION 1095015, ACTIVE 6 sec inserting
mysql tables in use 1, locked 1
4 lock struct(s), heap size 1136, 4 row lock(s), undo log entries 2
MySQL thread id 17, OS thread handle 140548711855872, query id 3056 183.6.50.229 root update
insert into t_bitfly values(5,5)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 2514 page no 4 n bits 72 index num_key of table `test`.`t_bitfly` trx id 1095015 lock_mode X
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
 0: len 8; hex 73757072656d756d; asc supremum;;

Record lock, heap no 3 PHYSICAL RECORD: n_fields 2; compact format; info bits 32
 0: len 4; hex 80000007; asc     ;;
 1: len 8; hex 8000000000000008; asc         ;;

*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 2514 page no 4 n bits 72 index num_key of table `test`.`t_bitfly` trx id 1095015 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 2; compact format; info bits 32
 0: len 4; hex 80000007; asc     ;;
 1: len 8; hex 8000000000000008; asc         ;;

省略。。。
```
一些注释：

* **LATEST DETECTED DEADLOCK**：标示为最新发生的死锁；

* **(1) TRANSACTION**：此处表示事务1开始 ；

* **MySQL thread id 16, OS thread handle 140548578129664, query id 3052 183.6.50.229 root update**：此处为记录当前数据库线程id；

* **insert into t_bitfly values(7,7)**：表示事务1在执行的sql ，不过比较悲伤的事情是show engine innodb status 是查看不到完整的事务的sql 的，通常显示当前正在等待锁的sql；

* **(1) WAITING FOR THIS LOCK TO BE GRANTED**：此处表示当前事务1等待获取行锁；

* **(2) TRANSACTION**：此处表示事务2开始 ；

* **insert into t_bitfly values(5,5)**：表示事务2在执行的sql

* **(2) HOLDS THE LOCK(S)**：此处表示当前事务2持有的行锁；

* **(2) WAITING FOR THIS LOCK TO BE GRANTED**：此处表示当前事务2等待获取行锁；


根据死锁日志可以看出：

事务一在执行`insert into t_bitfly values(7,7)`时，插入意向锁加锁时卡住；

事务二在执行`insert into t_bitfly values(5,5)`时，持有next-key锁，插入意向锁加锁时卡住。

结合上面执行的sql来分析：

* 事务一执行`delete from t_bitfly  where num = 5 ;`后，获取了 Gap Locks + Record Locks 也就是 next-key锁；

* 事务二执行`delete from t_bitfly  where num = 7 ;`后，获取了 Gap Locks + Record Locks 也就是 next-key锁；

* 事务一执行`insert into t_bitfly values(7,7)`时，持有next-key锁，插入意向锁，等待事务二的next-key锁解锁；

* 事务二执行`insert into t_bitfly values(5,5)`时，持有next-key锁，插入意向锁，等待事务二的next-key锁解锁；

产生死锁。

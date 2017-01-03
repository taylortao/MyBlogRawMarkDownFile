---
title: MySQL隔离级别
date: 2011-12-12 17:14:00
categories:
 - IT Technology
tags:
 - Database
 - MySQL
---

### 事务ACID

原子性(Atomicity)：要么全部执行,要么不执行。
一致性(Consistency)：不改变数据库中数据的一致性。
独立性(Isolation)：两个以上的事务不会出现交错执行的状态。
持久性(Durability)：对数据库所作的更改是持久的保存。


### Isolation并发可能引起的问题

> 1.脏读

允许读取到未提交的脏数据。
<!-- more -->

> 2.不可重复读

如果你在时间点T1读取了一些记录，在T2时再想重新读取一次同样的这些记录时，这些记录可能已经被改变、或者消失不见。

> 3.幻读

解决了不重复读，保证了同一个事务里，查询的结果都是事务开始时的状态（一致性）。但是，如果另一个事务同时提交了新数据，本事务再更新时，就会“惊奇的”发现了这些新数据，貌似之前读到的数据是“鬼影”一样的幻觉。


### 隔离级别

由ANSI/ISO定义的SQL-92标准定义的四种隔离级别

1.Read Uncommitted

2.Read Committed

3.Repeatable Read

4.Serializable


| 隔离解别 | 脏读 | 不可重复读 | 幻读 |
| - | :-:  | :-: | :-: |
| Read Uncommitted | Y | Y | Y |
| Read Committed | N | Y | Y |
| Repeatable(default) | N | N | Y |
| Serializable | N | N | N |



### 实验

Mysql 版本号

```mysql
mysql> select version();  
+------------+  
| version()  |  
+------------+  
| 5.1.52-log |  
+------------+  
1 row in set (0.00 sec) 
```


查看InnoDB存储引擎 系统级的隔离级别 和 会话级的隔离级别

```mysql
mysql> select @@global.tx_isolation,@@tx_isolation;  
+-----------------------+-----------------+  
| @@global.tx_isolation | @@tx_isolation  |  
+-----------------------+-----------------+  
| REPEATABLE-READ       | REPEATABLE-READ |  
+-----------------------+-----------------+  
1 row in set (0.00 sec) 
```


更改会话级的隔离级别
Session 1:

```mysql
mysql> set session tx_isolation='read-uncommitted';  
Query OK, 0 rows affected (0.00 sec) 
 
mysql> select @@global.tx_isolation,@@tx_isolation;  
+-----------------------+------------------+  
| @@global.tx_isolation | @@tx_isolation   |  
+-----------------------+------------------+  
| REPEATABLE-READ       | READ-UNCOMMITTED |  
+-----------------------+------------------+  
1 row in set (0.00 sec)  
```

Session 2:

```mysql
mysql> select @@global.tx_isolation, @@tx_isolation;  
+-----------------------+-----------------+  
| @@global.tx_isolation | @@tx_isolation  |  
+-----------------------+-----------------+  
| REPEATABLE-READ       | REPEATABLE-READ |  
+-----------------------+-----------------+  
1 row in set (0.00 sec)  
```


更改系统级的隔离级别
Session 1: 

```mysql 
mysql> set global tx_isolation='read-uncommitted';  
Query OK, 0 rows affected (0.00 sec)  
mysql> select @@global.tx_isolation,@@tx_isolation;  
+-----------------------+------------------+  
| @@global.tx_isolation | @@tx_isolation   |  
+-----------------------+------------------+  
| READ-UNCOMMITTED      | READ-UNCOMMITTED |  
+-----------------------+------------------+  
1 row in set (0.00 sec)  
```

Session 2:

```mysql  
mysql> select @@global.tx_isolation, @@tx_isolation;  
+-----------------------+-----------------+  
| @@global.tx_isolation | @@tx_isolation  |  
+-----------------------+-----------------+  
| READ-UNCOMMITTED      | REPEATABLE-READ |  
+-----------------------+-----------------+  
1 row in set (0.00 sec)  
```


关闭SQL语句的自动提交

```mysql
mysql> set autocommit=off;  
Query OK, 0 rows affected (0.00 sec)  
```

查看SQL语句自动提交是否关闭

```mysql
mysql> show variables like 'autocommit';  
+---------------+-------+  
| Variable_name | Value |  
+---------------+-------+  
| autocommit    | OFF   |  
+---------------+-------+  
1 row in set (0.00 sec)  
```


建立实验表

```mysql
mysql> create table tao (col1 tinyint unsigned, col2 varchar(20), primary key(col1));  
Query OK, 0 rows affected (0.08 sec)  
  
mysql> show create table tao \G;  
*************************** 1. row ***************************  
       Table: tao  
Create Table: CREATE TABLE `tao` (  
  `col1` tinyint(3) unsigned NOT NULL DEFAULT '0',  
  `col2` varchar(20) DEFAULT NULL,  
  PRIMARY KEY (`col1`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8  
1 row in set (0.00 sec)  
```

#### 演示脏读Dirty Reads

![demo of Dirty Reads](dirtyread.gif)

更改隔离级别为Read Committed后，不存在脏读的问题。

```mysql
mysql> set global tx_isolation='read-committed';  
Query OK, 0 rows affected (0.00 sec)  
mysql> set session tx_isolation='read-committed';  
Query OK, 0 rows affected (0.00 sec) 
```

#### 演示不可重复读Nonrepeatable Reads

![demo of Nonrepeatable Reads](nonrepeatableread.gif)

更改隔离级别为Repeatable Read后，不存在不可重复读的问题。

```mysql
mysql> set global tx_isolation='repeatable-read';  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> set session tx_isolation='repeatable-read';  
Query OK, 0 rows affected (0.00 sec)  
```

#### 演示幻读Phantoms

![demo of Phantoms](phantom.gif)


更改隔离级别为完全串行化 Serializable 后，不存在幻读的问题。

```mysql
mysql> set global tx_isolation='serializable';  
Query OK, 0 rows affected (0.00 sec)  
  
mysql> set session tx_isolation='serializable';  
Query OK, 0 rows affected (0.00 sec)  
```

在这种情况下，只允许一个事务在执行，其它事务必须等待这个事务执行完后才能执行。没有并发，只是单纯的串行。


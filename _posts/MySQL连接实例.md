---
title: MySQL连接实例
date: 2012-06-14 19:07:00
categories:
 - IT Technology
tags:
 - MySQL
toc: false
---

各种有趣的MySQL连接实例，直接上代码。

关于MySQL join的说明可参考：[MySQL的Join使用][1]

<!-- more -->

```sql
mysql> use test  
Database changed  
mysql> show tables;  
Empty set (0.00 sec)  
  
mysql> create table person( id smallint unsigned NOT NULL AUTO_INCREMENT, name varchar(30), primary key(id));  
Query OK, 0 rows affected (0.12 sec)  
  
mysql> create table score(id smallint unsigned NOT NULL, score int);  
Query OK, 0 rows affected (0.14 sec)  
  
mysql> show tables;  
+----------------+  
| Tables_in_test |  
+----------------+  
| person         |  
| score          |  
+----------------+  
2 rows in set (0.00 sec)  
  
mysql> insert into person(name) values ('zrx'),('genius'),('stupid');  
Query OK, 3 rows affected (0.11 sec)  
Records: 3  Duplicates: 0  Warnings: 0  
  
mysql> select * from person;  
+----+--------+  
| id | name   |  
+----+--------+  
|  1 | zrx    |  
|  2 | genius |  
|  3 | stupid |  
+----+--------+  
3 rows in set (0.00 sec)  
  
mysql> insert into score values (1, 9),(2, 100);  
Query OK, 2 rows affected (0.11 sec)  
Records: 2  Duplicates: 0  Warnings: 0  
  
mysql> insert into score values (4,20);  
Query OK, 1 row affected (0.11 sec)  

mysql> insert into score values (1,1);  
Query OK, 1 row affected (0.11 sec)  
  
mysql> select * from score;  
+----+-------+  
| id | score |  
+----+-------+  
|  1 |     9 |  
|  2 |   100 |  
|  4 |    20 |  
|  1 |     1 |  
+----+-------+  
4 rows in set (0.00 sec)  

mysql> select * from person inner join score ;  
+----+--------+----+-------+  
| id | name   | id | score |  
+----+--------+----+-------+  
|  1 | zrx    |  1 |     9 |  
|  2 | genius |  1 |     9 |  
|  3 | stupid |  1 |     9 |  
|  1 | zrx    |  2 |   100 |  
|  2 | genius |  2 |   100 |  
|  3 | stupid |  2 |   100 |  
|  1 | zrx    |  4 |    20 |  
|  2 | genius |  4 |    20 |  
|  3 | stupid |  4 |    20 |  
|  1 | zrx    |  1 |     1 |  
|  2 | genius |  1 |     1 |  
|  3 | stupid |  1 |     1 |  
+----+--------+----+-------+  
12 rows in set (0.00 sec)  

mysql> select * from person inner join score where person.id=score.id;  
+----+--------+----+-------+  
| id | name   | id | score |  
+----+--------+----+-------+  
|  1 | zrx    |  1 |     9 |  
|  2 | genius |  2 |   100 |  
|  1 | zrx    |  1 |     1 |  
+----+--------+----+-------+  
3 rows in set (0.00 sec)  

mysql> select * from (person, score);  
+----+--------+----+-------+  
| id | name   | id | score |  
+----+--------+----+-------+  
|  1 | zrx    |  1 |     9 |  
|  2 | genius |  1 |     9 |  
|  3 | stupid |  1 |     9 |  
|  1 | zrx    |  2 |   100 |  
|  2 | genius |  2 |   100 |  
|  3 | stupid |  2 |   100 |  
|  1 | zrx    |  4 |    20 |  
|  2 | genius |  4 |    20 |  
|  3 | stupid |  4 |    20 |  
|  1 | zrx    |  1 |     1 |  
|  2 | genius |  1 |     1 |  
|  3 | stupid |  1 |     1 |  
+----+--------+----+-------+  
12 rows in set (0.00 sec)  

mysql> select * from (person, score) using (id);  
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that  
corresponds to your MySQL server version for the right syntax to use near 'using  
 (id)' at line 1  
mysql> select * from (person, score) where person.id=score.id;  
+----+--------+----+-------+  
| id | name   | id | score |  
+----+--------+----+-------+  
|  1 | zrx    |  1 |     9 |  
|  2 | genius |  2 |   100 |  
|  1 | zrx    |  1 |     1 |  
+----+--------+----+-------+  
3 rows in set (0.00 sec)  

mysql> select * from person inner join score using (id);  
+----+--------+-------+  
| id | name   | score |  
+----+--------+-------+  
|  1 | zrx    |     9 |  
|  2 | genius |   100 |  
|  1 | zrx    |     1 |  
+----+--------+-------+  
3 rows in set (0.00 sec)  

mysql> select * from person natural join score;  
+----+--------+-------+  
| id | name   | score |  
+----+--------+-------+  
|  1 | zrx    |     9 |  
|  2 | genius |   100 |  
|  1 | zrx    |     1 |  
+----+--------+-------+  
3 rows in set (0.00 sec)  

mysql> select * from person left join score;  
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that  
corresponds to your MySQL server version for the right syntax to use near '' at  
line 1  

mysql> select * from person left join score where person.id=score.id;  
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that  
corresponds to your MySQL server version for the right syntax to use near 'where  
 person.id=score.id' at line 1  

mysql> select * from person left join score using(id);  
+----+--------+-------+  
| id | name   | score |  
+----+--------+-------+  
|  1 | zrx    |     9 |  
|  1 | zrx    |     1 |  
|  2 | genius |   100 |  
|  3 | stupid |  NULL |  
+----+--------+-------+  
4 rows in set (0.00 sec)  

mysql> select * from person left join score on person.id=score.id;  
+----+--------+------+-------+  
| id | name   | id   | score |  
+----+--------+------+-------+  
|  1 | zrx    |    1 |     9 |  
|  1 | zrx    |    1 |     1 |  
|  2 | genius |    2 |   100 |  
|  3 | stupid | NULL |  NULL |  
+----+--------+------+-------+  
4 rows in set (0.00 sec)  

mysql> select * from person right join score on person.id=score.id;  
+------+--------+----+-------+  
| id   | name   | id | score |  
+------+--------+----+-------+  
|    1 | zrx    |  1 |     9 |  
|    2 | genius |  2 |   100 |  
| NULL | NULL   |  4 |    20 |  
|    1 | zrx    |  1 |     1 |  
+------+--------+----+-------+  
4 rows in set (0.00 sec)  

mysql> select * from person full outer join score on person.id=score.id;  
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that  
corresponds to your MySQL server version for the right syntax to use near 'outer  
 join score on person.id=score.id' at line 1  

mysql> select * from person full join score on person.id=score.id;  
ERROR 1054 (42S22): Unknown column 'person.id' in 'on clause'
  
mysql> select * from person full join score using(id);  
+----+--------+-------+  
| id | name   | score |  
+----+--------+-------+  
|  1 | zrx    |     9 |  
|  2 | genius |   100 |  
|  1 | zrx    |     1 |  
+----+--------+-------+  
3 rows in set (0.00 sec)  

mysql> select * from person full join score on person.id=score.id;  
ERROR 1054 (42S22): Unknown column 'person.id' in 'on clause'  

mysql> select * from person p full join score s on p.id=s.id;  
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that  
corresponds to your MySQL server version for the right syntax to use near 'full  
join score s on p.id=s.id' at line 1  

mysql> select * from person full outer join score using(id);  
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that  
corresponds to your MySQL server version for the right syntax to use near 'outer  
 join score using(id)' at line 1  

mysql> select * from person left join score using(id) union select * from person  
 right join score using (id);  
+----+--------+--------+  
| id | name   | score  |  
+----+--------+--------+  
|  1 | zrx    | 9      |  
|  1 | zrx    | 1      |  
|  2 | genius | 100    |  
|  3 | stupid | NULL   |  
|  1 | 9      | zrx    |  
|  2 | 100    | genius |  
|  4 | 20     | NULL   |  
|  1 | 1      | zrx    |  
+----+--------+--------+  
8 rows in set (0.00 sec)  

mysql> select * from person left join score on person.id=score.id union select *  
 from person right join score on person.id=score.id;  
+------+--------+------+-------+  
| id   | name   | id   | score |  
+------+--------+------+-------+  
|    1 | zrx    |    1 |     9 |  
|    1 | zrx    |    1 |     1 |  
|    2 | genius |    2 |   100 |  
|    3 | stupid | NULL |  NULL |  
| NULL | NULL   |    4 |    20 |  
+------+--------+------+-------+  
5 rows in set (0.00 sec)  
```

MySQL本身不支持full join（全连接），但可以通过union来实现。

[1]: /2011/12/13/MySQL的Join使用/
---
title: MySQL的Join使用
date: 2011-12-13 22:17:00
categories:
 - IT Technology
tags:
 - Database
 - MySQL
---

在MySQL(以5.1为例)中，表连接的语法可以参见MySQL官方手册：[MySQL官方手册-JOIN][1]

在查询中，连接的语法类似
```mysql
SELECT select_expr FROM table_references  
```

table_references（对表的引用）的定义如下（也可以看成连接表达式）：（晕晕晕哈）
<!-- more -->
```mysql
table_references:  
    table_reference [, table_reference] ...  
  
table_reference:  
    table_factor  
  | join_table  
  
table_factor:  
    tbl_name [[AS] alias] [index_hint_list]  
  | table_subquery [AS] alias  
  | ( table_references )  
  | { OJ table_reference LEFT OUTER JOIN table_reference  
        ON conditional_expr }  
  
join_table:  
    table_reference [INNER | CROSS] JOIN table_factor [join_condition]  
  | table_reference STRAIGHT_JOIN table_factor  
  | table_reference STRAIGHT_JOIN table_factor ON conditional_expr  
  | table_reference {LEFT|RIGHT} [OUTER] JOIN table_reference join_condition  
  | table_reference NATURAL [{LEFT|RIGHT} [OUTER]] JOIN table_factor  
  
join_condition:  
    ON conditional_expr  
  | USING (column_list)  
  
index_hint_list:  
    index_hint [, index_hint] ...  
  
index_hint:  
    USE {INDEX|KEY}  
      [{FOR {JOIN|ORDER BY|GROUP BY}] ([index_list])  
  | IGNORE {INDEX|KEY}  
      [{FOR {JOIN|ORDER BY|GROUP BY}] (index_list)  
  | FORCE {INDEX|KEY}  
      [{FOR {JOIN|ORDER BY|GROUP BY}] (index_list)  
  
index_list:  
    index_name [, index_name] ...  
```

其中，table_factor是基本的表选择，而join_table是基于表的一些扩展。
下面，通过实验介绍一下表连接。
首先，假设有以下几个表

table1:

| id | book |
| :-: | :-: |
| 1 | java|
| 2 | c++ |
| 3 | php |

table2:

| id | author |
| :-: | :-: |
| 2 | zhang |
| 3 | wang |
| 4 | li |

table3:

| author | year |
| :-: | :-: |
| zhang | 2003 |
| ma | 2006 |
| liu | 2011 |


### Inner Join 内连接
将两个表中存在连接关系的字段，组成的记录集，叫做内连接。
内连接等价于

```mysql
mysql> select table1.id as id,book,author from table1, table2 where table1.id=table2.id;  
+------+------+--------+  
| id   | book | author |  
+------+------+--------+  
|    2 | c++  | zhang  |  
|    3 | php  | wang   |  
+------+------+--------+  
2 rows in set (0.00 sec)  
mysql> select * from table1 inner join table2 using (id);  
+------+------+--------+  
| id   | book | author |  
+------+------+--------+  
|    2 | c++  | zhang  |  
|    3 | php  | wang   |  
+------+------+--------+  
2 rows in set (0.00 sec) 
```

可以看出，两者是等价的。没有Using子句的Inner Join相当于是求两个table的笛卡尔积。

### Cross Join 交叉连接
在Mysql中，Cross Join可以用逗号表达式表示，例如(table1, table 2)。在Mysql中，Cross Join 和 Inner Join 是等价的，但是在标准SQL中，它们并不等价，Inner Join 用于带有on表达式的连接，反之用Cross Join。
Cross Join 指的是两个table的笛卡尔积。以下三句SQL是等价的。

```mysql
mysql> select * from table1 inner join table2;  
mysql> select * from table1 cross join table2;  
mysql> select * from (table1, table2);  
mysql> select * from table1 nature join table2;  
结果集：  
+------+------+------+--------+  
| id   | book | id   | author |  
+------+------+------+--------+  
|    1 | java |    2 | zhang  |  
|    2 | c++  |    2 | zhang  |  
|    3 | php  |    2 | zhang  |  
|    1 | java |    3 | wang   |  
|    2 | c++  |    3 | wang   |  
|    3 | php  |    3 | wang   |  
|    1 | java |    4 | li     |  
|    2 | c++  |    4 | li     |  
|    3 | php  |    4 | li     |  
+------+------+------+--------+  
```

不难理解，下面两句SQL也是等价的。
```mysql
mysql> select * from table1 left join (table2, table3) on (table2.id = table1.id and table2.author = table3.author);  
mysql> select * from table1 left join (table2 cross join table3) on (table2.id = table1.id and table2.author = table3.author);  
结果集：  
+------+------+------+--------+--------+------+  
| id   | book | id   | author | author | year |  
+------+------+------+--------+--------+------+  
|    1 | java | NULL | NULL   | NULL   | NULL |  
|    2 | c++  |    2 | zhang  | zhang  | 2003 |  
|    3 | php  | NULL | NULL   | NULL   | NULL |  
+------+------+------+--------+--------+------+  
```

### Natural Join 自然连接
NATURAL [LEFT] JOIN：这个句子的作用相当于INNER JOIN，并在USING子句中包含了联结的表中所有公共字段的。
也就是说：下面两个SQL是等价的。

```mysql
mysql> select * from table1 natural join table2;  
mysql> select * from table1 inner join table2 using (id);  
  
结果集：  
+------+------+--------+  
| id   | book | author |  
+------+------+--------+  
|    2 | c++  | zhang  |  
|    3 | php  | wang   |  
+------+------+--------+  
```

同时，下面两个SQL也是等价的。

```mysql
mysql> select * from table1 natural left join table2;  
mysql> select * from table1 left join table2 using(id);  
结果集：  
+------+------+--------+  
| id   | book | author |  
+------+------+--------+  
|    1 | java | NULL   |  
|    2 | c++  | zhang  |  
|    3 | php  | wang   |  
+------+------+--------+  
```

### Left Join 左外连接
左外连接A、B表的意思就是将表A中的全部记录和表B中字段连接形成的记录集，这里注意的是最后出来的记录集会包括表A的全部记录。
(左连接表1，表二)等价于(右连接表二，表一)。如下两个SQL是等价的：

```mysql
mysql> select * from table1 left join table2 using (id);  
mysql> select * from table2 right join table1 using (id);  
结果集：  
+------+------+--------+  
| id   | book | author |  
+------+------+--------+  
|    1 | java | NULL   |  
|    2 | c++  | zhang  |  
|    3 | php  | wang   |  
+------+------+--------+  
```

### Right Join 右外连接
右外连接和左外连接是类似的。为了方便数据库便于访问，推荐使用左外连接代替右外连接。

### 注意事项

#### 两个表求差集的方法
如果求 左表 - 右表 的差集，使用类似下面的SQL：

```mysql
SELECT left_tbl.* FROM left_tbl LEFT JOIN right_tbl ON left_tbl.id = right_tbl.id WHERE right_tbl.id IS NULL;  
```

例如

```mysql 
mysql> select table1.* from table1 left join table2 on table1.id = table2.id where table2.id is null;  
+------+------+  
| id   | book |  
+------+------+  
|    1 | java |  
+------+------+  
1 row in set (0.00 sec)  
```

注意：必须是table2.id 为空，而不能是table2.author为空，因为table2中可能存在id不为空但是author为空的数据。但是table2.id可以换成任意table2中不为空的列。

#### Using子句
Using子句可以使用On子句重写。但是使用Select *查询出的结果有差别。以下两句话是等价的：

```mysql
mysql> select id, book, author from table1 join table2 using (id);  
mysql> select table1.id, book, author from table1 join table2 on table1.id=table2.id;  
结果集：  
+------+------+--------+  
| id   | book | author |  
+------+------+--------+  
|    2 | c++  | zhang  |  
|    3 | php  | wang   |  
+------+------+--------+ 
```

但是下面两个有些许不同，使用on时候，重复的部分会被输出两次。

```mysql
mysql> select * from table1 join table2 using (id);  
+------+------+--------+  
| id   | book | author |  
+------+------+--------+  
|    2 | c++  | zhang  |  
|    3 | php  | wang   |  
+------+------+--------+  
2 rows in set (0.00 sec)  
mysql> select * from table1 join table2 on table1.id=table2.id;  
+------+------+------+--------+  
| id   | book | id   | author |  
+------+------+------+--------+  
|    2 | c++  |    2 | zhang  |  
|    3 | php  |    3 | wang   |  
+------+------+------+--------+  
2 rows in set (0.00 sec) 
```

#### Straight Join的使用
STRAIGHT_JOIN 和 JOIN相似，除了大部分情况下，在使用STRAIGHT_JOIN时候，先读右表后读左表。而在大部分情况下是先读左表的。STRAIGHT_JOIN仅用于少数情况下的表连接性能优化，比如右表记录数目明显少于左表。

#### Mysql表连接的运算顺序
在MySQL 5.1版本中，INNER JOIN, CROSS JOIN, LEFT JOIN, 和RIGHT JOIN 比逗号表达式具有更高的优先级。
因此SQL1被解析成SQL3，而不是SQL2。

```mysql
SQL1 :　SELECT * FROM t1, t2 JOIN t3 ON (t1.i1 = t3.i3);  
SQL2 :　SELECT * FROM (t1, t2) JOIN t3 ON (t1.i1 = t3.i3);  
SQL3 :　SELECT * FROM t1, (t2 JOIN t3 ON (t1.i1 = t3.i3));  
```

因此会报错，找不到i1列。因此以后在写这样的查询的时候，最好写明白，不要省略括号，这样能避免很多错误。

#### 循环的自然连接
在MySQL 5.1版本中，SQL1等价于SQL3， 而在MySQL以前版本中，SQL1等价于SQL2。
```mysql
SQL1 : SELECT ... FROM t1 NATURAL JOIN t2 NATURAL JOIN t3;  
SQL2 : SELECT ... FROM t1, t2, t3 WHERE t1.b = t2.b AND t2.c = t3.c;  
SQL3 : SELECT ... FROM t1, t2, t3 WHERE t1.b = t2.b AND t2.c = t3.c AND t1.a = t3.a; 
```

> * 一句话： 连接分为内连接，和外连接。（本来还有第三类交叉连接，交叉连接就是直接求笛卡尔积，在MySQL中等同于内连接）。内连接中还有一种自然连接，相当于不用加where语句，可以直接通过列名自动连接起来。外连接也有左外自然连接（natural left join）。

![demo of sql join](sqljoin.jpg)

注意：mysql貌似不支持全外链接，可以只用left join union right join来实现full outer join。



[1]: http://dev.mysql.com/doc/refman/5.7/en/join.html

---
title: mysql problems in leetcode
date: 2017-04-24 17:29:44
categories:
 - IT Technology
tags:
 - MySQL
---

### Questions and answers
The problems are all under: [leetcode database section](https://leetcode.com/problemset/database/)
And my solutions are here: [my solution](https://github.com/taylortao/myLeetcode/tree/master/Database)

### 177 - Nth Highest Salary   
For question 177, I need to know how to use mysql function.
1. Delete exists same function
`mysql> drop function if exists getNthHighestSalary;`

2. change default delimiter
Since the default delimiter is `;`, change it to `^^`
`mysql> delimiter ^^`

<!-- more -->

3. Create function
```
mysql> create function getNthHighestSalary(N int) returns int
    -> begin
    -> declare k int;
    -> set k = n-1;
    -> return (
    -> select distinct Salary from Employee
    -> where Salary is not null
    -> order by Salary desc
    -> limit k, 1
    -> );
    -> end^^
Query OK, 0 rows affected (0.00 sec)
```

4. Reset delimiter to its default
`delimiter ;`

5. Try to use the new function
```
mysql> select getNthHighestSalary(1);
+------------------------+
| getNthHighestSalary(1) |
+------------------------+
|                    900 |
+------------------------+
1 row in set (0.00 sec)
```

And the answer of 177 would be
```
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  DECLARE k int;
  SET K = n-1;
  RETURN (
      SELECT DISTINCT Salary 
      FROM Employee 
      WHERE Salary IS NOT NULL ORDER BY Salary DESC LIMIT k, 1
  );
END
```

### 178 - Rank Scores
There are several ways to think about it.

The most straight forward:

```
select Score,
	(select count(distinct s2.Score)
	from Scores s2 where s2.Score > s1.Score)+1 as Rank 
from Scores s1 order by Score desc;
```

Use a counter:
```
set @rk = 0;
set @pv = -0.1;
select Score, Rank
from (select Score, @rk := @rk + (Score <> @pv) as Rank, 
	@pv:=Score as Ps from Scores order by Score desc) as RkTable;
```

Or 

```
select Score, Rank 
from (select Score, @rk := @rk + (Score <> @pv) as Rank, @pv:=Score as Ps 
	from Scores, (SELECT @rk := 0, @pv := -1) init order by Score desc) as RkTable;
```

Simpler

```
select Score, @rk := @rk + (@pv <> (@pv:=Score)) as Rank 
from Scores, (SELECT @rk := 0, @pv := -1) init order by Score desc;
```

### 180 - Consecutive Numbers

A wrong solution, do not forget there might have -1 in the table.
```
select Num as ConsecutiveNums 
from (select Num, @cnt:=(Num=@pv)*(@cnt + 1) as Count, @pv:= Num 
	from Logs, (select @pv:=-1, @cnt:=0) init) as CntTable 
group by Num having max(Count)= 2;
```
Failed case:
```
[1, -1]
[2, -1]
[3, -1]
```
Add another parameter:

```
select Num as ConsecutiveNums 
from (select Num, @cnt:=(@cnt>=0)*(Num=@pv)*(@cnt + 1) as Count, @pv:= Num 
	from Logs, (select @pv:=-1, @cnt:=-1) init) as CntTable 
group by Num having max(Count)= 2;
```

Wow, it needs the nums that is emerges more than 3 times, not exactly 3 times.

```
select distinct Num as ConsecutiveNums 
from (select Num, @cnt:=(@cnt>=0)*(Num=@pv)*(@cnt + 1) as Count, @pv:= Num 
	from Logs, (select @pv:=-1, @cnt:=-1) init) as CntTable 
where Count=2
```

### 196 - Delete Duplicate Emails   

```
drop temporary table if exists t1;
create temporary table t1 (select min(Id) as Id from Person group by Email);
delete from Person where Id in (select * from t1);   
```

It will report:
`Commands out of sync; you can't run this command now`

Change it to: 

```
delete from Person 
where Id not in 
	(select * from (select min(Id) as Id from Person group by Email) as pd);
```

### 197 - Rising Temperature

My solution
```
select Id 
from (select Id, Date_Add(Date, Interval -1 day) as tyd, @pv as yt, 
	@pv:=Temperature as tt, @dt as yd, @dt:=Date as td 
	from Weather, 
	(select @pv:=1000, @dt:='0-0-0') init order by Date) as weathertable 
where tyd=yd and tt > yt;
```
And I found another solution, it is really simple and clear:
```
select w1.Id as Id 
from Weather w1 , Weather w2 
where w1.Temperature > w2.Temperature and To_Days(w1.Date) - To_Days(w2.Date) =1
```

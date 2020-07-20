### 176 第二高的薪水 Second Highest Salary

#### 解法一 表自连接

用表的自连接，构造偏序关系。再找次序的最大值，就一定是第二高的薪水。同时，max在没有元组输入时，还能返回NULL。比如在表中的元组少于2个时。

```mysql
SELECT MAX(l1.salary) as SecondHighestSalary
FROM Employee l1 JOIN Employee l2 
WHERE l1.salary < l2.salary;
```

[MAX函数](http://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html)

```
Syntax:
MAX([DISTINCT] expr) [over_clause]

Returns the maximum value of expr. MAX() may take a string argument; in
such cases, it returns the maximum string value. See
http://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html. The DISTINCT
keyword can be used to find the maximum of the distinct values of expr,
however, this produces the same result as omitting DISTINCT.

If there are no matching rows, MAX() returns NULL.

This function executes as a window function if over_clause is present.
over_clause is as described in
http://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html; it cannot be used with DISTINCT.
URL: http://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html
```

#### 解法二 子查询

用子查询找出最大值，再排除最大值的结果中，求最大值。

借助于MAX函数。

```mysql
select max(salary) as SecondHighestSalary
from Employee
where salary !=(select max(salary) from Employee);

select max(E.Salary) as SecondHighestSalary
from Employee as E
where E.Salary < (
select max(Salary)
from Employee 
);

select max(E1.Salary) as SecondHighestSalary
from Employee as E1
where 1 = (
	select count(*)
	from Employee as E2
	where E1.Salary < E2.Salary
);
```

也可以使用order+limit。

```mysql
SELECT
(SELECT DISTINCT Salary
 FROM Employee  
 ORDER BY Salary DESC    
 LIMIT 1 OFFSET 1) AS SecondHighestSalary;
```


### 185 部门工资前三高的所有员工 Department Top Three Salaries

每个部门都有工资最高的一批人，工资次高的一批人，工资再次高的一人。输出这三批人。

#### 解法一

判断每个人A是不是在这三批人中的一个。找出同一部门种比A薪水高的薪水种数N。用子查询完成。如果N<=2，那么A属于这三批人。

```mysql
select D.name as `Department`,E.name as Employee`,E.Salary
from Employee as E join Department as D on (E.departmentid = D.id)
where (
    select count(distinct E1.salary)
    from Employee as E1
    where E1.departmentid = E.departmentid and E1.salary > E.salary
) <=2
```

#### 解法二

先找出每个部门薪水第三高的薪水A。每个人的薪水只要大于等于A，他肯定在这三批人中。

员工表：

```
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Randy    | 85000  |
| IT         | Joe      | 85000  |
| IT         | Will     | 70000  |
| Sales      | Henry    | 80000  |
| Sales      | Sam      | 60000  |
+------------+----------+--------+
```

对员工表三次left join。方法如下：

```mysql
SELECT *
FROM Employee e1
LEFT JOIN Employee e2 ON(e1.DepartmentId=e2.DepartmentId AND e1.Salary>e2.Salary)
LEFT JOIN Employee e3 ON(e2.DepartmentId=e3.DepartmentId AND e2.Salary>e3.Salary)
```

**注意**：上面的小于号的顺序，e1.Salary>e2.Salary>e3.Salary。这个顺序非常得重要。

得出这样的结果：

```mysql
+------+-------+--------+--------------+------+-------+--------+--------------+------+-------+--------+--------------+
| Id   | Name  | Salary | DepartmentId | Id   | Name  | Salary | DepartmentId | Id   | Name  | Salary | DepartmentId |
+------+-------+--------+--------------+------+-------+--------+--------------+------+-------+--------+--------------+
|    4 | Max   |  90000 |            1 |    1 | Joe   |  85000 |            1 |    5 | Janet |  69000 |            1 |
|    4 | Max   |  90000 |            1 |    6 | Randy |  85000 |            1 |    5 | Janet |  69000 |            1 |
|    1 | Joe   |  85000 |            1 |    7 | Will  |  70000 |            1 |    5 | Janet |  69000 |            1 |
|    4 | Max   |  90000 |            1 |    7 | Will  |  70000 |            1 |    5 | Janet |  69000 |            1 |
|    6 | Randy |  85000 |            1 |    7 | Will  |  70000 |            1 |    5 | Janet |  69000 |            1 |
|    4 | Max   |  90000 |            1 |    1 | Joe   |  85000 |            1 |    7 | Will  |  70000 |            1 |
|    4 | Max   |  90000 |            1 |    6 | Randy |  85000 |            1 |    7 | Will  |  70000 |            1 |
|    2 | Henry |  80000 |            2 |    3 | Sam   |  60000 |            2 | NULL | NULL  |   NULL |         NULL |
|    1 | Joe   |  85000 |            1 |    5 | Janet |  69000 |            1 | NULL | NULL  |   NULL |         NULL |
|    4 | Max   |  90000 |            1 |    5 | Janet |  69000 |            1 | NULL | NULL  |   NULL |         NULL |
|    6 | Randy |  85000 |            1 |    5 | Janet |  69000 |            1 | NULL | NULL  |   NULL |         NULL |
|    7 | Will  |  70000 |            1 |    5 | Janet |  69000 |            1 | NULL | NULL  |   NULL |         NULL |
|    3 | Sam   |  60000 |            2 | NULL | NULL  |   NULL |         NULL | NULL | NULL  |   NULL |         NULL |
|    5 | Janet |  69000 |            1 | NULL | NULL  |   NULL |         NULL | NULL | NULL  |   NULL |         NULL |
+------+-------+--------+--------------+------+-------+--------+--------------+------+-------+--------+--------------+
```

从结果中发现，求第三高的薪水，只能在e3.Salary上求max。且要处理 e3.Salary 为null的情况。如果在 e1.Salary 上求max，得到的一定是每个部门的最高薪水。因此left join 左边的表的所有元组必然在结果中。

用[CASE WHEN END](http://www.mysqltutorial.org/mysql-case-function/)子句，对null字段进行处理。

在这里当值为NULL时，将其替换为0。

```mysql
SELECT e1.DepartmentId, 
CASE 
	WHEN MAX(e3.salary) IS NULL THEN 0 
	ELSE MAX(e3.salary) 
END AS max_salary
FROM Employee e1
LEFT JOIN Employee e2 ON(e1.DepartmentId=e2.DepartmentId AND e1.Salary>e2.Salary)
LEFT JOIN Employee e3 ON(e2.DepartmentId=e3.DepartmentId AND e2.Salary>e3.Salary)
GROUP BY e1.DepartmentId
```

得出结果：

```mysql
+--------------+------------+                          | DepartmentId | max_salary |                          +--------------+------------+                         |            1 |      70000 |                          |            2 |          0 |                          +--------------+------------+        
```

上面说顺序很重要，那么如果将上面<顺序，改为e1.Salary**<**e2.Salary<e3.Salary。 再求e1.Salary的max，得出的值一定时每个部门**最高的薪水，而不是第三高的薪水**。因为left join**最左边的表的元组一定全部都在！**

将上面的结果与员工表和部门表，再连接起来，得出结果。

```mysql
SELECT D.name AS `Department`,E.name AS `Employee`,E.Salary
FROM Employee AS E
JOIN Department AS D ON (E.departmentid = D.id)
JOIN (
	SELECT e1.DepartmentId, CASE WHEN MAX(e3.salary) IS NULL THEN 0 ELSE MAX(e3.salary) END AS m
	FROM Employee e1
	LEFT JOIN Employee e2 ON(e1.DepartmentId=e2.DepartmentId AND e1.Salary>e2.Salary)
	LEFT JOIN Employee e3 ON(e2.DepartmentId=e3.DepartmentId AND e2.Salary>e3.Salary)
	GROUP BY e1.DepartmentId
) AS F ON (E.departmentid = F.departmentid AND E.salary >= F.m)
```

#### 解法三

延续解法二的思路。 应用[用户变量](https://dev.mysql.com/doc/refman/8.0/en/user-variables.html)，求每个部门第三高薪水。

定义三个用户变量：@pre_salary——上一行的薪水；@pre_deptid——上一行的部门id；@salary_cnt——第几种薪水。

```
SELECT @pre_salary:= NULL, @pre_deptid:= NULL, @salary_cnt:=0;
```

其初始值如下，构成了一个表。

```mysql
+--------------------+--------------------+----------------+
| @pre_salary:= NULL | @pre_deptid:= NULL | @salary_cnt:=0 |
+--------------------+--------------------+----------------+
| NULL               | NULL               |              0 |
+--------------------+--------------------+----------------+
```

将这样的表命名为A。

```mysql
(
SELECT @pre_salary:= NULL, @pre_deptid:= NULL, @salary_cnt:=0
) AS A
```

先看将员工表与表A叉积。

```mysql
select *
FROM Employee t, 
(
	SELECT @pre_salary:= NULL, @pre_deptid:= NULL, @salary_cnt:=0
) 
AS A
```

结果为

```
+------+-------+--------+--------------+--------------------+--------------------+----------------+
| Id   | Name  | Salary | DepartmentId | @pre_salary:= NULL | @pre_deptid:= NULL | @salary_cnt:=0 |
+------+-------+--------+--------------+--------------------+--------------------+----------------+
|    1 | Joe   |  85000 |            1 | NULL               | NULL               |              0 |
|    2 | Henry |  80000 |            2 | NULL               | NULL               |              0 |
|    3 | Sam   |  60000 |            2 | NULL               | NULL               |              0 |
|    4 | Max   |  90000 |            1 | NULL               | NULL               |              0 |
|    5 | Janet |  69000 |            1 | NULL               | NULL               |              0 |
|    6 | Randy |  85000 |            1 | NULL               | NULL               |              0 |
|    7 | Will  |  70000 |            1 | NULL               | NULL               |              0 |
+------+-------+--------+--------------+--------------------+--------------------+----------------+
```

但这样的结果，对找每个部门的不同薪水没有帮助。要对结果按部门id递增排序，再按薪水降序。

```mysql
select *
FROM Employee t, 
(
	SELECT @pre_salary:= NULL, @pre_deptid:= NULL, @salary_cnt:=0
) 
AS A
ORDER BY t.DepartmentId, t.Salary DESC
```

排序后

```
+------+-------+--------+--------------+--------------------+--------------------+----------------+
| Id   | Name  | Salary | DepartmentId | @pre_salary:= NULL | @pre_deptid:= NULL | @salary_cnt:=0 |
+------+-------+--------+--------------+--------------------+--------------------+----------------+
|    4 | Max   |  90000 |            1 | NULL               | NULL               |              0 |
|    1 | Joe   |  85000 |            1 | NULL               | NULL               |              0 |
|    6 | Randy |  85000 |            1 | NULL               | NULL               |              0 |
|    7 | Will  |  70000 |            1 | NULL               | NULL               |              0 |
|    5 | Janet |  69000 |            1 | NULL               | NULL               |              0 |
|    2 | Henry |  80000 |            2 | NULL               | NULL               |              0 |
|    3 | Sam   |  60000 |            2 | NULL               | NULL               |              0 |
+------+-------+--------+--------------+--------------------+--------------------+----------------+
```

从这个结果中找出，每个部门的不同薪水个数。

对结果中的每行，执行下述逻辑，计数。

```mysql
if (当前行的salary = @pre_salary and 当前行的departmentid = @pre_deptid)
{
    //相同的薪水
    @salary_cnt 不变
}
else if(当前行的departmentid = @pre_deptid)
{
    //不同的薪水
    @salary_cnt = @salary_cnt + 1
}
else
{
    //不同的部门，计数重新从1开始
    @salary_cnt = 1
}
//更新pre_salary和pre_deptid
@pre_salary = 当前行的salary
@pre_deptid = 当前行的departmentid
```

将此逻辑翻译为SQL代码，结果命令为表B

```mysql
(
	SELECT
	@salary_cnt:= IF(t.Salary = @pre_salary AND t.DepartmentId = @pre_deptid, 
		        @salary_cnt, 
			IF(t.DepartmentId = @pre_deptid, 
                             @salary_cnt + 1, 
                             1)
		) 
		AS `cnt`
	,@pre_salary := t.Salary AS `salary`
	,@pre_deptid := t.DepartmentId AS `deptid`
	FROM Employee t, 
	(
		SELECT @pre_salary:= NULL, @pre_deptid:= NULL, @salary_cnt:=0
	) 
	AS A
	ORDER BY t.DepartmentId, t.Salary DESC
) 
AS B
```

结果为：

```
+------+--------+--------+
| cnt  | salary | deptid |
+------+--------+--------+
|    1 |  90000 |      1 |
|    2 |  85000 |      1 |
|    2 |  85000 |      1 |
|    3 |  70000 |      1 |
|    4 |  69000 |      1 |
|    1 |  80000 |      2 |
|    2 |  60000 |      2 |
+------+--------+--------+
```

从表B中，选出每个部门中前三种薪水，取最小的薪水。结果命名为C。

```mysql
(
SELECT B.deptid,MIN(B.salary) AS `salary`
FROM 
	(
		SELECT
		@salary_cnt:= IF(t.Salary = @pre_salary AND t.DepartmentId = @pre_deptid, 
			@salary_cnt, 
				IF(t.DepartmentId = @pre_deptid, @salary_cnt + 1, 1)
			) 
			AS `cnt`
		,@pre_salary := t.Salary AS `salary`
		,@pre_deptid := t.DepartmentId AS `deptid`
		FROM Employee t, 
		(
			SELECT @pre_salary:= NULL, @pre_deptid:= NULL, @salary_cnt:=0
		) 
		AS A
		ORDER BY t.DepartmentId, t.Salary DESC
	) 
	AS B
WHERE B.cnt < 4
GROUP BY B.deptid
) AS C
```

结果为：

```
+--------+--------+
| deptid | salary |
+--------+--------+
|      1 |  70000 |
|      2 |  60000 |
+--------+--------+
```

再将表C与员工表和部门表连接，所有薪水大于等于C.salary都取出来。

```mysql
SELECT D.name AS `Department`,E.name AS `Employee`,E.Salary
FROM Employee AS E
JOIN Department AS D ON (E.departmentid = D.id)
JOIN 
	(
	SELECT B.deptid,MIN(B.salary) AS `salary`
	FROM 
		(
			SELECT
			@salary_cnt:= IF(t.Salary = @pre_salary AND t.DepartmentId = @pre_deptid, 
				@salary_cnt, 
					IF(t.DepartmentId = @pre_deptid, @salary_cnt + 1, 1)
				) 
				AS `cnt`
			,@pre_salary := t.Salary AS `salary`
			,@pre_deptid := t.DepartmentId AS `deptid`
			FROM Employee t, 
			(
				SELECT @pre_salary:= NULL, @pre_deptid:= NULL, @salary_cnt:=0
			) 
			AS A
			ORDER BY t.DepartmentId, t.Salary DESC
		) 
		AS B
	WHERE B.cnt < 4
	GROUP BY B.deptid
	) AS C
	ON (D.id = C.deptid AND E.salary >= C.salary)
```

#### 解法四

计算每个员工的薪水，在其所在部门的薪水中的排名。最后取出每个部门中排名前三的员工。内在的逻辑与解法三殊途同归，但是具体的方法不同。

先取出每个部门不同薪水，并按部门id升序，薪水降序。结果命名为表A。

```mysql
(
SELECT DepartmentId,Salary
FROM Employee
GROUP BY DepartmentId,Salary
ORDER BY DepartmentId,Salary DESC
)
AS A
```

结果为：

```
+--------------+--------+
| DepartmentId | Salary |
+--------------+--------+
|            1 |  90000 |
|            1 |  85000 |
|            1 |  70000 |
|            1 |  69000 |
|            2 |  80000 |
|            2 |  60000 |
+--------------+--------+
```

在表A上求出每个薪水的排名。也是借助于**用户变量**：@pre_deptid——上一行的部门id，@rank——此行的排名。

```mysql
(SELECT @pre_deptid:=null, @rank:=0) AS B
```

叉积A和B，执行下面的逻辑计算@rank。

```mysql
if (@pre_deptid = 当前行的departmentid)
{
    //同一部门
    @rank = @rank + 1
}
else
{
    //不同部门。@rank重置为1
    @rank=1
}
```

逻辑转换为：

```mysql
CASE 
	WHEN @pre_deptid = DepartmentId THEN @rank:= @rank + 1
	WHEN @pre_deptid := DepartmentId THEN @rank:= 1
END AS `rank`
```

薪水排名逻辑，结果命名为

```mysql
(
SELECT A.DepartmentId, A.Salary,
	   CASE 
			WHEN @pre_deptid = DepartmentId THEN @rank:= @rank + 1
			WHEN @pre_deptid := DepartmentId THEN @rank:= 1
	   END AS `rank`
FROM (SELECT @pre_deptid:=null, @rank:=0) 
	   AS B,
	 (
		 SELECT DepartmentId,Salary
		 FROM Employee
		 GROUP BY DepartmentId,Salary
		 ORDER BY DepartmentId,Salary DESC
	 ) 
		 AS A
)
AS C
```

结果为

```mysql
+--------------+--------+------+
| DepartmentId | Salary | rank |
+--------------+--------+------+
|            1 |  90000 |    1 |
|            1 |  85000 |    2 |
|            1 |  70000 |    3 |
|            1 |  69000 |    4 |
|            2 |  80000 |    1 |
|            2 |  60000 |    2 |
+--------------+--------+------+
```

再将表C与员工表和部门表连接，取出排名小于等于3的员工。

```mysql
SELECT D.Name as Department, E.NAME  AS Employee, E.Salary
FROM (
        SELECT A.DepartmentId, A.Salary,
               CASE 
                    WHEN @pre_deptid = DepartmentId THEN @rank:= @rank + 1
                    WHEN @pre_deptid := DepartmentId THEN @rank:= 1
               END AS `rank`
        FROM (SELECT @pre_deptid:=null, @rank:=0) 
		       AS B,
             (
                 SELECT DepartmentId,Salary
                 FROM Employee
                 GROUP BY DepartmentId,Salary
                 ORDER BY DepartmentId,Salary DESC
             ) 
				 AS A
       )
       AS C
INNER JOIN Department AS D ON C.DepartmentId = D.Id
INNER JOIN Employee AS E ON C.DepartmentId = E.DepartmentId AND C.Salary = E.Salary AND C.rank <= 3
```
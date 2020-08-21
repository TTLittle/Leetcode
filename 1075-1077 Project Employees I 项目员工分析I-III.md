## 1075-1077 Project Employees I 项目员工分析I-III

### 1075Project Employees I 项目员工分析I

#### I 分析每个项目员工的平均工作年龄。

连接项目表和员工表，对项目分组，求员工工作年龄的平均值。

用group by 和 avg。

```mysql
select P.project_id,ROUND(avg(E.experience_years),2) as `average_years`
from Project as P 
join Employee as E
    on(P.employee_id = E.employee_id )
group by P.project_id
```

### 1076 Project Employees II 项目员工分析II

#### II 找出员工数最多的项目，可能有多个

##### 解法一

先算出每个项目的员工数。应用group by分组。

```mysql
(
    SELECT P.project_id,count(distinct P.employee_id) as `cnt`
    from Project as P
    group by P.project_id
) AS A
```

再求所有项目中最多的人数。

在上面基础上，降序排，取第一个。

```mysql
(
    SELECT count(distinct P.employee_id) as `cnt`
    from Project as P
    group by P.project_id
    order by cnt desc
    limit 0,1
) AS B
```

连接表A和表B，选出员工数等于最大值的项目。

```mysql
SELECT A.project_id
FROM 
(
    SELECT P.project_id,count(distinct P.employee_id) as `cnt`
    from Project as P
    group by P.project_id
) AS A
JOIN 
(
    SELECT count(distinct P.employee_id) as `cnt`
    from Project as P
    group by P.project_id
    order by cnt desc
    limit 0,1
) AS B
	ON(A.cnt = B.cnt)
GROUP BY A.project_id
```

##### 解法二

先求最多的人数。

逻辑同解法一。

```mysql
(
    SELECT count(distinct P.employee_id) as `cnt`
    from Project as P
    group by P.project_id
    order by cnt desc
    limit 0,1
) AS B
```

项目表与表B叉积，对项目分组，计算每个组的人数，选出每组人数等于最多人数的行。

```mysql
SELECT A.project_id
from Project AS A,
(
    SELECT count(distinct P.employee_id) as `cnt`
    from Project as P
    group by P.project_id
    order by cnt desc
    limit 0,1
) AS B
GROUP BY A.project_id
having COUNT(distinct A.employee_id) = MAX(B.cnt)
```

**注意**：having子句中用到MAX(B.cnt)。由于group by只对A.project_id分组。B.cnt不能直接用在“=”之后，尽管每行的B.cnt相等。需要将B.cnt包装在聚集函数中。此处选了MAX。或者将group by改为对A.project_id和B.cnt分组，B.cnt就可以直接用在having子句中。

```mysql
SELECT A.project_id
from Project AS A,
(
    SELECT count(distinct P.employee_id) as `cnt`
    from Project as P
    group by P.project_id
    order by cnt desc
    limit 0,1
) AS B
GROUP BY A.project_id,B.cnt
having COUNT(distinct A.employee_id) = B.cnt
```

### 1077 Project Employees III 项目员工分析III

#### III 每个项目中经验最高的员工

```
Project table:
+-------------+-------------+
| project_id  | employee_id |
+-------------+-------------+
| 1           | 1           |
| 1           | 2           |
| 1           | 3           |
| 2           | 1           |
| 2           | 4           |
+-------------+-------------+

Employee table:
+-------------+--------+------------------+
| employee_id | name   | experience_years |
+-------------+--------+------------------+
| 1           | Khaled | 3                |
| 2           | Ali    | 2                |
| 3           | John   | 3                |
| 4           | Doe    | 2                |
+-------------+--------+------------------+

Result table:
+-------------+---------------+
| project_id  | employee_id   |
+-------------+---------------+
| 1           | 1             |
| 1           | 3             |
| 2           | 1             |
+-------------+---------------+
```

##### 解法一

先算出每个项目最高的经验年份。

连接项目表和员工表，对项目group by，求最高年份。

结果命名为表A。

```mysql
(
    select P.project_id,max(E.experience_years) as `most`
    from Project as P join Employee as E on (P.employee_id = E.employee_id)
    group by P.project_id
) as A
```

再连接表A，项目表和员工表，求每个项目中，与最高年份相等的员工

```mysql
select P.project_id,P.employee_id
from Project as P
join
(
    select P.project_id,max(E.experience_years) as `most`
    from Project as P join Employee as E on (P.employee_id = E.employee_id)
    group by P.project_id
) as A
    on (P.project_id = A.project_id)
join Employee as E
    on (P.employee_id = E.employee_id and A.most = E.experience_years)
```

##### 解法二

对每个项目中的每个人，判断其是否是最高年份的员工。

求每个项目的最高年份，方法同解法一。

```mysql
select max(E1.experience_years) as `most`
from Project as P1 join Employee as E1
on P1.employee_id = E1.employee_id
group by P1.project_id
```

判断每个人是否是项目中的年份最高的人。应用IN。

```mysql
select P.project_id,E.employee_id
from Project as P join Employee as E
on P.employee_id = E.employee_id
where E.experience_years in (
    select max(E1.experience_years) as `most`
    from Project as P1 join Employee as E1
    on P1.employee_id = E1.employee_id
    where P1.project_id = P.project_id
    group by P1.project_id
)
```
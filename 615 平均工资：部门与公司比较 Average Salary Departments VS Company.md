### 615 平均工资：部门与公司比较 Average Salary: Departments VS Company

思路有些复杂，需要先理清楚再动手。

**注意**：pay_date存在同一个月但不同天的情况。

每月公司平均工资是每月公司中所有员工工资的平均值。

连接表salary和表employee，按年月分组，计算**每组工资的平均值**。

结果命名为表A。

```mysql
(
	SELECT DATE_FORMAT(S.pay_date,'%Y-%m') AS `pay_month`, AVG(S.amount) AS `avg_salary`
	FROM salary AS S
	JOIN employee AS E ON (S.employee_id = E.employee_id)
	GROUP BY `pay_month`
) AS A
```

[DATA_FORMAT](http://www.mysqltutorial.org/mysql-date_format/)函数，用于格式化日期。

每月每部门平均工资是每月每部门中所有员工工资的平均值。

连接表salary和表employee，按部门和年月分组，计算每组员工的平均值。

结果命名为表B。

```mysql
(
	SELECT E.department_id,DATE_FORMAT(S.pay_date,'%Y-%m') AS `pay_month`, AVG(S.amount) AS `dept_avg_salary`
	FROM salary AS S
	JOIN employee AS E ON (S.employee_id = E.employee_id)
	GROUP BY E.department_id,`pay_month`
) AS B 
```

连接表A和表B，比较相同年月中，部门平均工资与公司平均工资。

```mysql
IF(
	B.dept_avg_salary > A.avg_salary,
	'higher', IF(B.dept_avg_salary < A.avg_salary,
		'lower',
		'same'
	)
) AS `comparison`
```

或者用[CASE](http://www.mysqltutorial.org/mysql-case-function/)子句。

```mysql
case
	when B.dept_avg_salary > A.avg_salary then 'higher'
	when B.dept_avg_salary < A.avg_salary then 'lower'
	ELSE 'same'
END AS `comparison`
```

综合以上逻辑：

```mysql
SELECT B.pay_month,
B.department_id, 
IF(
	B.dept_avg_salary > A.avg_salary,
	'higher', IF(B.dept_avg_salary < A.avg_salary,
		'lower',
		'same'
	)
) AS `comparison`
FROM 
(
	SELECT DATE_FORMAT(S.pay_date,'%Y-%m') AS `pay_month`, AVG(S.amount) AS `avg_salary`
	FROM salary AS S
	JOIN employee AS E ON (S.employee_id = E.employee_id)
	GROUP BY `pay_month`
) AS A
JOIN 
(
	SELECT E.department_id,DATE_FORMAT(S.pay_date,'%Y-%m') AS `pay_month`, AVG(S.amount) AS `dept_avg_salary`
	FROM salary AS S
	JOIN employee AS E ON (S.employee_id = E.employee_id)
	GROUP BY E.department_id,`pay_month`
) AS B 
ON (A.pay_month = B.pay_month)
```

或

```mysql
SELECT B.pay_month,
B.department_id, 
case
	when B.dept_avg_salary > A.avg_salary then 'higher'
	when B.dept_avg_salary < A.avg_salary then 'lower'
	ELSE 'same'
END AS `comparison`
FROM 
(
	SELECT DATE_FORMAT(S.pay_date,'%Y-%m') AS `pay_month`, AVG(S.amount) AS `avg_salary`
	FROM salary AS S
	JOIN employee AS E ON (S.employee_id = E.employee_id)
	GROUP BY `pay_month`
) AS A
JOIN 
(
	SELECT E.department_id,DATE_FORMAT(S.pay_date,'%Y-%m') AS `pay_month`, AVG(S.amount) AS `dept_avg_salary`
	FROM salary AS S
	JOIN employee AS E ON (S.employee_id = E.employee_id)
	GROUP BY E.department_id,`pay_month`
) AS B 
ON (A.pay_month = B.pay_month)
```
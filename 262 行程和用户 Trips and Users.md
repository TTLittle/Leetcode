### 262 行程和用户 Trips and Users

从行程表Trips和用户表Users中，统计每天非禁止用户的取消率，需要知道非禁止用户有哪些，总行程数，取消的行程数。

#### 解法一

首先确定被禁止用户的行程记录，再剔除这些行程记录。

行程表中，字段client_id和driver_id，都与用户表中的users_id关联。因此只要client_id和driver_id中有一个**被禁止**了，此条行程记录要**被剔除**。

**client_id与driver_id不一定相同** ，正确的做法是对client_id和driver_id各自关联的users_id，**同时检测**是否被禁止。

```
if (client_id = users_id_1 且 users_id_1没被禁止 并且 client_id = users_id_2 且 users_id_2没被禁止){
    此条记录没被禁止。
}
```

SQL代码：

```mysql
SELECT *
FROM Trips AS T
JOIN Users AS U1 ON (T.client_id = U1.users_id AND U1.banned ='No')
JOIN Users AS U2 ON (T.driver_id = U2.users_id AND U2.banned ='No')
```

在此基础上，按日期分组，统计每组的 总行程数，取消的行程数 。

每组的总行程数：COUNT(T.STATUS)。

每组的取消的行程数：

```mysql
SUM(
	IF(T.STATUS = 'completed',0,1)
)
```

**取消率** = 每组的取消的行程数 / 每组的总行程数

完整逻辑为:

```mysql
SELECT T.request_at AS `Day`, 
	ROUND(
			SUM(
				IF(T.STATUS = 'completed',0,1)
			)
			/ 
			COUNT(T.STATUS),
			2
	) AS `Cancellation Rate`
FROM Trips AS T
JOIN Users AS U1 ON (T.client_id = U1.users_id AND U1.banned ='No')
JOIN Users AS U2 ON (T.driver_id = U2.users_id AND U2.banned ='No')
WHERE T.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.request_at
```

其中[SUM求和函数，COUNT计数函数](http://www.mysqltutorial.org/mysql-aggregate-functions.aspx)，[ROUND四舍五入函数](http://www.mysqltutorial.org/mysql-math-functions/mysql-round/)。

#### 解法二

思路与解法一相同。而采用不同的方法排除掉被禁止用户的行程记录。想到排除，就联想到集合差。

client_id和driver_id的全部为集合U。被禁止的users_id集合为A。

U减去A的结果为**没被禁止的用户**。

行程表left join 表A两次，A.users_id都为NULL的行都是没被排除的行。

```mysql
SELECT *
FROM trips AS T LEFT JOIN 
(
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
) AS A ON (T.Client_Id = A.users_id)
LEFT JOIN (
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
) AS A1
ON (T.Driver_Id = A1.users_id)
WHERE A.users_id IS NULL AND A1.users_id IS NULL
```

补上其它部分的逻辑为：

```mysql
SELECT T.request_at AS `Day`, 
	ROUND(
			SUM(
				IF(T.STATUS = 'completed',0,1)
			)
			/ 
			COUNT(T.STATUS),
			2
	) AS `Cancellation Rate`
FROM trips AS T LEFT JOIN 
(
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
) AS A ON (T.Client_Id = A.users_id)
LEFT JOIN (
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
) AS A1
ON (T.Driver_Id = A1.users_id)
WHERE A.users_id IS NULL AND A1.users_id IS NULL AND T.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.request_at
```

## 解法三

与解法二思路相同。找出被禁止的用户后，不再连接行程表和用户表，直接从行程表中排除掉被被禁止用户的行程记录。

被禁止的用户用子查询：

```mysql
(
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
)
```

行程表中client_id和driver_id都在此子查询结果中的行要剔除掉。

```mysql
SELECT *
FROM trips AS T
WHERE 
T.Client_Id NOT IN (
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
)
AND
T.Driver_Id NOT IN (
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
)
```

补上其它部分：

```mysql
SELECT T.request_at AS `Day`, 
	ROUND(
			SUM(
				IF(T.STATUS = 'completed',0,1)
			)
			/ 
			COUNT(T.STATUS),
			2
	) AS `Cancellation Rate`
FROM trips AS T
WHERE 
T.Client_Id NOT IN (
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
)
AND
T.Driver_Id NOT IN (
	SELECT users_id
	FROM users
	WHERE banned = 'Yes'
)
AND T.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY T.request_at
```
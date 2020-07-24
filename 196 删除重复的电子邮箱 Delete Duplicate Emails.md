### 196 删除重复的电子邮箱 Delete Duplicate Emails

#### 解法一

按email分组，找到每组id最小的行。

```mysql
(
SELECT MIN(P.id) AS `Id`,P.Email
FROM Person AS P
GROUP BY P.email
) AS P2
```

从原表中**[DELETE](http://www.mysqltutorial.org/mysql-delete-join/)**掉不在表2中的行。

```mysql
DELETE P1
FROM Person AS P1,
(
SELECT MIN(P.id) AS `Id`,P.Email
FROM Person AS P
GROUP BY P.email
) AS P2
WHERE P1.Id != P2.Id AND P1.Email = P2.Email
```

**注意**：**DELETE**与**FROM**之间，只放置了P1。说明只删除P1中的行，不删除P2中的行。

FROM后，P1和P2叉积。当然也可以应用内连接。

```mysql
DELETE P1
FROM Person AS P1
JOIN 
(
SELECT MIN(P.id) AS `Id`,P.Email
FROM Person AS P
GROUP BY P.email
) AS P2
ON (P1.Id != P2.Id AND P1.Email = P2.Email)
```

#### 解法二

既然DELETE 可以结合JOIN，直接表自连接，删除所有比当前行ID大的行。

```mysql
DELETE P1
FROM Person AS P1 JOIN Person AS P2 ON (P1.email = P2.email AND P1.id > P2.id)
```

#### 解法三

从集合的角度看。按email分组，找到每组id最小的行。 命名为集合A。那么，从全集U中保留集合A，删除U减去A的差集。再删除差集数据。应用LEFT JOIN。

```mysql
DELETE U
FROM Person AS U
LEFT JOIN (
	SELECT MIN(id) AS `id`,email
	FROM Person
	GROUP BY email
) AS A ON (U.email = A.email AND U.id = A.id)
WHERE A.id IS NULL
```
### 601 体育馆的人流量 Human Traffic of Stadium

```
表 stadium：
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
+------+------------+-----------+
结果为：
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
+------+------------+-----------+
```

#### 解法一

先将哪些连续三天客流量都>=100的行找出来。

表三次连接。结果命名为表A。

```mysql
(
	SELECT S1.id AS `id1`,S2.id AS `id2`,S3.id AS `id3`
	FROM stadium AS S1
	JOIN stadium AS S2 ON(S1.id +1 = S2.id AND S1.people >=100 AND S2.people >=100)
	JOIN stadium AS S3 ON(S2.id +1 = S3.id AND S2.people >=100 AND S3.people >=100)
) AS A
```

那么，id1，id2，id3都是满足条件的行号。

与表Stadium连接。取出id1或id2或id3对应的行，但可能有些行重复取，要去重。

```mysql
SELECT DISTINCT S.*
FROM 
(
	SELECT S1.id AS `id1`,S2.id AS `id2`,S3.id AS `id3`
	FROM stadium AS S1
	JOIN stadium AS S2 
    ON(S1.id +1 = S2.id AND S1.people >=100 AND S2.people >=100)
	JOIN stadium AS S3 
    ON(S2.id +1 = S3.id AND S2.people >=100 AND S3.people >=100)
) AS A
JOIN stadium AS S ON(A.id1 = S.id OR A.id2=S.id OR A.id3=S.id)
```

#### 解法二（该思路略繁琐）

对每一行A，如果A的前两行的客流量都>=100，或者A的前一行或后一行的客流量都>=100，或者 A的后两行的客流量都>=100 。那么，行A就是结果中的一条。

```mysql
SELECT *
FROM stadium AS S1
WHERE S1.people >= 100 AND 
(
	2=(
		SELECT COUNT(*)
		FROM stadium AS S
		WHERE S.people >= 100 AND (S.id = S1.id-1 OR S.id = S1.id-2)
	) OR 
	2=(
		SELECT COUNT(*)
		FROM stadium AS S
		WHERE S.people >= 100 AND (S.id = S1.id-1 OR S.id = S1.id+1)
	) OR 
	2=(
		SELECT COUNT(*)
		FROM stadium AS S
		WHERE S.people >= 100 AND (S.id = S1.id+1 OR S.id = S1.id+2)
	)
)
```

A的前两行的客流量都>=100，逻辑如下：

```mysql
2=(
	SELECT COUNT(*)
	FROM stadium AS S
	WHERE S.people >= 100 AND (S.id = S1.id-1 OR S.id = S1.id-2)
)
```

A的前一行或后一行的客流量都>=100，逻辑如下：

```mysql
2=(
	SELECT COUNT(*)
	FROM stadium AS S
	WHERE S.people >= 100 AND (S.id = S1.id-1 OR S.id = S1.id+1)
)
```

A的后两行的客流量都>=100，逻辑如下：

```mysql
2=(
	SELECT COUNT(*)
	FROM stadium AS S
	WHERE S.people >= 100 AND (S.id = S1.id+1 OR S.id = S1.id+2)
)
```

#### 解法三

与解法一思路相近。

表自叉积三次。

筛选出客流量>=100，并且三个id依次加1的行。

```mysql
SELECT DISTINCT S1.*
FROM stadium AS S1,stadium AS S2,stadium AS S3
WHERE S1.people>=100 AND S2.people>=100 AND S3.people>=100 AND (
	S1.id +1 = S2.id AND S1.id+2=S3.id OR
	S1.id +1 = S2.id AND S1.id-1=S3.id OR
	S1.id -1 = S2.id AND S1.id-2=S3.id
)
ORDER BY S1.id
```
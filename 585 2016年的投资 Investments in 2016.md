### 585 2016年的投资 Investments in 2016

投保人在 2016 年成功投资的两个条件：

第一，*他在 2015 年的投保额 (TIV_2015) 至少跟一个其他投保人在 2015 年的投保额相同*。

要在2015年找到另一个人B，此人的投保额要与A相同。
第二，*他所在的城市必须与其他投保人都不同（也就是说维度和经度不能跟其他任何一个投保人完全相同）*。

还要与其它B都不在同一个城市。

#### 解法一

采用排除法。

对每个投保人A，如果同城存在另一个人B，则A和B都要排除。

SQL代码：

```mysql
exists(
    select *
    from insurance as B
    where B.PID != A.PID and B.LAT = A.LAT and B.LON = A.LON
)
```

如果没有一个人B，与A在2015的投保额相同，则A要排除。

```mysql
exists(
    select *
    from insurance as B
    where B.PID != A.PID and B.TIV_2015 = A.TIV_2015 
)
```

合起来排除掉无用的数据，再对2016的投保额求和。

```mysql
select sum(A.TIV_2016) as `TIV_2016`
from insurance as A
where exists(
    select *
    from insurance as B
    where B.PID != A.PID and B.TIV_2015 = A.TIV_2015 
) and not exists(
    select *
    from insurance as B
    where B.PID != A.PID and B.LAT = A.LAT and B.LON = A.LON
)
```

## 解法二

分三步解决。

第一步，找出所有**2015投保额相同**的人。

结果命名为表A。

```mysql
(
	SELECT DISTINCT A.*
	FROM 
	insurance AS A
	JOIN insurance AS B ON (B.PID != A.PID AND B.TIV_2015 = A.TIV_2015)
) AS A
```

第二步，找出所有**在同一个城市**的人。

结果命名为表B。

```mysql
(
	SELECT DISTINCT A.pid
	FROM 
	insurance AS A
	JOIN insurance AS B 
    ON (B.PID != A.PID AND B.LAT = A.LAT AND B.LON = A.LON)
) AS B
```

第三步，从表A中排除掉表B中的id，

使用典型的集合差，用left join连接表A和表B，找null，再求和。

```mysql
SELECT sum(A.TIV_2016) as `TIV_2016`
FROM 
(
	SELECT DISTINCT A.*
	FROM 
	insurance AS A
	JOIN insurance AS B ON (B.PID != A.PID AND B.TIV_2015 = A.TIV_2015)
) AS A
LEFT JOIN 
(
	SELECT DISTINCT A.pid
	FROM 
	insurance AS A
	JOIN insurance AS B ON (B.PID != A.PID AND B.LAT = A.LAT AND B.LON = A.LON)
) AS B
	ON (A.pid = B.pid)
WHERE B.pid IS NULL
```
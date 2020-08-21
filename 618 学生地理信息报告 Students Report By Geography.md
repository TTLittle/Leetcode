### 618 学生地理信息报告 Students Report By Geography

`student` 表

```
| name   | continent |
|--------|-----------|
| Jack   | America   |
| Pascal | Europe    |
| Xi     | Asia      |
| Jane   | America   |
```

在continent字段上构造**[透视表](https://zh.wikipedia.org/wiki/透视表)**。 使得每个学生按照姓名的字母顺序依次排列在对应的大洲下面。 期望的输出是：

```
| America | Asia | Europe |
|---------|------|--------|
| Jack    | Xi   | Pascal |
| Jane    |      |        |
```

分组+行转列

#### 解法一

按continent字段分组，聚合，组内排序。可得。

```mysql
+--------+-----------+
| name   | continent |
+--------+-----------+
| Jack   | America   |
| Jane   | America   |
| Xi     | Asia      |
| Pascal | Europe    |
+--------+-----------+
```

现在要**行转列**。

用America,Asia,Europe作为属性，Jack,Jane,Xi,Pascal作为值。

确定 Jack,Jane,Xi,Pascal 所在的**数据行**。

从 America 属性看，Jack在第1行，Jane在第2行；

从 Asia属性看，Xi在第1行。

从Europe属性看，Pascal在第1行。

最后一步，**确定每个数据行中的数据**。

按照 America,Asia,Europe 的顺序，

第1行：Jack Xi Pascal

第2行：Jane NULL NULL

合起来得出结果为：

```
| America | Asia | Europe |
|---------|------|--------|
| Jack    | Xi   | Pascal |
| Jane    |      |        |
```

从上面的思路看，**重点是两步**：

第一，确定每个属性中各个数据所在的数据行。 其实是求每个数据的排名。

第二，确定每个数据行都有哪些数据。按照排名放置数据。

**求每个数据的排名**

求排名通常有两种方法，表自连接和用户变量法。

**先讲表自连接法求排名**

本题的输入表中，可能会存在多行相同的(name,continent)。

比如：

```
...
(A,B)
(A,B)
(A,B)
...
```

那么，仅用两个字段，用表自连接，无法确定每个(A,B)的排名。

将数据的行号作为第三个字段，用于区分相同的数据。

定义用户变量：@row_id——数据的行号，从1开始。

```mysql
(SELECT @row_id:=0) AS T
```

给数字增加行号，结果表命名为S1：

```mysql
(
	SELECT 
        S.*,
        @row_id:=(@row_id + 1) AS `row_id`
	FROM student AS S,(SELECT @row_id:=0) AS T
) AS S1
```

S1表自连接算排名。

在同一个洲中，对每个人A，找出所有人B，满足条件：A.name > B.name 或 A.name = B.name 且 A.row_id > B.row_id。

意思是name字典序小的人排名在前，name字典序相同的人，行号小的排名在前。

S1表自连接也分join和left join。

用join，并且 “A.row_id > B.row_id ”改为“ A.row_id >= B.row_id ”。每个人都有排名，且排名从1开始。

用left join，每个人的排名从0开始。

此处用join，算排名的逻辑为：

```mysql
SELECT S1.continent,S1.NAME,S1.row_id,COUNT(*) AS `trank`
FROM 
(
	SELECT S.*,@row_id:=(@row_id + 1) AS `row_id`
	FROM student AS S,(SELECT @row_id:=0) AS T
) AS S1 
JOIN 
(
	SELECT S.*,@n_row_id:=(@n_row_id + 1) AS `n_row_id`
	FROM student AS S,(SELECT @n_row_id:=0) AS T
) AS S2 
ON (S1.continent = S2.continent AND (S1.NAME > S2.NAME OR (S1.NAME = S2.NAME AND S1.row_id >= S2.n_row_id)))
group BY S1.continent,S1.NAME,S1.row_id
order BY S1.continent,S1.NAME
```

尽管是S1表自连接，却引入了S2表。因为表名要唯一。

另外S2中的row_id也改为n_row_id。两者值相等。**由于mysql中**，S1表中的用户变量@row_id，会在表S2中**被共享**。如果在表S2中继续用@row_id表示行号，其值显然不对。才新增了变量@n_row_id作为行号。

此外，group by子句中，分组条件是：S1.continent,S1.NAME,S1.row_id。

因为要确定每个人的排名，分组依据应该是每个唯一的人。仅用 S1.continent,S1.NAME 不能唯一确定每个人，必须带上row_id。这才是前面引入row_id的意义。

**再讲用户变量法求排名**

用户变量法则相对简单。按continent升序，再按name升序。同一continent内，按name从小到大，排名从1开始。

用户变量：@trank——排名。@pre_con——前一行的continent。

排名逻辑如下，结果命名为表A

```mysql
(
	SELECT S.*,
	@trank:=if(@pre_con = S.continent,@trank + 1,1
	) AS `trank`,
	@pre_con:=S.continent AS `pre`
	FROM student AS S,(SELECT @pre_con:=NULL,@trank:=0) AS T
	ORDER BY S.continent,S.NAME
) AS A
```

**按照排名放置数据**

用上面的排名算法，得到的排名数据，格式为：name,continent,trank。

现要明确，

第1行数据，必须来自排名为1的所有行；

......

第i行数据， 必须来自排名为i的所有行；

这需要一个聚合操作,因此对排名数据,用group by分组.

每组内部,要根据continent确定name属于一个属性A.那么,此行属性A的值为name,其它属性值为NULL.

逻辑为:

```mysql
MAX(if(A.continent = 'America',A.NAME,NULL)) AS `America`,
MAX(if(A.continent = 'Asia',A.NAME,NULL)) AS `Asia`,
MAX(if(A.continent = 'Europe',A.NAME,NULL)) AS `Europe`
```

结合两种排名算法,最终结果为:

```mysql
SELECT 
MAX(if(A.continent = 'America',A.NAME,NULL)) AS `America`,
MAX(if(A.continent = 'Asia',A.NAME,NULL)) AS `Asia`,
MAX(if(A.continent = 'Europe',A.NAME,NULL)) AS `Europe`
FROM
(
	SELECT S1.continent,S1.NAME,S1.row_id,COUNT(*) AS `trank`
	FROM 
	(
		SELECT S.*,@row_id:=(@row_id + 1) AS `row_id`
		FROM student AS S,(SELECT @row_id:=0) AS T
	) AS S1 
	JOIN 
	(
		SELECT S.*,@n_row_id:=(@n_row_id + 1) AS `n_row_id`
		FROM student AS S,(SELECT @n_row_id:=0) AS T
	) AS S2 
	ON (S1.continent = S2.continent AND (S1.NAME > S2.NAME OR (S1.NAME = S2.NAME AND S1.row_id >= S2.n_row_id)))
	group BY S1.continent,S1.NAME,S1.row_id
	order BY S1.continent,S1.NAME
) AS A
GROUP BY A.trank
```

或者是:

```mysql
SELECT 
MAX(if(A.continent = 'America',A.NAME,NULL)) AS `America`,
MAX(if(A.continent = 'Asia',A.NAME,NULL)) AS `Asia`,
MAX(if(A.continent = 'Europe',A.NAME,NULL)) AS `Europe`
FROM
(
	SELECT S.*,
	@trank:=if(@pre_con = S.continent,
		@trank + 1,
		1
	) AS `trank`,
	@pre_con:=S.continent AS `pre`
	FROM student AS S,(SELECT @pre_con:=NULL,@trank:=0) AS T
	ORDER BY S.continent,S.NAME
) AS A
GROUP BY A.trank
```
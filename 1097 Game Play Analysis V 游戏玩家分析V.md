### 1097 Game Play Analysis V 游戏玩家分析V

分析玩家的安装日期，以及安装日期后的一日留存

#### 解法一

先找到每个玩家的安装日期。 安装日期 定义为 每个玩家 第一次登录的日期。

对玩家分组，求每组最小的日期，即为登录日期。

结果命名为表A。

```mysql
(
	SELECT player_id,MIN(event_date) AS `install_date`
	FROM Activity
	GROUP BY player_id
) AS A
```

依次统计安装日期的总人数和安装日期第二天的总人数。

连接表A和活动表，再按登录日期分组，统计每组的总人数，以及第二天的总人数。

**注意**：因存在玩家第二天没有登录，需用left join。连接条件同一个玩家第二天又登录。

```mysql
SELECT A.install_date,COUNT(A.player_id) AS `installs`,COUNT(AA.player_id) AS `return_cnt`
FROM 
(
	SELECT player_id,MIN(event_date) AS `install_date`
	FROM Activity
	GROUP BY player_id
) AS A
left JOIN Activity AS AA 
	ON (AA.event_date = DATE_ADD(A.install_date,INTERVAL 1 DAY) AND AA.player_id = A.player_id)
GROUP BY A.install_date
```

1天后的留存率=return_cnt / installs。

#### 解法二

用表连接法求每个玩家的最小日期，即为安装日期。不存在比最小的日期更小的日期了，用left join，最小日期的B.event_date是不存在的。

```mysql
SELECT A.player_id,A.event_date
FROM Activity AS A
LEFT JOIN Activity AS B ON (A.player_id = B.player_id AND A.event_date > B.event_date)
WHERE B.event_date IS NULL
```

再连接Activity表算出最小日期第二天还登录的玩家。

```mysql
SELECT A.player_id,A.event_date,C.player_id
FROM Activity AS A
LEFT JOIN Activity AS B 
ON (A.player_id = B.player_id AND A.event_date > B.event_date)
left JOIN Activity AS C
ON (A.player_id = C.player_id AND C.event_date = DATE_ADD(A.event_date,INTERVAL 1 DAY))
WHERE B.event_date IS NULL
```

基于此，对最小日期分组，统计安装数和1日后的留存率。

```mysql
SELECT 
A.event_date AS `install_dt`,
COUNT(A.player_id) AS `installs`,
round(COUNT(C.player_id)/COUNT(A.player_id),2) AS `Day1_retention`
FROM Activity AS A 
left JOIN Activity AS B
	ON (A.player_id = B.player_id AND A.event_date > B.event_date)
left JOIN Activity AS C
	ON (A.player_id = C.player_id AND C.event_date = DATE_ADD(A.event_date,INTERVAL 1 DAY))
WHERE B.event_date IS NULL
GROUP BY A.event_date
```
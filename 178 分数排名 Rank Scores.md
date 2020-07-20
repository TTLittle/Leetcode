### 178 分数排名 Rank Scores

分数表中Score字段的排名。分数从高到低，名次为从1到N。相同的分值，其名次是相同的。

#### 解法一

按Id和Score分组。每组中，大于等于每个Score的不同Score数目就是其排名。

```mysql
select S1.Score,count(distinct S2.Score) as `Rank`
from Scores as S1 join Scores as S2 on (S1.Score <= S2.Score)
group by S1.Id,S1.Score
order by S1.Score desc;
```

**注意**：FROM子句中关系的**属性**都可以在HAVING和SELECT子句中用**聚集运算**，但是只有出现在GROUP BY子句中的**属性**，才能以**不聚集**的方式出现在HAVING和SELECT子句中。

因此，上面的S1.ID可以不出现在SELECT子句中，S1.SCORE可以出现在SELECT子句中。其中它字段不能以非聚集的方式出现在SELECT子句中。比如，S2.Score。

#### 解法二

延续解法一的思路。 将“大于等于每个Score的不同Score数目”在子查询中实现，并且不用再分组。

```mysql
select S1.Score,
    (select count(distinct S2.Score) 
     from Scores as S2 
     where S1.Score <= S2.Score
    ) as `Rank`
from Scores as S1
order by S1.Score desc;
```

#### 解法三

继续优化解法一。先将表中不同的score全部取出来，以子查询的方式完成。再与表连接，求 “大于等于每个Score的不同Score数目” 。依然要分组。

```mysql
SELECT s1.Score, COUNT(s2.score) AS `Rank`
FROM Scores s1
JOIN
    (
        /*get score rank descending as temporary table s2*/
        SELECT DISTINCT Score
        FROM Scores
        ORDER BY Score DESC
    ) AS s2 
ON (s1.Score <= s2.Score)
GROUP BY s1.Id,s1.Score
ORDER BY s1.Score desc
```
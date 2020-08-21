### 614 二级关注者 Second Degree Follower

follow表：

```
+-------------+------------+
| followee    | follower   |
+-------------+------------+
|     A       |     B      |
|     B       |     C      |
|     B       |     D      |
|     D       |     E      |
+-------------+------------+
```

**followee被**关注者 , **follower**关注者。

关注者本身也可能被其它人关注，成为被关注者。

对每一个关注者，查询他的关注者数目。

先找出follower被哪些人关注。

```mysql
select *
from follow as F1 
join follow as F2 
on (F1.follower = F2.followee)
```

再按follower分组，算出每个follower被关注的人数。

```mysql
select F1.follower,count(distinct F2.follower) as `num`
from follow as F1 
join follow as F2 
on (F1.follower = F2.followee)
group by F1.follower
```
### 574 当选者 Winning Candidate

表: Candidate

```
 +-----+---------+
 | id  | Name    |
 +-----+---------+
 | 1   | A       |
 | 2   | B       |
 | 3   | C       |
 | 4   | D       |
 | 5   | E       |
 +-----+---------+
```

表: `Vote`

```
+-----+--------------+
| id  | CandidateId  |
+-----+--------------+
| 1   |     2        |
| 2   |     4        |
| 3   |     3        |
| 4   |     2        |
| 5   |     5        |
+-----+--------------+
id 是自动递增的主键，
CandidateId 是 Candidate 表中的 id.
```

#### 解法一（此方法更佳）

按CandidateId分组，统计投票个数，取投票数最高的CandidateId。

再连接表Candidate，取姓名。

```mysql
select C.Name
from Candidate as C join
(
    select V.CandidateId,count(V.id) as cnt
    from Vote as V
    group by V.CandidateId
    order by cnt desc
    limit 0,1
) as A
on (C.id = A.CandidateId)
```

#### 解法二

直接连接两表，再按Candidate分组，统计投票数，取最高人。

有个**重要的细节**是Vote表中的某些CandidateId可能不在Candidate 表中。

先讲一个错误答案。

```
select C.Name
from Candidate as C join
Vote as V on(C.id = V.CandidateId)
group by V.CandidateId,C.Name
order by count(V.id) desc
limit 0,1
```

两表内连接，**会过滤掉**哪些没有名字的CandidateId。恰好这些CandidateId的投票数又最多。

再讲一个错误答案。

```mysql
select C.Name
from Vote as V 
left join 
Candidate as C
on(C.id = V.CandidateId)
group by V.CandidateId,C.Name
order by count(V.id) desc
limit 0,1
```

left join，**保留了**没有名字的CandidateId。 恰好这些CandidateId的投票数又最多。 那么，可能有值是**NULL**的行。则又不正确。

但是在其中添加where和having子句过滤掉Name是NULL的行，又不正确。这 **会过滤掉**那些没有名字的CandidateId。恰好这些CandidateId的投票数又最多。

所以要在外围过滤掉Name是NULL的行。

```mysql
select A.Name
from(
    select C.Name
    from Vote as V left join 
    Candidate as C
     on(C.id = V.CandidateId)
    group by V.CandidateId,C.Name
    order by count(V.id) desc
    limit 0,1
) as A
where A.Name is not null
```
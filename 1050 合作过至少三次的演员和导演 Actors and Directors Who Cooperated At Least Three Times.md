### 1050 合作过至少三次的演员和导演 Actors and Directors Who Cooperated At Least Three Times

找出一起至少合作过3次的演员和导演。

对二元组(演员,导演)分组，计算每组个数，选出每组个数>=3的行。

应用group by和count。

```mysql
select A.actor_id,A.director_id
from ActorDirector as A
group by A.actor_id,A.director_id
having count(A.timestamp) >= 3
```
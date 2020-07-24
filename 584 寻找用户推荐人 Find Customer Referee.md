### 584 寻找用户推荐人 Find Customer Referee

寻找推荐人不是2的人。

如果过滤掉推荐人是2的人。但也有可能没有推荐人。

```mysql
select name
from customer
where referee_id != 2 or referee_id is null
```
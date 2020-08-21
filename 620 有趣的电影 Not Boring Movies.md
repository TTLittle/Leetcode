### 620 有趣的电影 Not Boring Movies

```
下表 cinema:

+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+

对于上面的例子，则正确的输出是为：

+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```

找出所有影片描述为**非**`boring` (不无聊) 的并且 **id 为奇数** 的影片，结果请按等级 `rating` 排列 。

把要求转换成条件即可。

```mysql
select *
from cinema as C
where C.id % 2 =1 and C.description !='boring' 
order by rating desc;
```

取模可以用：&和mod代替。

where可以是：

```mysql
where C.id&1 =1 and C.description !='boring' 
```

或是：

```mysql
where mod(C.id,2) =1 and C.description !='boring' 
```
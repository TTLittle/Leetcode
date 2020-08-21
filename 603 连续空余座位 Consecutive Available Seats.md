### 603 连续空余座位 Consecutive Available Seats

大于等于 2 个连续空余的座位 成为连续空余座位。

座位的seat_id连续自增。

表自连接，找出所有相邻的空位。相邻的座位指seat_id相差为1的座位。

并按seat_id排序 。

```mysql
select distinct C1.seat_id
from cinema as C1 
join cinema as C2
on(C1.free='1' and C2.free='1' and (C1.seat_id+1=C2.seat_id or C1.seat_id-1=C2.seat_id))
order by C1.seat_id
```


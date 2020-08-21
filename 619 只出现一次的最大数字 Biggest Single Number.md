### 619 只出现一次的最大数字 Biggest Single Number

**找出仅出过一次的最大数。**

对数字分组，每组计数，选出计数等于1的行。

从多个数字再取最大值。

```mysql
select max(A.num) as `num`
from 
(
    select M.num as `num`
    from my_numbers as M
    group by M.num
    having count(M.num)=1
) as A
```

有一种易错的方法。

```mysql
select A.num
from 
(
    select M.num as `num`
    from my_numbers as M
    group by M.num
    having count(M.num)=1
    order by M.num desc
    limit 0,1
) as A
```

思路上是对的，但当 没有只出现一次的数字 时，表A为空但不是NULL。不符合题目的要求。

修改为下面即可。但这样的修改还不及上面的直接。

```mysql
select max(A.num) as `num`
from 
(
    select M.num as `num`
    from my_numbers as M
    group by M.num
    having count(M.num)=1
    order by M.num desc
    limit 0,1
) as A
```

也可修改为如下。

应用[IFNULL](http://www.mysqltutorial.org/mysql-ifnull/)。当第一个参数不为NULL时，返回第一个参数，否则返回第二个参数。

```mysql
SELECT IFNULL(
	(
		select M.num
		from my_numbers as M
		group by M.num
		having count(M.num)=1
		order by M.num desc
		limit 0,1
	)
, NULL
) AS `num`
```

由于本例中表只有一列。

上述代码还可简化为：

```mysql
SELECT IFNULL(
	(
		SELECT *
		from my_numbers as M
		group by M.num
		having COUNT(*)=1
		order by M.num desc
		limit 0,1
	)
, NULL
) AS `num`
```
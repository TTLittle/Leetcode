### leetcode1098 Unpopular Books 不受欢迎的书

找出过去一年里，销售量少于10的书。最近1个月才开卖的书除外。

今天定义为2019-06-23.

#### 解法一

先过滤掉最近1个月才开卖的书。

结果集命名为表A。

```mysql
(
    select *
    from books as B
    where B.available_from <= DATE('2019-05-23')
) AS A
```

连接表A和订单表。用left join。排除掉2018-06-23到2019-06-23之外的订单。

```mysql
SELECT *
from
(
    select *
    from books as B
    where B.available_from <= '2019-05-23'
) AS A
left join orders as O 
    on (A.book_id = O.book_id and O.dispatch_date BETWEEN '2018-06-23' AND '2019-06-23');
```

再按A.book_id分组，输出销售量少于10个的书。

```mysql
select A.book_id,max(A.NAME) AS `name`
from
(
    select *
    from books as B
    where B.available_from <= DATE('2019-05-23')
) AS A
left join orders as O 
    on (A.book_id = O.book_id and O.dispatch_date BETWEEN DATE('2018-06-23') AND DATE('2019-06-23'))
group by A.book_id
HAVING SUM(if(O.quantity IS NULL,0,O.quantity)) < 10
```

#### 解法二

直接连接书表和订单表。 用left join。排除掉2018-06-23到2019-06-23之外的订单。

再排除掉 最近1个月才开卖的书。

```mysql
SELECT *
FROM
books AS B
LEFT JOIN orders AS O ON (B.book_id = O.book_id AND O.dispatch_date BETWEEN DATE('2018-06-23') AND DATE('2019-06-23'))
WHERE B.available_from <= DATE('2019-05-23')
```

再按A.book_id分组，输出销售量少于10个的书。

```mysql
SELECT B.book_id, MAX(B.NAME) AS `name`
FROM
books AS B
LEFT JOIN orders AS O ON (B.book_id = O.book_id AND O.dispatch_date BETWEEN DATE('2018-06-23') AND DATE('2019-06-23'))
WHERE B.available_from <= DATE('2019-05-23')
GROUP BY B.book_id
HAVING SUM(IF(O.quantity IS NULL,0,O.quantity)) < 10
```
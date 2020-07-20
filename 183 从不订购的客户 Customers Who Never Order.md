### 183 从不订购的客户 Customers Who Never Order

找出不买商品的客户，输出姓名。

#### 解法一

顾客表的id和订单表的customerid关联，得出的是买了的东西的顾客。用left join，没买东西的顾客，其对应的订单为空。这是一种求集合差的方法。

```mysql
select C.name as `Customers`
from Customers as C left join Orders as O on (C.id = O.customerid)
where O.id is NULL;
```

先用子查询将买过东西的顾客id选出来。 在应用left join求集合差。

```mysql
select C.name as `Customers`
from Customers as C left join (
    select distinct customerid
    from Orders
) as O on (C.id = O.customerid)
where O.customerid is NULL;
```

#### 解法二

既然求集合差，用not in也可以。 先用子查询将买过东西的顾客id选出来。 然后排除这些顾客的id即可。

```mysql
select C.name as `Customers`
from Customers as C 
where C.id not in (
    select distinct customerid
    from Orders
) 
```

集合差定义：C=A-B。C中的元素等于在A中但是不在B中。因此，对A中的每个元素a，如果元素a不在B中，则元素a就是集合C的元素。

**[EXISTS](http://www.mysqltutorial.org/mysql-exists/)**是布尔运算符，常用于测试子查询。

```mysql
SELECT 
    select_list
FROM
    a_table
WHERE
    [NOT] EXISTS(subquery);
```

当subquery返回任何行时，EXISTS返回true，否则返回false。

因此得出下述算法。

```mysql
select C.name as `Customers`
from Customers as C 
where not exists (
    select distinct customerid
    from Orders as O
    where O.customerid = C.id
) ;
```
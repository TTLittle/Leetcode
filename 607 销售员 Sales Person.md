### 607 销售员 Sales Person

找出没有向公司RED卖过东西的销售员。

#### 解法一

集合差算法。

从全部销售员中排除掉向RED公司卖过的销售员。

先找出向RED公司卖过东西的销售员。

```mysql
(
    select distinct O.sales_id
    from Company as C join Orders as O
        on(C.name = 'RED'  and C.com_id = O.com_id)
) as A
```

从全部销售员中排除掉这些销售员。

用left join

```mysql
select SP.name
from salesperson as SP left join 
(
    select distinct O.sales_id
    from Company as C join Orders as O
        on(C.name = 'RED'  and C.com_id = O.com_id)
) as A
    on(SP.sales_id = A.sales_id)
where A.sales_id is NULL
```

#### 解法二

每个人销售产品给公司。找出每个人业务往来的公司。

```mysql
select *
FROM 
salesperson AS SP LEFT JOIN Orders AS O
	ON(SP.sales_id = O.sales_id)
left join Company AS C
	ON(C.com_id = O.com_id)
```

**注意**：存在某些人没有与任何公司有业务往来。也存在某些公司没有订单。所以用left join保留没有业务的人。

对人分组，统计每个人与red公司相关的订单数。并找出订单数为0的人。

```mysql
SELECT max(SP.NAME) AS `name`
FROM 
salesperson AS SP LEFT JOIN Orders AS O
	ON(SP.sales_id = O.sales_id)
left join Company AS C
	ON(C.com_id = O.com_id)
GROUP BY SP.sales_id
HAVING SUM(If(C.NAME='RED',1,0)) = 0
```
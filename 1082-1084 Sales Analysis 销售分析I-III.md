## 1082-1084 Sales Analysis I 销售分析I-III

### 1082 Sales Analysis I 销售分析I

#### I 找出总售价最高的卖家。

##### 解法1

先找出每个卖家的总售价：对卖家分组，求和。

```mysql
select seller_id,sum(price) as `price`
from sales
group by seller_id
```

取最高的总售价，可以用max

```mysql
select max(A.price) as `max_price`
from
(
	select seller_id,sum(price) as `price`
	from sales
	group by seller_id
) as A
```

也可以用limit取最大值

```mysql
select seller_id,sum(price) as `price`
from sales
group by seller_id
order by price desc
limit 0,1
```

再找出总售价与最高售价相等的卖家。连接表A与卖家总售价表。

```mysql
select C.seller_id
from
(
    select max(A.price) as `max_price`
    from
    (
        select seller_id,sum(price) as `price`
        from sales
        group by seller_id
    ) as A
) as B
join (
    select seller_id,sum(price) as `price`
    from sales
    group by seller_id
) as C
    on(B.max_price = C.price)
```

或

```mysql
select C.seller_id
from
(
	select seller_id,sum(price) as `price`
	from sales
	group by seller_id
    order by price desc
    limit 0,1
) as B
join (
    select seller_id,sum(price) as `price`
    from sales
    group by seller_id
) as C
    on(B.price = C.price)
```

##### 解法二

求与最大值相等的问题。

第一种采用解法一的思路，先求出最大值，再表与最大值连接，选出等于最大值的行。

第二种对每行数据，判断其与最大值是否相等。每次判断时都会计算一次最大值。

求最高售价同解法一。

```mysql
	SELECT SUM(price) AS `price`
	FROM sales
	GROUP BY seller_id
	ORDER BY price DESC
	LIMIT 0,1
```

每个卖家都检查一次与最高售价是否相等。

```mysql
SELECT S.seller_id
FROM sales AS S
GROUP BY S.seller_id
HAVING SUM(S.price) = (
	SELECT SUM(price) AS `price`
	FROM sales
	GROUP BY seller_id
	ORDER BY price DESC
	LIMIT 0,1
)
```

### 1083 Sales Analysis II 销售分析II

#### 找出买了S8但没买iphone的人。

##### 解法一

先找出买了S8的人

```mysql
(
    select distinct buyer_id
    from Product as P join Sales as S
        on(P.product_id = S.product_id and P.product_name ='S8')
) as A
```

再找出买了iPhone的人

```mysql
(
    select distinct buyer_id
    from Product as P join Sales as S
        on(P.product_id = S.product_id and P.product_name ='iPhone')
) as B
```

再从买了S8的人中排除掉买了iPhone的。

用left join求差集

```mysql
select A.buyer_id
from
(
    select distinct buyer_id
    from Product as P join Sales as S
        on(P.product_id = S.product_id and P.product_name ='S8')
) as A
left join
(
    select distinct buyer_id
    from Product as P join Sales as S
        on(P.product_id = S.product_id and P.product_name ='iPhone')
) as B
    on(A.buyer_id = B.buyer_id)
where B.buyer_id is NULL
```

##### 解法二

对每个买家，统计其买S8的总数量A和iphone的总数量B，选出A大于0但是B等于0的买家。

连接销售表和产品表。

注意：

1、并不是每个产品都会被用户买。因此要用left join。

2、要按买家分组，计算A和B，选出A>0且B=0的买家。

```mysql
select S.buyer_id
from Sales as S left join Product as P 
    on(P.product_id = S.product_id)
group by S.buyer_id
having  sum(if(P.product_name ='S8',1,0)) > 0 and sum(if(P.product_name ='iPhone',1,0)) =0
```

### 1084 Sales Analysis III 销售分析III

#### III 仅有第一季度销售出去的产品

##### 解法一

先找出第一季度卖的产品：

```mysql
(
    select distinct S.product_id
    from Sales as S
    where (S.sale_date between '2019-01-01' and '2019-03-31')
) as A
```

再找出其它季度卖的产品：

```mysql
(
    select distinct S.product_id
    from Sales as S
    where (S.sale_date < '2019-01-01' or S.sale_date > '2019-03-31')
)as B
```

从第一季度卖的产品中排除掉其它季度卖过的产品。

用表A left join 表B，计算集合差。

```mysql
select P.product_id,P.product_name
from
Product as P
join
(
    select distinct S.product_id
    from Sales as S
    where (S.sale_date between '2019-01-01' and '2019-03-31')
) as A
    on(P.product_id =A.product_id)
left join 
(
    select distinct S.product_id
    from Sales as S
    where (S.sale_date < '2019-01-01' or S.sale_date > '2019-03-31')
)as B
    on (A.product_id = B.product_id)
where B.product_id is NULL
```

##### 解法二

先选出所有售卖过的产品。

```mysql
select *
from
Product as P join Sales as A
    on(P.product_id =A.product_id)
```

选出在其它季度售卖过的产品。

```mysql
(
    select distinct S.product_id
    from Sales as S
    where (S.sale_date < '2019-01-01' or S.sale_date > '2019-03-31')
)as B
```

再从所有售卖过的产品中排除掉其它季度售卖过的产品。

用left join做集合差。

```mysql
select distinct P.product_id,P.product_name
from
Product as P join Sales as A
    on(P.product_id =A.product_id)
left join 
(
    select distinct S.product_id
    from Sales as S
    where (S.sale_date < '2019-01-01' or S.sale_date > '2019-03-31')
)as B
    on (A.product_id = B.product_id)
where B.product_id is NULL
```
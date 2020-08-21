## 1068-1070 Product Sales Analysis I 产品销售分析I-III

### 1068 Product Sales Analysis I 产品销售分析I

取产品的名称，必须连接表Project。

按需要的字段，原样输出。

```mysql
select P.product_name,S.year,S.price
from
Sales as S join Product as P
    on(S.product_id = P.product_id)
```

### 1069 销售分析ⅡProduct Sales Analysis II

#### II 求每个产品的销售总量。

对每个产品分组，每组产品的销量求和。

应用group by 和 sum。

```mysql
select S.product_id,sum(S.quantity) as `total_quantity`
from Sales as S
group by S.product_id
```

### 1070 Product Sales Analysis III 产品销售分析III

Sales表:

```
+---------+------------+------+----------+-------+
| sale_id | product_id | year | quantity | price |
+---------+------------+------+----------+-------+ 
| 1       | 100        | 2008 | 10       | 5000  |
| 2       | 100        | 2009 | 12       | 5000  |
| 7       | 200        | 2011 | 15       | 9000  |
+---------+------------+------+----------+-------+
```

#### III 找出第一年销售此商品时的信息.

找到每个商品第一次销售的年份.

应用**[MIN函数](http://www.mysqltutorial.org/mysql-aggregate-functions.aspx)**和group by.

```mysql
(
    select S.product_id,min(S.year) as `first_year`
    from Sales as S
    group by S.product_id
) as A
```

再连接Sales表取出first_year时的信息.

```mysql
select A.product_id,A.first_year,B.quantity,B.price
from 
(
    select S.product_id,min(S.year) as `first_year`
    from Sales as S
    group by S.product_id
) as A
join Sales as B
    on(A.product_id = B.product_id and A.first_year = B.year)
```

## 解法二

思路同解法一.

但应用**IN**.

```mysql
select product_id,S.year as `first_year`,quantity,price
from Sales as S
where (S.product_id,S.year) in (
    select product_id,min(year)
    from Sales
    group by product_id
)
```

或（下面这个方法会超时）

```mysql
select product_id,S.year as `first_year`,quantity,price
from Sales as S
where S.year in (
    select min(year)
    from Sales
    where product_id = S.product_id
)
```
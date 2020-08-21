### 1045 买下所有产品的客户 Customers Who Bought All Products

#### 有N种商品，找出买过所有N种商品的人。

##### 解法一

算出每个人买过的产品种数A， 哪些A等于产品总种数的人就是结果.

**注意**:一个人可能多次购买同一种商品

对每个人分组,算出产品种数.

```mysql
(
	SELECT C.customer_id,COUNT(DISTINCT C.product_key) AS `cnt`
	FROM Customer AS C
	GROUP BY C.customer_id
) AS A
```

产品总种数.

```mysql
(
	SELECT COUNT(*) AS `cnt`
	FROM Product
) AS B
```

连接表A和表B,A.cnt = B.cnt选出买过全部商品的人.

```mysql
SELECT A.customer_id
FROM 
(
	SELECT C.customer_id,COUNT(DISTINCT C.product_key) AS `cnt`
	FROM Customer AS C
	GROUP BY C.customer_id
) AS A
JOIN 
(
	SELECT COUNT(*) AS `cnt`
	FROM Product
) AS B
	ON(A.cnt = B.cnt)
```

##### 解法二

思路同解法一.不再用表连接，对每个人分组,直接选出购买过全部产品的人 .

```mysql
SELECT C.customer_id
FROM Customer AS C
GROUP BY C.customer_id
HAVING COUNT(DISTINCT C.product_key) = (SELECT COUNT(*) FROM Product)
```


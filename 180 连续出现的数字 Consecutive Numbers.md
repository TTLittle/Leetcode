### 180 连续出现的数字 Consecutive Numbers

从表中，查找所有至少连续出现三次的数字。

#### 解法一

题目暗示，每行的id是连续的。因此，表三次自连接，将连续三行且数字都相等行选出来。

```mysql
SELECT distinct L1.num AS ConsecutiveNums
FROM `Logs` AS L1
JOIN `Logs` AS L2 ON (L1.num = L2.num AND L1.id + 1 = L2.id)
JOIN `Logs` AS L3 ON (L2.num = L3.num AND L2.id + 1 = L3.id)
```

#### 解法二

使用用户变量, 仅从行数据考虑，需要**用户变量**记录前一行数据。当前行数据与前一行数据比较是否相同。

定义两个**用户变量**：

@pre ： 前一行数据

@count：与前一行数据连续相同的个数

初始化@pre和@count，定义为一张表：(SELECT @pre:= NULL,@count:=0) AS temp

Logs与temp叉积，计算出与每行数字连续相同的数字个数。

与前一行数据比较的逻辑为：

@count:= IF(@pre=t.Num, @count+1, 1)

@pre与t.Num ： 前一行数据与当前行相同。

如果相同，@count+1，否则@count=1。

这些结果形成一张新表：a(Num,cnt,USELESS)。

最终只要从a中取出cnt大于等于3的行。

```mysql
SELECT DISTINCT a.Num AS ConsecutiveNums
FROM (
		SELECT 
		t.Num,
		(@count:= IF(@pre=t.Num, @count+1, 1)) AS cnt,
		 @pre:=t.Num AS USELESS
		FROM LOGS as t,(SELECT @pre:= NULL,@count:=0) AS b
) AS a
WHERE a.cnt >= 3;
```
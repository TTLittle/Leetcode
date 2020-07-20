### 182 查找重复的电子邮箱 Duplicate Emails

#### 解法一

如果一个字段的值在表中重复了，那么含有重复值的行数一定超过1。

group by 对Email分组，那么Email重复的行个数大于1。

having 筛选出这些行。

```mysql
select Email
from Person as P
group by Email
having count(Id) > 1
```

#### 解法二

如果假设表中的字段Id是唯一的，还可以自连接表，选中哪些Id不同，Email相同的行。注意投影后要distinct去重。

```mysql
select distinct P1.Email
from Person as P1 join Person as P2 on (P1.Id != P2.Id and P1.Email = P2.Email)
```

#### 解法三

对每行的Email，子查询中都在表中查找一次，Email相同的行个数超过1。选中此行。此方法显然效率低。

```mysql
select distinct P1.Email
from Person as P1 
where 1 < (
    select count(*)
    from Person as P2
    where P1.email = P2.email
);
```


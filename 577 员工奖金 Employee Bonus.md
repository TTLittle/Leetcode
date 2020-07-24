### 577 员工奖金 Employee Bonus

找出奖金少于1000的员工

存在员工没有奖金的情况。应用left join。

```mysql
select E.name,B.bonus
from Employee as E 
left join Bonus as B
on(E.empId = B.empId)
where B.bonus is NULL or B.bonus < 1000
```

有一个迷惑性很强但是错误的答案。

```mysql
select E.name,B.bonus
from Employee as E 
left join Bonus as B
on(E.empId = B.empId and (B.bonus is NULL or B.bonus < 1000))
```

相比于之前的解法，只是将where条件移入on条件中。

当有员工的奖金超过1000时，on条件不满足。又由于是left join，此员工还是被保留下来了。这样，on条件没有起到应有的作用。
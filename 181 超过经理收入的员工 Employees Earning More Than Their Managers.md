### 181 超过经理收入的员工 Employees Earning More Than Their Managers

通过表的自连接，找出每个员工的经理，筛选出薪水比经理薪水高的员工。

```mysql
select E1.Name as `Employee`
from Employee as E1 join Employee as E2 on (E1.ManagerId  = E2.Id and E1.salary > E2.salary)
```


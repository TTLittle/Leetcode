### 184 部门工资最高的员工 Department Highest Salary

员工表的departmentid 与部门表的id关联。

#### 解法一

先找出每个部门的最高薪水。

连接员工表和部门表，group by对部门分组，再求每组的最高薪水。用子查询得出临时表F(id,name,m)。

```mysql
(
select D.id,D.name,max(E1.salary) as m
from Department as D
join Employee as E1 
on (D.id = E1.departmentid)
group by D.id,D.name
) as F
```

再次，连接员工表和临时表F，条件是部门id相同且薪水与每个部门最高薪水相同。与上面的组合起来得出算法：

```mysql
select F.name as `Department`,E.name as `Employee`,E.Salary
from Employee as E
join (
    select D.id,D.name,max(E1.salary) as m
    from Department as D
    join Employee as E1 
        on (D.id = E1.departmentid)
    group by D.id,D.name
) as F
    on (E.departmentid = F.id and F.m = E.salary);
```

另外员工表中有部门id和薪水，但没有部门name。可先求部门的最高薪水，形成临时表，再连接员工表和部门表，取出部门name和员工name。则有下面的算法：

```mysql
select D.name as `Department`,E.name as `Employee`,E.Salary
from Employee as E
join (
    select E1.departmentid,max(E1.salary) as m
    from Employee as E1
    group by E1.departmentid
) as F
on (E.departmentid = F.departmentid and F.m = E.salary)
join Department as D
on (F.departmentid = D.id)
```

#### 解法二

对每一个员工，找出员工所在部门的最高薪水。此处的查找过程，用子查询实现。如果员工的薪水等于部门的最高薪水就是结果。

```mysql
select D.name as `Department`,E.name as `Employee`,E.Salary
from Employee as E
join Department as D
    on (E.departmentid = D.id)
where E.salary = (
    select max(E1.salary) 
    from Employee as E1
    where E1.departmentid = E.departmentid
)
```

当然二元组(部门id,部门最高薪水)作为整体t，结合in判断t是否在已存在的组合中。

```mysql
select D.name as `Department`,E.name as `Employee`,E.Salary
from Employee as E
join Department as D
    on (E.departmentid = D.id)
where (E.departmentid,E.salary) in (
    select E1.departmentid,max(E1.salary) 
    from Employee as E1
    where E1.departmentid
    group by E1.departmentid
)
```
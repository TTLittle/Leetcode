### 177 第N高的薪水 Nth Highest Salary

#### 解法一

对每一个薪水A，只要大于等于A的不同薪水个数等于N即可。

因此子查询求出大于等于A的不同薪水个数B。当B=N时，能得出结果。

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
    select distinct t1.Salary
    from Employee t1
    where N = (
        select count(distinct t2.Salary)
        from Employee t2
        where t2.Salary >= t1.Salary
    )
  );
END
```

**[CREATE FUNCTION](https://dev.mysql.com/doc/refman/8.0/en/create-function.html)** 的详情参考手册。

## 解法二

直接用order by和limit。编号是从0开始，所以limit的偏移也是从0开始。第N个变成参数时，要改为N-1。

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    SET N = N - 1;
  RETURN (  
      select distinct salary
      from Employee
      order by salary desc
      limit N,1
  );
END
```
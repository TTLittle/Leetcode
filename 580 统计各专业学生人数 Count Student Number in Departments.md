### 580 统计各专业学生人数 Count Student Number in Departments

从***student\*** 表 和***department\*** 表中，统计每个专业的人数。

求“每个......”类型的题目，一般是要用到group by，而且往往是多个表之间先join，再分组。

连接两表。由于存在某些专业没有学生。因此left join更合适。

按dept_id分组，计算每组学生数。

查询结果按学生人数降序，再按专业名称升序。

```mysql
select D.dept_name,count(S.student_id) as `student_number`
from department as D 
left join student as S 
on (D.dept_id = S.dept_id)
group by D.dept_id,D.dept_name
order by `student_number` desc,D.dept_name
```
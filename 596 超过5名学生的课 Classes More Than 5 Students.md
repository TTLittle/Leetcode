### 596 超过5名学生的课 Classes More Than 5 Students

对课程用group by分组，过滤学生少于5个的课程。

```mysql
select C.class
from courses as C
group by C.class
having count(distinct C.student) >=5
```


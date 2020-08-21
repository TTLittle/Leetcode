### 610 判断三角形 Triangle Judgement

应用三角形成立的条件，两边之和大于第三边。

用[IF](http://www.mysqltutorial.org/mysql-if-function.aspx)语句。

```mysql
select 
T.*,
if((T.x+T.y>T.z) and (T.x+T.z>T.y) and (T.y+T.z>T.x),'Yes','No') as `triangle`
from triangle as T
```

或用[CASE](http://www.mysqltutorial.org/mysql-case-function/)语句。

```mysql
select 
T.*,
case when (T.x+T.y>T.z) and (T.x+T.z>T.y) and (T.y+T.z>T.x) then 'Yes' 
    else 'No' end  as `triangle`
from triangle as T
```
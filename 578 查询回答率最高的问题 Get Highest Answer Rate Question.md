### 578 查询回答率最高的问题 Get Highest Answer Rate Question

按question_id分组，每组计算回答率，并按回答率降序。

回答率 = 'answer'的个数 / 'show'的个数

SQL代码

```mysql
sum(if(S.action='answer',1,0))/sum(if(S.action='show',1,0))
```

取最高回答率的行。

```mysql
select S.question_id as `survey_log`
from survey_log as S
group by S.question_id 
order by sum(if(S.action='answer',1,0))/sum(if(S.action='show',1,0)) desc
limit 0,1
```
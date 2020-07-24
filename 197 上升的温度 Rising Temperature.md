### 197 上升的温度 Rising Temperature

找出温度比前一天高的行。

使用**自连接**即可

**确定日期的前一天：**

日期函数： [DATEDIFF](http://www.mysqltutorial.org/mysql-datediff.aspx)(date1,date2) ，返回date1与date2之间相差的天数。

```mysql
SELECT W1.Id
FROM weather AS W1 JOIN weather AS W2 ON (DATEDIFF(W1.RecordDate,W2.RecordDate) = 1 AND W1.Temperature > W2.Temperature)
```

同样，函数： [TIMESTAMPDIFF](http://www.mysqltutorial.org/mysql-timestampdiff/)(unit,begin,end)，返回begin与end之间相差多少个unit。unit为DAY时，即为相差“天”。

```mysql
SELECT W2.Id
FROM weather AS W1 JOIN weather AS W2 
ON (TIMESTAMPDIFF(DAY,W1.RecordDate,W2.RecordDate) = 1 AND W1.Temperature < W2.Temperature)
```
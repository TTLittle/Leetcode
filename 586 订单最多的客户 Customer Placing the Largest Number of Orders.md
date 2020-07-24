### 586 订单最多的客户 Customer Placing the Largest Number of Orders

找出订单最多的客户。

对客户分组，计算每组订单数，再订单数降序，取第一个客户。

```mysql
select O.customer_number
from orders as O
group by O.customer_number
order by count(O.order_number) desc
limit 0,1
```


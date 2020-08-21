### 627 交换工资 Swap Salary

```
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
运行你所编写的更新语句之后，将会得到以下表:

| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```

update语句中，应用条件语句。

```mysql
update salary
set
    sex = if(sex='m','f','m')
```

或者

```mysql
update salary
set
    sex = case when sex='m' then 'f'
          else 'm' end
```
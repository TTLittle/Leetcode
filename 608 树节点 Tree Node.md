### 608 树节点 Tree Node

树表

```
+----+------+
| id | p_id |
+----+------+
| 1  | null |
| 2  | 1    |
| 3  | 1    |
| 4  | 2    |
| 5  | 2    |
+----+------+
```

得出的节点类型

```
+----+------+
| id | Type |
+----+------+
| 1  | Root |
| 2  | Inner|
| 3  | Leaf |
| 4  | Leaf |
| 5  | Leaf |
+----+------+
```

节点类型判断依据：

没有父节点的是**根**。

没有子节点的是**叶子**。

其它的是**内部节点**。

#### 解法一

父节点为NULL是根节点。

```mysql
T.p_id is NULL
```

在父节点中出现过的是内部节点。

```mysql
exists (
            select *
            from tree as T1
            where T1.p_id = T.id
)
```

其它的是叶子节点。

合并上述逻辑得：

```mysql
select T.id,
if(T.p_id is NULL,
   'Root'
   ,
   if(
        exists (
            select *
            from tree as T1
            where T1.p_id = T.id
        )
       ,
       'Inner'
       ,
       'Leaf'
   ) 
  )as 'Type'
from tree as T
```

#### 解法二

从id集合中，逐步排除掉根节点，叶节点，剩下的都是内节点。

考虑表left join自连接。

```mysql
SELECT *
FROM tree AS T1 LEFT JOIN tree AS T2 ON (T1.id = T2.p_id)
```

显然T1.p_id为NULL的是根节点。

叶子节点的id不可能出现在p_id字段。因此T.p_id为NULL的是叶子节点。

剩下的是内节点。

合起来，判断节点类型的逻辑为：

```mysql
if(T1.p_id IS NULL,
	'Root',
	if(T2.p_id IS NULL,
	'Leaf',
	'Inner'
	)
	) AS `Type`
```

完整的逻辑为：

```mysql
SELECT distinct T1.id,
if(T1.p_id IS NULL,
	'Root',
	if(T2.p_id IS NULL,
	'Leaf',
	'Inner'
	)
	) AS `Type`
FROM tree AS T1 LEFT JOIN tree AS T2 ON (T1.id = T2.p_id)
```
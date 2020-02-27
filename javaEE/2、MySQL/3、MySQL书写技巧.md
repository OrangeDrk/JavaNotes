[TOC]



# SQL查询分类

1. 过滤查询：where
2. 分组查询：group by ... having ...
3. 排序查询：order by
4. 分页查询：limit
5. 多表查询
6. 子查询



# SQL书写技巧

## SQL查询书写规则

select ... from ... where ... group by  having ... order by ... limit

## SQL执行顺序

from ... where ... group by  having ... select ... order by ... limit



## 使用技巧：

> 1. 将该查询需要的查询关键字【按照SQL书写顺序写好】
> 2. 按照执行顺序填写每个关键字之间的【空】

```sql
select uname,ucity	#..4
from user			#..1
where age>20		#..2
group by ucity		#..3
#having #没有用到	
order by age desc	#..5
```



## 子查询

### 非关联子查询

1. 将子查询当作一张表使用
2. 将子查询当作一个集合来使用（子查询结果必须只有一列）

### 关联子查询
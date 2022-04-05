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



# 各种查询详解

```sql
create database mydb2;
use mydb2;

create table user (id int, name varchar(20),age int, addr varchar(20));
insert into user values (1,'aa',19,'bj');
insert into user values (2,'bb',23,'sh');
insert into user values (3,'cc',26,'gz');
insert into user values (4,'dd',24,'bj');
insert into user values (5,'ee',31,'sz');
insert into user values (6,'ff',27,'sh');
insert into user values (7,'gg',26,'bj');

create table dept (id int ,name varchar(20));
insert into dept values (999,'行政部');
insert into dept values (888,'财务部');
insert into dept values (777,'销售部');
insert into dept values (666,'科技部');

create table emp (id int,name varchar(20),did int);
insert into emp values (1,'萨达姆',999);
insert into emp values (2,'哈利波特',888);
insert into emp values (3,'孙悟空',777);
insert into emp values (4,'朴乾',777);
insert into emp values (5,'本拉登',555);

create table emp2( id int, name varchar(20), salary int, did int, job varchar(20) );
insert into emp2 values (1,'aaa',3200,999,'it');
insert into emp2 values (2,'bbb',4500,888,'it');
insert into emp2 values (3,'ccc',7300,999,'saler');
insert into emp2 values (4,'ddd',3000,999,'saler');
insert into emp2 values (5,'eee',2800,777,'hr');
insert into emp2 values (6,'fff',5100,777,'it');
insert into emp2 values (7,'ggg',9100,777,'it');
insert into emp2 values (8,'hhh',9900,666,'saler');

```



## 过滤查询

1. 语法结构：`select ... from ... where...`
2. 

## 非关联子查询

1. 将子查询当作一张表使用
2. 将子查询当作一个集合来使用（子查询结果必须只有一列）

## 关联子查询
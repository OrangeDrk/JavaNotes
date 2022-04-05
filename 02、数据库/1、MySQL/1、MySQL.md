[TOC]

# 介绍

## 数据库的概念

- 数据库：存储数据的仓库
- 数据库类型：【层次式数据库，网络式数据库】，关系型数据库

## 关系型数据库

- 常用的关模型来存储的数据的数据库叫做关系型数据库
  - 常见数据库
    - 商业数据库
      - Oracle
      - SQL Server
      - DB2
      - Sybase
    - 开源数据库
      - MySQL
      - SQL Lite





## 登陆MySQL

- 方式一：搜索mysql命令窗口，然后直接点击输入密码

- 方式二：打开cmd命令提示符窗口，然后输入命令

  - `mysql -u root -p`—回车
  - `enter password:`—输入密码123456

  ```
  C:\Users\Administrator>mysql -u root -p
  Enter password: ******
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 9
  Server version: 5.7.27 MySQL Community Server (GPL)
  
  Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
  
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  ```
  
  



------



# database基本命令

## 创建一个数据库

1. 默认字符集utf8

   > 默认字符集情况下，大小写不敏感

   `create database test01;` 

2. 设置字符集为gbk

   > `character set`：指定数据库采用的字符集

   `create database test02 character set gbk;`

3. 大小写敏感问题： A=a 是相等的吗？？

   > `collate`校对规则
   >
   > `utf8_general_ci` ：大小写不敏感
   >
   > `utf8_bin`：大小写敏感

   - 大小写不敏感：`create database test02 character set utf8 collate utf8_general_ci;`
   - 大小写敏感：`create database javaweb02 character set utf8 collate utf8_bin;`

4. 判断是否存在，存在不创建，不存在则创建

   > `if not exists`

   `create database if not exists javaweb;`

```mysql
mysql> create database test01;
Query OK, 1 row affected (0.03 sec)

mysql> create database test02 character set gbk;
Query OK, 1 row affected (0.01 sec)

mysql> create database test03 character set utf8 collate utf8_general_ci;
Query OK, 1 row affected (0.07 sec)
```



------

## 获取所有的数据的配置文件路径

1. 模糊查询

   > `like(%con%)`
   >
   > - like：模糊查询，匹配字符
   > - %%：之间写要匹配的内容，任意一对匹配即可

```mysql
show variables like '%con%';# variables：变量
show variables like '%dir%';
```



------

## 显示、删除数据库

- 显示所有的数据库

  > `show database;`

  ```mysql
  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | sys                |
  | test               |
  +--------------------+
  5 rows in set (0.00 sec)
  
  ```

- 显示数据库创建语句

  > `show create database test01;`

- 数据库删除语句

  - 正常删除
  
    `drop database test01;`
    
  - 扩展：
  
    > 在没有test08库下，删除test08
  >
    > `mysql> drop database test08;`//报错
    >
    > `mysql> drop database if exists  test08;`//警告
    >
    > ==加if exists后就不会报错==
  
    ```mysql
    mysql> drop database if exists test08;
    Query OK, 0 rows affected, 1 warning (0.00 sec)
    
    mysql> drop database if exists  test08;
    Query OK, 0 rows affected, 1 warning (0.00 sec)
    ```
  
    



------



## 修改数据库

> 改变数据库的字符集以及字符集的检对规则
>
> `mysql> alter database test01 character set gbk;`

```mysql
mysql> alter database test01 character set gbk;
Query OK, 1 row affected (0.00 sec)
```



------



## 选择/使用数据库

> `use xxx;`

```
mysql> use test
Database changed
```

## 查看当前使用的数据库

> `select database();`

```mysql
mysql> select database();
+------------+
| database() |
+------------+
| test       |
+------------+
1 row in set (0.00 sec)
```



------



## 退出命令

> `ctrl+c`键盘组合按键
> `exit;`
> `quit;`

```mysql
mysql> exit
Bye

mysql> quit
Bye
```



------



# table基本命令

> MySQL常用的数据类型
>
> - 字符串型
>   - varchar、char
> - 大数据类型
>   - blob、text
> - 数值型
>   - tinyint、smallint、int、bigint、float、double
> - 逻辑型
>   - bit（1：true/0：false）
> - 日期型
>   - date、time、datetime、timestamp



## 创建表

> 必须先使用一个数据库才能创建表
>
> ==语法格式==：`create table 表名();`

```mysql
create table db_user
(
    id   int(10),
    name varchar(20),
    password varchar(20),
    birthday date
);
		    	
```



## 查看表结构

> 查看一个表的表结构
>
> ==语法格式==：`desc 表名;`

```mysql
desc employee;
```



## 查看所有表

> 查看当前使用的库中的所有表
>
> ==语法格式==：`show tables;`

```mysql
show tables;
```



## 查看表的创建 语句

> 查看创建该表的sql语句
>
> ==语法格式==：`show create table 表名;`

```mysql
show create table user;
```



## 表单字段的约束

> - 定义==主键约束==
>   - `primary key`：主键，不允许为空，不允许重复；
>   - 删除主键：`alter table table_name drop primary key;`
>   - 主键自增：`auto_increment`
>   - ==一个表中可以有多个主键，称为联合主键==
> - 定义==唯一约束==
>   - `unique`
>   - 例如：`name varchar(20) unique`：表明name不能重复
> - 定义==非空约束==
>   - `not null`
>   - 例如：`salary double not null`:表明salary字段不能为空

```mysql
create table employee
(
    id int primary key auto_increment,/*主键、自增*/
    name varchar(200) unique not null ,/*not null不能为空  unique:不可重复*/
    gender char(1) not null ,
    birthday date,
    entry_date date,
    job varchar(10),
    salary double not null ,
    resume text/*文本类型：可以存放篇幅较长的纯文本*/
);
```



## 修改表

> - 添加:==add==
>
>   `alter table table_name add column_name column_type;`
>
> - 修改:==modify==
>
>   `alter table table_name modify column_name column_type;`
>
> - 删除:==drop==
>
>   `alter table table_name drop column_name ;`
>
> - 修改表名:==rename==
>
>   `rename table old_table_name to new_table_name;`
>
> - 修改字段名
>
>   `alter table table_name change column_name new_column_name column_type;`

1. 添加实例

   > 在员工表的基础上添加一个 image 列，类型为：blob

   ```mysql
   alter table employee add image blob;#二进制形式存储，大类型
   ```

2. 修改实例

   > 修改job列，使其长度为60

   ```mysql
   alter table javaweb.employee modify job varchar(60);
   ```

3. 删除实例

   > 删除gender列

   ```mysql
   alter table javaweb.employee drop gender;
   ```

4. 修改表名实例

   > 表名改为employee1

   ```mysql
   rename table javaweb.employee to employee1;
   ```

5. 修改表中字段名实例

   > 列名name修改为username

   ```mysql
   alter table employee change name username varchar(30);
   ```

## 删除表

> 删除选定的表以及表中的所有数据
>
> ==语法格式==：`drop table table_name;`

```mysql
drop table user;
```



------



# 数据库的CRUD

> 数据库表记录CRUD语句
>
> - Insert语句（增加数据）
> - Update语句（更新数据）
> - Delete语句（删除数据）
> - Select语句（查找数据）

## mysql中文乱码

> - mysql有六处使用了字符集，分别为：
>
>   - clinet、connection、database、results、server、system
>
> - client是客户端使用的字符集
>
> - connection是连接数据库的字符集设置类型，如果程序没有指明连接数据库使用的字符集类型，就按照服务器端默认的字符集设置
>
> - database是数据服务器中某个库使用的字符集设定，如果建库时没有指明，将使用服务器安装时指定的字符集设置
>
> - results是数据库给客户端返回时使用的字符集设定，如果没有指明，使用服务器默认的字符集
>
> - server是服务器安装时指定的默认字符集设定
>
> - system是数据库系统使用的字符集设定（utf-8不可修改）
>
>   - `show variables like 'character%';`
>   - `set names gbk;`临时修改当前cmd窗口和mysql的通信编码字符集
>
> - 通过修改my.ini 修改字符集编码
>
>   > 修改此文件时，先停止mysql服务，完成后重新启动
>   >
>   > 停止：`net stop mysql`
>   >
>   > 启动：`net start mysql`
>
> ```ini
> [mysql]
> # 设置mysql客户端默认字符集
> default-character-set=utf8 
> [mysqld]
> #设置3306端口
> port = 3306 
> # 设置mysql的安装目录
> basedir=D:\mysql\mysql-5.7.27-winx64
> # 设置mysql数据库的数据的存放目录
> datadir=D:\mysql\data 
> # 允许最大连接数
> max_connections=200
> # 服务端使用的字符集默认为8比特编码的latin1字符集
> character-set-server=utf8
> # 创建新表时将使用的默认存储引擎
> default-storage-engine=INNODB
> 
> ```



------



## Insert语句

==语法格式==：

```mysql
insert into table_name[参数列表,.......] 
values(column_value,column_value.....);
```

> 向表中插入一条数据
>
> - 插入的数据应与字段的数据类型相同；
>   - 字符串是数值类型时/数值是字符串类型时：mysql会根据类型自动转换再插入
>   - 日期：必须符合日期格式
> - 数据的大小应该在列的定义范围内，例如：不能将一个长度为80的字符串加入到长度为40的列中。
> - 在values中列出的数据位置必须与被加入的列的排列位置相对应。
> - ==字符和日期型数据应包含在单引号中。==
> - 插入空值：不指定或`insert into table value(null)`
> - 如果要插入所有字段可以省写列列表，直接按表中字段顺序写值列表
> - 数据尝试插入数据库失败时，这条记录也会使用一个自增索引，如果插入失败，也会占用一个主键
>
> 

- 向employee表中插入员工信息


```mysql
insert into javaweb.employee values
(
 default,'卤岩','2010-01-01','2019-01-01','掏粪工',100,'这是一个gou信息',null
);
```



------



## Update语句

==语法格式：==

```mysql
update 表名
set 列名=值 [,col2_name=expr2]
[where 条件语句];
```

`where`:表示有条件的关键字，可以在这个关键字之后添加条件内容，sql语句只会影响满足条件的行。

> 作用：
>
> - update语法可以用新值更新原有表行中的各列
> - set子句指示要修改哪些列和要赋予哪些值
> - where子句指定应用更新哪些行。如果没有where，则更新所有的行

- 将所有员工的薪水修改为5000元

  ```mysql
  update employee
  set salary = 5000;
  ```

- 将姓名为“李帅”的员工的薪水修改为3000元

  ```mysql
  update employee
  set salary = 3000
  where username = '李帅';
  ```

- 将姓名为“曹洋”的员工薪水修改为4000元，job改为ccc

  ```mysql
  update employee
  set salary = 4000,
      job    = 'ccc'
  where username = '曹洋';
  ```

- 将“李帅”的薪水在原有基础上加1000元

  ```mysql
  update employee
  set salary = salary + 1000
  where username = '李帅';
  ```

  

## Delete语句

==语法格式：==

```mysql
delete from table_name [where where_definition]
```

> 作用:
>
> - 删除表中的数据
>
> - 如果不使用where子句，将删除表中所有数据
>
> - delete只能删除行，不能删除列。可用update
>
>   `update table_name set 字段名='';`
>
> - delete语句仅删除记录，不删除表本身。如要删除表，使用`drop table`语句
>
>   `drop table table_name;`
>
> - 同inset和update一样，从一个表中删除记录将引起其他表的参照完整性问题，在修改数据库数据时，始终不能忘记一个潜在的问题：==外键约束==
>
> - 删除表中数据也可使用`truncate table`语句
>
>   - truncate:删除整张表然后再重建
>
> delete和truncate的比较：
>
> - `truncate`：删除效率高,但是可能会摧毁表于表之间的关系。单表可以使用，多表不要使用
> - `delete from`：逐条删除表中记录，效率较低，但是不会摧毁表，能维护表结构和表关系。适用于单表和双表

- 删除表中id为73的记录

  ```mysql
  delete
  from employee
  where id = 10;
  ```

- 删除表中所有的记录

  ```mysql
  delete
  from employee;
  ```

- 使用truncate删除表中记录

  ```mysql
  truncate employee;
  ```

  

## Select语句

> 关键字的执行优先级：
>
> `from  >  where  >  select  >  order by`



### 基本select语句

==语法格式==：

```mysql
select [distinct] *|{col1,col2,.....}
from 表名;
```

> 作用：
>
> - 查询数据
> - `select`：指定查询哪些列的数据
> - `col`：指定列名
> - `*`：代表查询所有列
> - `from`：指定查询哪张表
> - `distinct`：可选，指显示结果时，是否提出重复数据

- ​	查询表中所有学生的信息

  ```mysql
  select * 
  from exam;
  ```

- 查询表中所有的学生的姓名和对应的英语成绩

  ```mysql
  select name, english
  from exam;
  ```

- 过滤表中重复数据：`distinct`去重

  > 只要一行中有一个数据与其他行数据不一样，就不算是重复数据

  ```mysql
  select distinct *
  from exam;
  ```

- 过滤英语成绩相同的数据

  ```mysql
  select distinct english
  from exam;
  ```



------



### 含有表达式

==语法格式==

```mysql
select * | {col1 | expression, col2 | expression....}
from 表名
```

==as别名==

> 使用别名时，`as`可省略，直接用空格隔开即可

```mysql
select 列名 as 别名
from 表名
```



- 在所有的学生分数上加特长分10分，并显示

  ```mysql
  #别名带有as
  select name, 
  		chinese + 10 as chHight, 
  		math + 10 as mathHight, 
  		english + 10 as enHight
  from exam;
  
  #别名，省略as
  select name, 
  		chinese + 10 chHight, 
  		math + 10 mathHight, 
  		english + 10 enHight
  from exam;
  ```

- 统计每个学生的总分

  ```mysql
  select name, chinese + math + english as sum
  from exam;
  ```



------



### 有where条件

==语法格式==

```mysql
select xxx
from 表名
where 条件
```

- 查询英语成绩大于90分的同学

  ```mysql
  select *
  from exam
  where english > 90;
  ```

- 查询总分大于200分的所有同学

  ```mysql
  #错误写法：因为关键字执行顺序：from->where->select->order by
  #where语句优先于select，所以where中无法用select中的别名
  select name,chinese+math+english as sum
  from exam
  where sum>200;
  
  #正确写法：
  select name,chinese+math+english as sum
  from exam
  where chinese+math+english>200;
  ```



------



### 有运算符

==where子句中常用的运算符==

**比较运算符**

| 运算符                    | 解释                               |
| ------------------------- | ---------------------------------- |
| < , >,  <=,  >=,  =,   <> | 小于，大于，小于(大于)等于，不等于 |

**逻辑运算符**

| 运算符                | 解释                                                         |
| --------------------- | ------------------------------------------------------------ |
| between...and...      | 显示在某一区间的值                                           |
| `in(set)`             | 显示在in列表中的值。例：`in(100,200)`                        |
| `like '张%'`          | 模糊查询：<br />`%`代表0-多个任意字符；<br />`_`表示一个字符 |
| `is null`             | 判断是否为空：<br />`select * from user where id is null;`   |
| `ifnull(原值,替代值)` | 如果原值为null，则使用代替值<br />`select ifnull(score,0) from exam;` |
| and                   | 多个条件同时成立                                             |
| or                    | 多个条件任意一个成立即可                                     |
| not                   | 不成立。例：`where not(salary>100);`                         |



- 查询英语分数在80-100之间的同学

  > `between...and...`所表示的区间为`[n,m]`

  ```mysql
  select *
  from exam
  where english between 70 and 95;
  ```

- 查询数学分数为75，86，87的同学

  ```mysql
  select * 
  from exam
  where math in(75,86,87);
  ```

- 查询所有姓张的同学的成绩

  ```mysql
  select *
  from exam
  where name like '张%';
  ```

- 查询数学分数>70,语文分数>80的同学

  ```mysql
  select *
  from exam
  where math>70 and chinese>80;
  ```

- 查询数学分>70,或者语文分>80的同学

  ```mysql
  select *
  from exam
  where math>70 or chinese>80;
  ```

- 查询数学不及格（60分）的同学

  ```mysql
  select *
  from exam
  where math not(math>=60);
  
  #写法二：
  select *
  from exam
  where math<60;
  ```

- 查询未参加数学考试（null）的同学

  ```mysql
  select name,ifnull(math,0)
  from exam
  where math is null;
  ```

  

### 有order by

> `order by` 特点：
>
> - 永远写在整个sql语句的最后
> - 是所有关键字中最后一个执行的关键字
>
> 扩展：
>
> - `order by`会严重影响查询效率，使查询速度变得很慢

==语法格式==

> `asc`：升序（默认）
>
> `desc`：降序
>
> `order by` 指定排序的列，可以是表中的列名，也可以是select语句后指定的列名

```mysql
select 字段名1,字段名2,...
from 表名
order by 字段名 asc|desc
```



- 对语文成绩排序后输出

  ```mysql
  #不写参数，默认升序
  select *
  from exam
  order by chinese;
  
  #参数升序
  select *
  from exam
  order by chinese asc;
  
  #参数降序
  select *
  from exam
  order by chinese desc;
  ```

- 对总分按照降序输出

  ```mysql
  select name,chinese+ifnull(math,0)+english as sum
  from exam
  order by sum desc;
  ```

- 对姓关的学生的成绩排序输出

  ```mysql
  select *
  from exam
  where name like '关%' 
  order by chinese;
  ```



### 分组操作

> 分组子句和聚集函数经常在一起使用。表示在分组后，在进行每组的聚集函数执行操作

==语法格式==

```mysql
select 列名,...
from 表名
group by 字段名
```

**图解**：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/外键.jpg)



- 对订单中商品归类后，显示每一类商品的总价

  ```mysql
  select product sum(price)
  from orders
  group by product;
  ```

#### having子句

> 对分组结果进行过滤
>
> ==having和where的区别：==
>
> - where在分组前进行条件过滤
> - having在分组后进行条件过滤
> - 使用where的地方都可以用having替换。但是having可以使用分组函数，而where后不可以使用

- 查询购买了几类商品，并且每类总价大于100的商品

  ```mysql
  select product
  from oreders
  group by product
  having sum(price)>100;
  ```

  



## 聚集函数

### count

> 求表中一共有多少行

==语法格式==

> `count(*)`
>
> `count(1)`：参数可以是任意值，可查询出行数

```mysql
select count(*)|count(列名)
from 表名
[where 判断语句];
```



- 统计一个班有多少个学生

  ```mysql
  #通过count(*)统计
  select count(*)
  from exam;
  
  #通过count(1)统计
  select count(1)
  from exam;
  
  #通过count(id)统计
  select count(id)
  from exam;
  ```

- 统计数学成绩大于90的学生有多少个

  ```mysql
  select count(*)
  from exam
  where math>90;
  ```

- 统计总分大于250的人数

  ```mysql
  select count(1)
  from exam
  where chinese+ifnull(math,0)+english>250;
  ```



### sum

> 求某一列的和

==语法格式==

```mysql
select sum(列名) {,sum(列名),...}
from 表名
[where 条件]
```

> 注意：
>
> - sum仅对数值起作用，否则会报错
> - 对多列求和，`,`号不能少



- 统计一个班级数学总成绩

  ```mysql
  select sum(math) as mathSum
  from exam;
  ```

- 统计一个班语文、英语、数学各科的总成绩

  ```mysql
  select sum(chinese) cSum,
  		sum(english) eSum,
  		sum(ifnull(math,0)) mSum
  from exam;
  ```

- 统计一个班级语文、英语、数学的成绩总和

  ```mysql
  select sum(chinese+english+ifnull(math,0))
  from exam;
  
  #写法二
  select sum(chinese)+
  		sum(ifnull(math,0))+
  		sum(english)
  from exam;
  ```

- 统计一个班级语文成绩的平均分

  ```mysql
  select sum(chinese)/count(chinese) chAvg
  from exam;
  ```



### avg

> 求平均值

==语法格式==

```mysql
select avg(列名) {，avg(列名),...}
from 表名
```



- 求一个班级数学的平均分

  ```mysql
  select avg(ifnull(math,0))
  from exam;
  ```

- 求一个班级总分的平均分

  ```mysql
  select avg(chinese+ifnull(math,0)+english) avg
  from exam;
  ```



### max/min

> 求最大值/最小值

==语法格式==

```mysql
select max(列名)/min(列名)
from 表名
[where 条件]
```



- 求班级英语最高分和最低分

  ```mysql
  select max(english) max,
  		min(english) min
  from exam;
  ```

- 求班级最高语文成绩的学生姓名和成绩值

  ```mysql
  select *
  from exam;
  where chinese = 
  	(
          select max(chinese) 
          from exam;
      )
  ```

  

------



# 库的导出&导入

## 导出

> 可视化工具导出

- IDEA中，右键要导出的database，选择：dump with mysqldump
- 窗口中选择mysqldump.exe
- 下方out path选择要导出的路径，最后点Run

> cmd窗口导出（不常用）

- mysqldump -u 用户名 -p 数据库名 > 文件名.sql
  - mysqldump -u root -p 
  - db_name > d:/1.sql



## 导入

> 可视化工具IDEA

- 选择目标数据库，右键——>Run Sql Script
- 选择导出的.spl文件即可

> cmd窗口导入（不常用）

- mysql –u 用户名 -p 数据库名 < 文件名.sql  
  - mysql -u root -p db_name < d:/1.sql
  - mysql -u root -p mydb3 < d:/1.sql



------



# 外键

> 只是描述一种两表之间的关系
>
> 不建议使用
>
> - 大数据量大吞吐量的表，有外键的话，影响执行效率
> - 小数据量的表，也可以不适用

- 图示：

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/group-by.png)

- 插入数据时：先向主键中插入信息，再向声明外键的表中插入信息

- 删除数据时：先把声明外键的表中数据删除，再把主键中的数据删除



------



# 多表设计

> 表于表之间的一种关系描述

## 1对1

> 案例：user基本表，userInfo详细表

- user基本表

  ```mysql
  create table user
  (
      id   int primary key auto_increment,
      name varchar(20)
  );
  insert into user values (default, '张三');
  insert into user values (default, '李四');
  insert into user values (default, '王五');
  insert into user values (default, '王六');
  insert into user values (default, '王器');
  insert into user values (default, '王巴');
  ```

- userInfo详细表

  ```mysql
  create table userInfo
  (
      user_id int primary key,
      email   varchar(20),
      phone   varchar(15),
      address varchar(100)
  );
  
  insert into userInfo values (1, '110@qq.com', '110', '家');
  insert into userInfo values (2, '120@qq.com', '110', '家');
  insert into userInfo values (3, '130@qq.com', '110', '家');
  insert into userInfo values (4, '140@qq.com', '110', '家');
  insert into userInfo values (8, '140@qq.com', '180', '家');
  insert into userInfo values (9, '140@qq.com', '190', '家');
  ```

### 查询

- 写法一：

  ```mysql
  select *
  from user,userInfo
  where userInfo.user_id = user.id;
  ```

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/1对多.png)

- 写法二：

  > `join`:内连接
  >
  > - join左边的表 与 join右边的表 通过on的条件去查询，只有当左边的表有记录且 右边的表也有记录，才能查询出来
  > - ![](https://gitee.com/sxhDrk/images/raw/master/imgs/左连接.jpg)

  ```mysql
  select *
  from user 
  join userinfo 
  on user.id=userInfo.user_id;
  ```

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/1对多.png)

- 写法三：

  > `left join` :左连接。
  >
  > - 左表作为主表，去关联右表。
  >- 无论左表中的数据 是否 能够匹配右表，左表的数据都全部显示
  > - ![](https://gitee.com/sxhDrk/images/raw/master/imgs/左连接示意图.jpg)

  ```mysql
  select * 
  from user 
  left join userinfo 
  on user.id=userInfo.user_id;
  ```

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/内连接.jpg)

- 写法四：

  > `right join`:右连接
  >
  > - 右表作为主表，关联左表
  >
  >   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/右连接.jpg)

  ```mysql
  select * 
  from user 
  right join userinfo 
  on user.id=userInfo.user_id;
  ```

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/右连接示意图.jpg)

- 写法五：笛卡儿积查询（==执行结果没有意义==）

  > - 没有on查询条件
  >
  > - 表1有m条数据，表2有n条数据，匹配结果是：m*n条数据
  >
  > 缺点：
  >
  > - 全匹配的效果，执行效率很慢，执行结果没有意义
  > - 以后在使用多表查询时，一定要要加上关联条件（on ....）

  ```mysql
  select * from user,userinfo;
  ```

  

## 1对多

> 案例：emp员工表，dept部门表
>
> 一个部门对应着多名员工

- emp员工表

  ```mysql
  create table emp(
      id int primary key auto_increment,
      dept_id int,/*对应部门表的id主键*/
      name varchar(20)
  );
  
  insert into emp values (default,100,'张三');
  insert into emp values (default,100,'李四');
  insert into emp values (default,100,'王五');
  insert into emp values (default,100,'王六');
  ```

  

- dept部门表

  ```mysql
  create table dept(
      dept_id int primary key auto_increment,
      dept_name varchar(20)
  );
  
  insert into dept values (100,'大数据开发部');
  insert into dept values (200,'大数据运维部');
  ```
  
  

### 查询

- 查询每一名员工所在的部门信息

  ```mysql
  select * 
  from emp 
  left join dept 
  on emp.dept_id=dept.dept_id;
  ```

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/1对多左连接.jpg)

- 查询一个部门下所有的员工

  ```mysql
  select * 
  from emp 
  right join dept 
  on emp.dept_id=dept.dept_id;
  ```

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/1对多右连接.jpg)

## 多对多

> 案例：teacher表，student表
>
> 多对多关系下，必须要有一个关系表，来维护两表之间的联系
>
> 一个老师对应多个学生，一个学生对应多个老师

- teacher表

  ```mysql
  create table teacher(
     t_id int primary key auto_increment,
     t_name varchar(20),
     t_age int
  );
  insert into teacher values (default,'曹洋',38);
  insert into teacher values (default,'许宁',38);
  insert into teacher values (default,'朴乾',38);
  ```

  

- student表

  ```mysql
  create table student(
      s_id int primary key auto_increment,
      s_name varchar(20),
      s_age int
  );
  insert into student values (default,'梁山伯',20);
  insert into student values (default,'祝英台',18);
  insert into student values (default,'西门庆',19);
  ```

  

- t_s关系表

  > 多对多的关系下：必须要维护一个关系表

  ```mysql
  create table t_s(
      t_id int,/*与老师表的id对应*/
      s_id int/*与学生表的id对应*/
  );
    #一个老师有多个学生，一个学生对应多个老师
    insert into t_s values (1,1);
    insert into t_s values (1,2);
    insert into t_s values (1,3);
    #第二个老师的学生
    insert into t_s values (2,1);
    insert into t_s values (2,2);
    insert into t_s values (2,3);
    #第三个老师
    insert into t_s values (3,1);
    insert into t_s values (3,2);
  ```

  


### 查询

- 查询每一个老师下面对应的学生的信息，要求：显示老师信息同时显示学生信息

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/多对多查询.jpg)

  ```mysql
  select tt.*,s.s_name,s.s_age
  from(select t.*,ts.s_id 
       from teacher t 
       join t_s ts 
       on t.t_id=ts.t_id
  	) tt join student s on tt.s_id=s.s_id;
  
  ```

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/多对多.jpg)





------



# 面试题

## delete与truncate的区别

- delete：
  - delete from删除表中的数据
  - 逐条删除，效率较低，但不会摧毁表，能维护表结构和表关系。适用于单表或多表的删除
- truncate：
  - 删除整张表然后再重建
  - 删除效率较高，但是可能会摧毁表于表之间的关系。适用于单表的删除

## select语句的*和单独写索引效率问题

一个sql的查询效率和索引有很大关系。

- 如果一个表中有索引，将会大大提升查询效率
- 使用`*`号和书写全部字段都会使用id这个索引，只要使用索引，他们的查询效率都会很高
- 如一定要去分两者效率，则书写全部字段比书写*号要快一些
  - 原因：
    - `*`号需要判断当前表中使用了哪些字段
    - 直接书写字段名称：数据库只需要根据指定的字段名称查询即可，无需判断使用了哪些字段

## count(*)和count(1)谁的效率更高

`count(*)`和`count(1)`两者的执行效率相似。

- 如果表中的字段数量较少，则两者的查询效率几乎一致

- 如果字段中字段数量较多，
  - `count(1)`查询效率稍快
  - `count(*)`在计数之前需要统计有多少个字段，这个统计时间十分短暂，可以忽略不计。
- 所以在字段数量较少的时候，两者的查询效率是一致的

数据库中存在范式这个概念。范式是用来规定当前数据库建表时的规范。这个规范规定了如何创建一个表。例如：字段中的数据含义必须独立。字段之间关联性降至最低等

按照这些规范来创建的表，字段数量一定不会很多，所以字段数量较多这种情况，一般不会存在。所以就不会考虑`count(*)`统计多个字段的时间。

最终可以认为`count(*)`和`count(1)`两者的执行效率是一致的。
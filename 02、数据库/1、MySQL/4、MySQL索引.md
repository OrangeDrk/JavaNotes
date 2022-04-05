[TOC]





# Mysql索引

> ==索引==是帮助MySQL高效获取数据的==排好序==的==数据结构==

索引数据结构：

- 二叉树：不适合
- 红黑树：树深度太大
- Hash表：无序，无法排序，无法范围查询
- ==B+Tree==：MySQL中使用的索引结构



# 引入索引

## 二叉树索引

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111351.png)

每一个节点保存的是key-value数据

- key：索引值
- value：该数据的存放地址

> 比如：col2为索引字段
>
> `select * from user where Col2 = 89`
>
> - 会先在树中找到索引值89
> - 获取value值（该数据存放地址），然后通过地址拿取数据

当索引值为自增时，二叉树就变成了链表，==所以MySQL数据库不采用这个索引结构==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111352.png)

## 红黑树索引

> 每次新增数据，都会进行大量的平衡判断，而且数据量特别大的时候，红黑树的深度会很大，再进行搜索时会比较耗时
>
> ==所以MySQL索引结构不采用红黑树==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111353.png)

## Hash表

通过索引值进行hash散列计算，算出来的值和数据内存地址存放到映射表中

在对数据进行查询时，通过key的散列值查找数据内存地址。所以无法实现范围查询、比较查询等



## B-树

- 叶节点具有相同的高度，叶节点的指针为空
- 所有索引元素不重复
- 节点中的数据索引从左到右递增排序

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111354.png)

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111355.png)

## B+树

> B-Tree的变种

- 非叶子节点不存储data，只存储索引（冗余索引），可以放更多的索引
- 叶子节点包含所有索引字段
- 叶子节点用指针连接，提高区间访问的性能

> 每一个叶子节点默认16384字节（16KB）
>
> 每个索引节点：
>
> - 索引值：8B
> - 指针：6B
>
> 所以，每个叶子节点（16KB）/每个索引节点（14B）= 1170个索引节点。 所以B+数在高度为3的时候，可以存储的索引节点为：1170×1170×16 ≈ 2千万
>
> ==所以索引可以支持千万级别表的快速查找==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111356.png)

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111357.png)



# 数据库存储引擎

## MyISAM

MyISAM索引文件和数据文件是分离的（非聚集索引）





> 存储结构
>
> ==叶子节点存放的是数据的地址==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111358.png)

> 数据库表的设置

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111359.png)

> 磁盘存储形式
>
> - xxx.frm：存储表的结构信息
> - xxx.MYD：存储表数据
> - xxx.MYI：存储表索引结构

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111400.png)









## InnoDB

InnoDB索引文件和数据文件是放在一起的（聚集索引）

- 表数据文件本身就是按照B+Tree组织的一个索引结构文件

- 是一个聚集索引

- InnoDB表必须有主键，并且推荐使用==整形==的==自增主键==。

  - 默认会拿主键id作为聚集索引
  - 如果没有主键，会取非空的唯一索引作为聚集索引
  - 如果以上都没有，InnoDB会自己维护一个唯一id作为聚集索引

  > ==整形==：整形数字的比较速度快，UUID字符串还要转换成ASCII表去比对
  >
  > ==自增主键==：InnoDB索引结构中，叶子节点通过指针连接，形成了有序的链表，如果主键不是自增，新增一个中间的主键，会导致索引表的分裂，重新整理
  >
  > ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111401.png)
  >
  > 结果变成
  >
  > ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111402.png)
  >
  > ==主键不自增，插入后会进行额外操作，这个过程会很耗时==

  



> 存储结构
>
> ==叶子节点会存放该数据的所有字段==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111403.png)



> 数据库表的设置

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111404.png)



> 磁盘存储形式
>
> - xxx.frm：存储表的结构信息
> - xxx.ibd：存储表的索引+数据

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111405.png)



### InnoDB中的线程

(1) Master Thread

- 刷新脏页到磁盘
- 将日志缓冲刷新到磁盘
- undo页回收
- 合并插入缓冲

(2) IO Thread

- 通过Async IO 处理IO请求

(3) Purge Thread

- 回收已经不在使用的undo页

  > 事务提交之前，通过undolog（回滚日志）记录事务开始之前的状态。
  >
  > 当事务提交之后，该undolog日志就不在需要，需要Purge Thread线程回收

(4) Page Cleaner Thread

- 脏页刷新，减轻Master负担，提升性能



# 索引类型

## 单值索引

> 表结构：（age,name,addr）

以单列为索引。以age为单值索引

一个表可以有多个单值索引，age可以，name也可以

```sql
create index name_index on user(name)
```



## 唯一索引

唯一索引不能重复。

比方说age就不能当作唯一索引，因为很多人都可能使18岁

```sql
create unique index name_index user(name)
```

==一般使用id作为单值索引==

## 复合索引(联合索引)

由多个列构成的索引，相当于书的二级目录

```sql
create index name_age_index on user(name,age)
```

`(name,age)`:先通过name查询，name一样再通过age查询

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111406.png)

> 复合索引的列不一定都能用到（==最左前缀原则==）

如User表的name和city加联合索引就是(name,city)。而==最左前缀原则==指的是，**如果查询的时候查询条件精确匹配索引的左边连续一列或几列，则此列就可以被用到**。如下：

```sql
select * from user where name=xx and city=xx ; //可以命中索引
select * from user where name=xx ; // 可以命中索引
select * from user where city=xx ; // 无法命中索引            
```

- 第一句SQL：
  - 如果查询出多个name，则再去查询city
  - 如果查询出一个name，那就没有必要再去查询city了
- 第二句SQL：因为只查询name，所以city就不用再去查询了

> 需要注意的是，查询的时候如果两个条件都用上了，但是顺序不同，如 `city= xx and name ＝xx`，那么现在的查询引擎会自动优化为匹配联合索引的顺序，这样是能够命中索引的。
>
> 由于最左前缀原则，在创建联合索引时，索引字段的顺序需要考虑字段值去重之后的个数，较多的放前面。ORDER BY子句也遵循此规则。



## 避免冗余索引

冗余索引指的是索引的功能相同，能够命中就肯定能命中 ，那么就是冗余索引。如`(name,city)`和`(name)`这两个索引就是冗余索引

- 能够命中后者的查询肯定是能够命中前者的
- 在大多数情况下，都应该尽量扩展已有的索引而不是创建新索引。

MySQL 5.7 版本后，可以通过查询 sys 库的 `schema_redundant_indexes` 表来查看冗余索引



## 为字段添加索引

1. 添加Primary key（主键索引）

   ```sql
   alter table `table_name` add PRIMARY KEY (`column`)
   ```

   

2. 添加unique（唯一索引）

   ```sql
   alter table `table_name` add UNIQUE (`column`)
   ```

   

3. 添加Index（普通索引）

   ```sql
   alter table `table_name` add index index_name (`column`)
   ```

   

4. 添加FullText（全文索引）

   ```sql
   alter table `table_name` add FULLTEXT (`column`)
   ```

   

5. 添加多列索引

   ```sql
   alter table `table_name` add index index_name (`column`,`column`,`column`)
   ```



# 主键索引和非主键索引

> 例如下表，主键为ID，K为非主键索引

| ID   | name | age  | K    |
| ---- | ---- | ---- | ---- |
| 100  | 张一 | 10   | 1    |
| 300  | 张二 | 10   | 2    |
| 400  | 张三 | 10   | 3    |
| 600  | 张四 | 10   | 4    |

> 主键索引和非主键索引示意图

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111407.png)

其中R代表一整行数据

从图中不难发现，主键索引和非主键索引的区别是：

- 非主键索引的叶子节点存放的是==主键值==
- 主键索引的叶子节点存放的是==整行数据==

> 其中非主键索引也被称为==二级索引==
>
> 主键索引也被称为==聚簇索引==

查询过程：

- `select * from user where ID=100`:主键查询，只需要搜索ID这颗B+树

- `select * from user where K=1`:非主键查询

  - 先搜索K索引树，得到主键值：100
  - 再到ID索引树搜索，得到整行数据

  > 此过程也被称为==回表==



# SQL优化

SQL优化是一种概率层面的优化，SQL会通过SQL优化器自动再进行优化。可以通过explain关键字查看优化过程

```sql
explain select * from user where name='zhangsan'
```



## 多表查询优化

```sql
select * 
from teacher2 t left join course2 c 
on t.cid = c.cid 
where c.cname='java'
```

索引往哪张表加？

- 小表驱动大表

  > 小表：10个条
  >
  > 大表：300条
  >
  > where 小表.x(10) = 大表.y(300);	---> 循环10次
  >
  > ​				大表.y(300) = 小表.x(10);	---->循环300次
  >
  > 当编写`.. on t.cid = c.cid`时，将数据量小的表放左边

- 索引建立在经常使用的字段上

  > 如上述查询，`t.cid=c.cid`，t.cid字段每循环一次使用300次，所以给该字段加索引
  >
  > ==一般左外连接，给左表加索引；右外连接，给右表加索引==



## 其他优化方法

### `exist`和`in`

`select .. from table where exist (子查询)`

`select .. from table where column in (子查询)`

- 如果主查询的数据集大，使用in
- 如果子查询的数据集大，使用exist

> `exists`用法：将主查询的结果，放到子查询中进行匹配。匹配到了返回匹配数据；匹配不到返回空
>
> - `select tname from teacher where exists (select * from teacher where tid=999)`
>
> `in`用法：将主查询字段放到子查询中匹配
>
> - `select * from teacher where id in (select id from B) `





## 避免索引失效

1. 复合索引

   - 不要跨列或者无序使用（最左前缀原则）

   - 尽量使用全索引匹配

     > 比方说索引是（a,b,c），查询时就全部使用上

   - 左侧索引失效，右侧也失效；右侧失效左侧不影响

   - 不要再索引上进行任何操作，不然索引会失效

     > 操作：计算、函数、类型转换
     >
     > 失效操作：`select ... where A.x*3=....`，x为索引，计算后会失效

   - 复合索引不能使用`!=`或`is null`、`is not null`，否则自身以及右侧索引会失效

2. 补救。尽量使用索引覆盖，使用设置的所有索引

   > 索引是：(a,b,c)
   >
   > 使用：`select a,b,c from xx...where a=..and b=...`

3. like尽量以==常量==开头，不要以`%`开头，否则索引失效

   > `select * from xx where name like '%x%'`会失效。如果一定要使用%，可以使用索引覆盖
   >
   > `select * from xx where name like 'x%'`

4. 不要使用类型转换（显示、隐式），否则索引失效

   > `select * from teacher where tname='abc'`：不会失效
   >
   > `select * from teacher where tname=123`：类型转换导致索引失效

5. 不要使用`or`，否则索引会失效

   > `select * from teacher where tname='' or tcid>1 `:or左侧索引失效

### 可以用到索引的情况

```
like "aa%"
<
<=
=
>
>=
between
in
```

### 失效的情况

```
<>
not in
!=
```



# 慢查询

慢查询日志：MySQL提供的一种日志记录，用于记录MySQL中响应时间超过阈值的SQL语句（long_query_time，默认10秒）

1. 检查是否开启了慢查询日志：`show variables like '%slow_query_log%'`

   - 临时开启：`set global slow_query_log = 1`

   - 永久开启：
     - 在`/etc/my.cnf`中添加配置

       ```sh
       [mysqld]
       slow_query_log=1
       slow_query_log_file=/var/lib/mysql/localhost-slow.log
       ```

       

2. 查询慢查询阈值：`show variables like '%long_query_time%';`

   - 临时设置阈值：`set global long_query_time =5 `（修改完后重新登录生效）

   - 永久设置阈值

     - 在`/etc/my.cnf`中添加配置

       ```sh
       [mysqld]
       long_query_time=3
       ```

       重启服务生效

3. 查看慢查询SQL

   `show global status like '%slow_queries%';`，会显出慢查询个数。具体是哪个语句，需要在日志中查看






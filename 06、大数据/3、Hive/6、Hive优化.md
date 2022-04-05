[TOC]



# 优化

## map side join

如果a join b，且a表较小，b表较大，那么可以考虑将a表放到内存中，然后专心处理b表，如果处理过程中需要a表的数据，那么从内存中将a表的数据找出来，而不需要从HDFS上再读取数据；

默认小表的大小不过25M；要求小表 join 大表，不能是大表 join 小表。

通过`set hive.auto.convert.join=true;`



## join

如果出现join和where，先where减少整体数据量，然后再进行join

优化前：

```mysql
select m.cid,u.id 
from order m join customer u 
on m.cid=u.id 
where m.dt=’20160801’; 
```

优化后：

```mysql
elect m.cid,u.id 
from (select cid from order where dt=’20160801’)m 
join customer u 
on m.cid = u.id
```



## group by

将相同的值放到一组，因为数据本身有倾斜特性(因为数据本身不均匀)，就会导致有的组数据多，有的组数据少，分到数据少的组的ReduceTask执行的就快；分到数据多的组的ReduceTask执行的就慢，这就导致了数据倾斜。

通过`set hive.groupby.skewindata=true;`来启动二阶段聚合



## distinct和聚合函数

count属于聚合函数，当聚合函数和distinct同时出现的时候，可以考虑先用多个ReduceTask去重，最后再利用一个ReduceTask来聚合

优化前：`select count(distinct id )from tablename`

优化后：`select count(*) from (select distinct id from tablename)tmp; `



1. 优化前

   - 由于对id=引入了distinct操作，所以在Map阶段无法利用combine对输出结果去消重，必须将id作为key输出
   - 在reduce阶段再对来自于不同的MapTask的结果进行消重，计入最终统计值
   - 由于ReduceTask的数量默认为1，所以导致MapTask的所有结果都只能由这一个ReduceTask处理，这就使得ReduceTask的执行效率成为整个任务的瓶颈
   - 虽然在使用hive的时候可以通过set     mapred.reduce.tasks设置ReduceTask的数量，但是Hive在处理COUNT这种“全聚合(full aggregates)”计算时，它会忽略用户指定的Reduce     Task数，而强制使用1

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/distinct-count优化前.png)

2. 优化后

   - 利用Hive对嵌套语句的支持，将原来一个MapReduce作业转换为两个作业：在第一阶段选出全部的非重复id，在第二阶段再对这些已消重的id进行计数
   - 在第一阶段我们可以通过增大Reduce的并发数，并发处理Map输出
   - 在第二阶段，由于id已经消重，因此COUNT(*)操作在Map阶段不需要输出原id数据，只输出一个合并后的计数即可。这样即使第二阶段Hive强制指定一个Reduce     Task，极少量的Map输出数据也不会使单一的Reduce Task成为瓶颈
   - 这一优化使得在同样的运行环境下，优化后的语句执行只需要原语句20%左右的时间

   ![](G:\笔记\java笔记\大数据\4、Hive\img\distinct-count优化后.png)



## 整理切片个数

如果数据字段比较多处理逻辑比较复杂，可以考虑将切片调小增多任务数；

如果数据字段少处理逻辑比较简单，可以考虑将切片调大减少任务数



## JVM重用

每一个MapTask或者ReduceTask都会启动和关闭一次JVM子进程，那么如果一个节点接受到大量的MapTask或者ReduceTask就需要频繁的开启和关闭JVM子进程。

可以考虑JVM重用，即JVM子进程启动之后执行多个MapTask或者ReduceTask再结束，而不是执行一次就关闭，这样子可以减少JVM子进程频繁的开启和关闭，节省资源
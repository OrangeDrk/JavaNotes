[TOC]



# 概述

1. Sqoop是Apache提供的工具，用于HDFS和关系型数据库之间数据的导出和导入
2. 可以从HDFS导出数据到关系型数据库，也可以从关系型数据库导入数据到HDFS



# 安装和使用

1. 准备Sqoop安装包，官网地址：http://sqoop.apache.org
2. 配置jdk环境变量和Hadoop的环境变量。因为Sqoop在使用时回去找环境变量对应的路径，从而进行完整的工作
3. 解压Sqoop的安装包
4. 在关系型数据库中新建一个数据库`sqoop`，对应的表`order`
5. 需要将连接的数据库**驱动包**下载到Sqoop的**lib**目录下
6. 在Sqoop的**bin**目录下进行命令操作



# 基础指令

| 说明                     | 指令示例                                                     |
| ------------------------ | ------------------------------------------------------------ |
| 查看mysql所有数据库      | sh sqoop list-databases --connect  jdbc:mysql://192.168.150.138:3306/  -username root -password root |
| 查看指定数据库下的所有表 | sh  sqoop list-tables --connect jdbc:mysql://hadoop02:3306/hive -username root  -password root |
| 关系型数据库 ->hdfs      | 实现步骤：     先在mysql数据库的test数据下建立一张tabx表，并插入测试数据    <br />建表：create table tabx (id int,name varchar(20));  <br />插入：insert into tabx (id,name) values  (1,'aaa'),(2,'bbb'),(3,'ccc')，(1,'ddd'),(2,'eee'),(3,'fff');     <br />进入到sqoop的bin目录下，执行导入语句：<br />sh sqoop import --connect jdbc:mysql://192.168.150.138:3306/test   --username root --password root --table tabx --target-dir '/sqoop/tabx'       --fields-terminated-by '\|' -m 1; |
| hdfs ->关系型数据库      | 执行：sh sqoop export  --connect jdbc:mysql://192.168.150.138:3306/test --username root --password root  --export-dir '/sqoop/tabx/part-m-00000' --table taby -m  1 --fields-terminated-by '\|'     注：sqoop只能导出数据，不能自动建表。所以在导出之前，要现在mysql数据库里建好对应的表 |
| sh sqoop import -help    | 查看import的帮助指令                                         |



# 案例

> 将HDFS的`/order/order.txt`导入数据库

1. 在MySQL创建库和表

   ```mysql
   create database sqoop
   create table orders(oid int,odate varchar(25),pid int,num int);
   ```

2. 在sqoop的bin目录下执行命令

   ```sh
   sh sqoop export --connect jdbc:mysql://hadoop01:3306/sqoopdemo --username root --password root --export-dir '/order/order.txt' table orders -m 1 --fields-terminated-by ' '
   ```
   
3. 查看MySQL中数据库信息

   ```mysql
   select * from orders;
   ```

   
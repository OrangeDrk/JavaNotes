[TOC]

# MyCat介绍

## MyCat

mycat国内性能非常高，企业级数据库中间件。一些常见的互联网应用，双11，京东等都在使用mycat



## 特点

> 性能高

1. 支持水平分片，分片表格总数据量百亿级别（官方指南说千亿级别）。一般数据量到达千亿，都会使用大数据存储技术，hdfs、hbase等等
2. 读写分离性能速度高。取决于底层计算逻辑。而且不是任何一个中间件都能支持读写分离



## mycat使用结构

### 结构

中间件：类似于代理的意思。之前客户端直接访问数据库，有了中间件之后，客户端系统要通过中间件才能使用后端数据库的集群

![](https://gitee.com/sxhDrk/images/raw/master/imgs/mycat结构.png)

### 工作逻辑

1. 客户端软件系统整理sql，发送给中间件mycat

2. mycat接收到sql语句，开始计算数据分片

   - 通过sql语句的解析，得到计算分片结果
   - 找到对应数据库分片位置
   - 一个数据的分片可能是主从结构的数据库集群

3. 开始计算读写分离

   - select、show都是读操作，访问主从结构中的从节点，或者主节点，都能实现读逻辑
   - creat、drop、insert等，都是写操作，访问主节点执行

4. 连接真实库

   > 确定了当前操作的sql语句具体要访问后端哪个数据库节点，从而从连接池中，获取连接资源发送sql到后端真实库，并将返回数据返回给客户端使用

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/mycat连接真实库.png)



## mycat连接数据库假死

==原因==：中间件连接后端数据库的线程逻辑是阻塞线程，导致假死的出现

![](https://gitee.com/sxhDrk/images/raw/master/imgs/mycat假死.png)

==假死==：当中间件连接后端数据库的线程是阻塞线程，从【sql—mycat—连接后端数据库获取信息】，这条路一直处于繁忙状态。很可能在高并发访问中间件时，有一段时间（时间很短）会发生线程被占满了，导致错误的判断该连接的这个后端数据库宕机，进行故障转移，记录宕机等操作，其实后端数据库没有宕机

==mycat在这里使用连接后端数据库的线程是非阻塞线程NIO==，所以不会出现假死现象

> 阻塞线程和非阻塞线程
>
> 线程被分配就是执行代码任务
>
> - 阻塞线程：在等待调用结果返回之前，是挂起状态（占用），不能执行其他任务
> - 非阻塞线程：在等待调用结果返回之前，可以执行其他功能
>
> 高并发下区别：
>
> - 阻塞线程：并发处理能力低
> - 非阻塞线程：并发处理能力高





# MyCat使用



## 安装和启动

### 安装解压

> MyCat是基于Java语言开发的，运行要在jdk环境下。每个版本的MyCat要求的JDK版本不一样。MyCat1.5要求最低JDK1.7
>
> MyCat登录就是MySQL登录命令，所以需要MySQL环境支持

将MyCat压缩文件解压到`/home/software/`下

```sh
[root@10-9-104-184 software]# tar -xf /home/resources/Mycat-server-1.5.1-RELEASE-20161130213509-linux.tar.gz -C ./
```



### 启动MyCat

1. MyCat目录结构

   - bin：命令脚本所在文件夹
   - conf：所有xml配置文件所在的文件夹
   - logs：日志文件夹

2. 启动MyCat

   > 进入bin目录

   - `mycat start`

     后台运行，日志输出将会进入到logs文件夹中的wrapper.log的文件中

   - `mycat console`
   
     控制台运行，直接打印日志到控制台

### 登录MyCat

1. 使用mysql命令登录

   > 必须指定mycat的端口、运行服务器ip地址、mycat的用户名和密码

   ```sh
   [root@10-9-104-184 conf]# mysql -utest -ptest -h10.9.104.184 -P8066
   ```

   

2. Windows客户端登录

   > sqlYog、Navicat等Windows客户端是专门为SQL数据库准备的，如果登录mycat只能实现一些常用常见的命令发送。



# MyCat的配置文件

> 在mycat根目录有conf文件夹，其中有三个xml配置，都会使用到。
>
> ==一旦涉及到修改xml，习惯备份原文件。==

## `server.xml`

### 标签结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://org.opencloudb/">
    <system>
        <property name="defaultSqlParser">druidparser</property>
    </system>
    <!--mycat的登录用户名和密码，能看到的数据库-->
    <user name="root">
        <property name="password">root</property>
        <property name="schemas">TESTDB</property>
    </user>
    <!--
    <quarantine> 
        <whitehost>
            <host host="127.0.0.1" user="mycat"/>
            <host host="127.0.0.2" user="mycat"/>
        </whitehost>
        <blacklist check="false"></blacklist>
    </quarantine>-->
</mycat:server>
```

### 标签和属性

#### `system`

> 与mycat进程启动时占用的资源、内部配置属性有关

==子标签==：`property`：可以配置压缩格式、端口号、占用系统的线程资源等等

#### `user`

> mycat登录的用户、权限配置等

- `name`：用户名
- `password`：用户密码
- `schemas`：该用户可以访问的mycat数据库，可以访问多个数据库，每个数据库用`,`隔开（必须是mycat中存在的库，如果任何一个库不存在，都会报错）
- `readOnly`：用户只读

#### `quarantine`

> 防火墙配置

- `whitehost`：白名单

  > 只有客户端ip地址在白名单范围内，才允许访问mycat

  `<host host="127.0.0.1"  user="mycat"/>`

  > 只有客户端是从127.0.0.1的ip访问mycat，并且使用mycat用户登录才能登陆成功

- `blacklist`：黑名单

  > 在黑名单中列出的sql规则，都不能使用，会被过滤掉
  >
  > 例如：drop、没有where的delete等不能执行
  >
  > 标签中的值为空的不检查

  `<blacklist  check="true">selectAllColumnAllow<blacklist>`

  > 限制客户端不能使用`select *` 这种语句



## `schema.xml`

### 标签结构

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/" >

    <!--mycat的虚拟库-->
    <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
        <!--绑定真实表-->
        <table name="t1_student" primaryKey="id" dataNode="dn1" />
    </schema>

    <!--分片数据-->
    <dataNode name="dn1" dataHost="localhost1" database="haha" />

    
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM1" url="10.42.0.33:3306" user="root" password="root" />
    </dataHost>
</mycat:schema>
```



### 标签和属性



#### `schema`：数据库对象

> 可以在一个schema.xml文件中配置多个schema标签，每一个都表示客户端可以看到的一个数据库（不是真实的数据库）

```xml
<schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
```

属性：

- `name`：客户端可以看到和使用的数据库名称

  > 配置多个schema标签，对应`server.xml`用户配置标签中的`schemas`属性

- `checkSQLschema`：是否需要添加表格的数据库名称

  > true：添加该表的数据库名称
  >
  > false：不添加该表的数据库名称
  >
  > 例如：student表格，在数据库db1中
  >
  > sql语句：select * from student
  >
  > - false：`elect * from student`
  > - true：`select * from db1.student`

- `sqlMaxLimit`：最大分页查询数

  > ==整数值==，当sql语句做批量查询，mycat防止性能浪费，在判断sql语句中有没有limit关键字，有的话自动拼接limit 0,100  查询前100条。



#### `table`：表格对象

> 在一个schema中，存在的数据库表格，可以有多个table标签，表示多个表格

```xaml
<table name="t1_student" primaryKey="s_id" dataNode="dn1,dn2,dn3" rule="auto-sharding-long" />
```

属性：

- `name`：表格名称

  > 所有表格数据一定来自真实库，表格名称要和真实库对应
  >
  > 并且同一个schema标签中只能存在唯一一个名称（不能有同名的表格）

- `primaryKey`：主键名称

  > 对应表格的真实数据中的主键名字，默认是id
  >
  > 当不是id时，需要配置这个属性 

- `dataNode`：数据库水平分片

  > table表格数据，可以根据数据量大小，实现水平切分，对应的数据分片计算绑定dataNode标签，这里可以设置当前表格被切分到了几个数据分片中
  >
  > 可以对应一个分片，可以对应多个分片。 

- `rule`：分片规则

  > 当表格有对应多个数据分片时，要指定切分数据分片的计算逻辑，可以给配置rule规则。例如：
  >
  > - `auto-sharding-long`  ：整数范围约束,表示以主键某个字段的整数做分片计算
  >   - 0-500万对应第一个分片
  >   - 500万-1000万对应第二个分片
  >   - 1000万-1500万对应第三个分片。
  > - `sharding-by-murmur`：以字符串，使用一致性hash分片  



#### `dataNode`：分片对象

> 在mycat管理的一个主从数据库集群中作为分片使用，需要将集群包装在一个数据分片对象中
>
> 一个`dataNode`标签表示一个数据分片
>
> ==最主要的作用就是计算分片，利用名称、下标实现分片计算==

```xml
<dataNode name="dn1" dataHost="localhost1" database="db1" />
```

属性：

- `name`：分片名称

  > 按照配置顺序、每个`dataNode`标签下标（从0开始）这些条件做一致性hash计算

- `dataHost`：绑定数据库集群管理对象

  > 一个分片不负责数据库集群的管理，只是绑定数据库集群管理对象`dataHost`，dataHost使用名字来绑定。 

- `database`：绑定真实库

  > 指定真实库名称 。
  >
  > 当前分片绑定的真实数据库集群使用，真实数据库中database可能有多个，当前分片用的是哪个库（垂直划分）



#### `dataHost`：数据库集群管理对象

> 负责管理一个真正的数据库主从集群。连接池、读写分离都是由这个标签对象实现的计算。 

```xml
<dataHost name="localhost1"  maxCon="1000" minCon="10" 
          balance="0" writeType="0" dbType="mysql"  
          dbDriver="native" switchType="1"  slaveThreshold="100">
```

属性：

- `name`：当前dataHost的名字，绑定dataNode时使用
- `maxCon`:当前dataHost管理的数据主从集群每个节点的连接池中最大连接
- `minCon`:最小连接
- `balance`:读写分离的读逻辑
- `writeType`:读写分离的写逻辑
- `dbType`:默认mysql，数据库软件类型
- `dbDriver`:默认mysql叫做native，如果是其他数据库给定driver全路径值
- `switchType`:故障转移有关
- `slaveThreshold`:100是毫秒数，表示当前主从集群，从节点sql延迟100毫秒以上时，将不会使用该节点处理读数据逻辑



#### `heardbeat`：心跳检测

> dataHost连接使用后端数据时，实现心跳检测的sql语句，一般有2中常用的：
>
> - `select user()`
> - `show slave status`(只有使用该语句，slaveThreshold:100才能生效) 



#### `writeHost`：写操作主机

> 写主机。表示在一个主从集群中用来负责写操作的主机。==只能在其中配置主节点==

```xml
<writeHost host="hostM1" url="10.202.4.39:3306" 
           user="root" password="sf123456">
```

属性：

- `host`:代号名称，相当于name  一般会使用默认结构 hostM1 第一个主节
- `url`：ip:port（该节点ip+端口） 
- `user`:登录数据库用户名（该节点用户名）
- `password`：登录密码（该节点密码）



#### `readHost`：读操作主机

> 读主机。表示在一个主从集群中用来负责读操作的主机。==主从节点都可以==在这里配置，==一般都是使用从节点==

```xml
<readHost host="hostS2"  url="192.168.1.200:3306" 
          user="root" password="xxx" />
```

属性：

- `host`:代号名称，相当于name  一般会使用默认结构`hostM1S1`表示第一个主节点第一个从节点
- `url`：ip:port（该节点ip+端口） 
- `user`:登录数据用户名（该节点用户名）
- `password`：登录密码（该节点密码）



![](https://gitee.com/sxhDrk/images/raw/master/imgs/mycat配置文件的图解.png)



# 入门案例

## 结构

![](https://gitee.com/sxhDrk/images/raw/master/imgs/mycat配置入门案例.png)

> 最终通过客户端，登录到mcyat 操作`TESTDB.t1_student`，实际数据来源从后端真实库`t1_student`



## 配置

> 选取一个数据测试使用
>
> 创建真实库：`haha`，真实表：`t1_student`。表结构：id，name

修改配置文件：`schema.xml`

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/" >
    <!--mycat虚拟库-->
    <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
        <!--绑定真实的表-->
        <table name="t1_student" primaryKey="id" dataNode="dn1"/>
    </schema>

    <!--分片1-->
    <dataNode name="dn1" dataHost="localhost1" database="haha"/>

    <!--主从集群管理对象-->
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM1" url="10.9.100.26:3306" user="root" password="root"/>
    </dataHost>
</mycat:schema>
```

> 因为案例中只有一个表，没有对该表进行分片操作，所以没有读写分离，不能实现高可用故障转移
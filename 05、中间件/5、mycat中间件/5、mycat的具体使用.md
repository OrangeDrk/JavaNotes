[TOC]





# MyCat读写分离

> 基于高可用结构中的==主从数据库==，实现：
>
> - 主节点写数据
> - 从节点读数据
>
> 的计算分配

## 读写分离的逻辑和结构

![](https://gitee.com/sxhDrk/images/raw/master/imgs/读写分离逻辑和结构1.png)

![](https://gitee.com/sxhDrk/images/raw/master/imgs/分片表.png)



## 修改`schema.xml`文件

> 使用一个`<writeHost>`标签无法完成读写分离，要添加一个`<writeHost>`指向另一台云主机数据库

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/" >

    <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
        <table name="t1_student" primaryKey="id" dataNode="dn1"/>
    </schema>

    <dataNode name="dn1" dataHost="localhost1" database="haha"/>

    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM1" url="10.9.100.26:3306" user="root" password="root"/>
        <writeHost host="hostM2" url="10.9.104.184:3306" user="root" password="root"/>
    </dataHost>
</mycat:schema>
```



## `<dataHost>`属性详解

> `<dataHost>`标签是管理一个数据库主从集群的标签，绑定到`<dataNode>`使用。由他来实现读写分离，涉及到【2个属性】

### 1、`writeType`

> 控制当前`<dataHost>`数据库集群的==写逻辑==

- `writeType=1`：1.5以上mycat不推荐使用。

  > `writeType=1`时
  >
  > - 会覆盖掉读的逻辑，`balance`的值失效。
  > - 读和写的操作都会==随机==的在多个`<writeHost>`中进行

- `writeType=0`

  > `writeType=0`时
  >
  > - 当前的写操作会在配置的第一个`writeHost`进行
  > - 读操作由`balance`的值决定

### 2、`balance`

> 控制当前`<dataHost>`数据库集群的==读逻辑==，但是只有在`writeType=0`时生效

- `balance=0`：

  > ==不开启读的分离==，所有的读都在第一个`<writeHost>`进行

- `balance=1`：

  > ==最常用的一种配置==，都会从所有的`<readHost>`和备用的`<writeHost>`进行读操作。
  >
  > 如果并发比较高，第一个`<writeHost>`也会分得一部分读的操作

- `balance=2`：

  > 随机的在所有的`<writeHost>`中进行读操作

- `balance=3`：

  > 随机的在所有的`<readHost>`进行读操作。
  >
  > 如果没有`<readHost>`，就只会在第一个`<writeHost>`进行读数据





# MyCat高可用故障转移

## 双向热备关系

一个分片中的主从数据库，存在双向的主从。

只要其中一个节点宕机，那么另一个将会负责这个分片的读写逻辑



## MyCat故障转移配置

> `<dataHost>`中的属性`switchType`控制故障转移
>
> - 在一个`<dataHost>`中如果有多个`<writeHost>`，将会在加载时按照书写顺序分配下标号，下标从0开始
> - `writeType`总会在下标为0的`<writeHost>`中进行写操作
> - 当下标为0的`<writeHost>`宕机后，将下标顺延（下标为1的，变为下标0），不会影响写操作，实现故障转移



1. `switchType=-1`：

   > ==不开启故障转移==，当第一个写节点宕机后，整个分片就不可以使用了

2. `switchType=1`：

   > ==开启故障转移==，index=0的`<writeHost>`宕机后，index=1的`<writeHost>`会替换到index=0的位置，实现故障转移，不影响写操作
   >
   > ==宕机节点，在恢复之前，不管是读操作还是写操作，都无法进行==



> **读写分离，故障转移。在一个分片中的数据库集群存在，就能保证这个分片的高性能，高可靠性的运行**





# 分片表的配置

> 准备真实库：mycat
>
> - t1_student:表格id为整数，==使用范围约束计算==
> - t2_user:做分片表格，id为字符串，==使用一致性hash计算==
> - t3_category:商品分类↓
> - t3_product:商品表格 实现==全局表==分片表配置
> - t5_order：订单表↓
> - t6_order_item:==ER分片表==配置

## 表格的类型

> mycat中`<table>`标签能表示客户端使用的一个真实数据库表格。可以根据：配置使用方式、数据量大小的不同，分为两种类型：
>
> - 非分片表
> - 分片表

### 非分片表

> 表格数据全部来自于一个分片，一个数据库主从集群

![](https://gitee.com/sxhDrk/images/raw/master/imgs/读写分离逻辑和结构2.png)



### 分片表

> 如果整体数据量在一个表格中比较庞大（千万级、亿级），可以利用mycat实现分片表的配置
>
> 一个table中对应绑定多个`<dataNode>`，并且利用分片计算规则实现对应分片计算，实现对应分片的读、写操作

![](https://gitee.com/sxhDrk/images/raw/master/imgs/全局表.png)



## `schema`的配置

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/" >

    <!--mycat中的虚拟数据库-->
    <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
        <!--mycat虚拟库中绑定的真实表格-->
        <table name="t1_student" primaryKey="id" dataNode="dn1,dn2" 
               rule="auto-sharding-long"/>
    </schema>

    <!--分片对象-->
    <dataNode name="dn1" dataHost="localhost1" database="mycat"/>
    <dataNode name="dn2" dataHost="localhost2" database="mycat"/>
    
    <!--集群分布式管理对象-->
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <!--写节点，节点ip+端口，账号+密码-->
        <writeHost host="hostM1" url="10.9.104.184:3306" user="root" password="root"/>
    </dataHost>
    
    <!--集群分布式管理对象-->
    <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM2" url="10.9.100.26:3306" user="root" password="root"/>
    </dataHost>
    
</mycat:schema>

```

> 如果重启mycat出现错误：找到错误`Caused by...`提示，按照提示找错



## 分片计算规则

### `auto-sharding-long`:整数范围约束

> 默认逻辑：==以每个表格数据中的列 id为计算分片的字段==,并且是整数计算
>
> 对id计算：
>
> - id值在==[0-500w]==的一批数据计算切分到dn1（第一个数据分片）  
> - ==[500w-1000w]==切分到dn2  （第二个数据分片）  
> - ==[1000w-1500w]==切分到dn3(第三个数据分片)
> - 超过这个范围的1500w以上报错。



### `sharding-by-murmur`：一致性hash约束

> 逻辑：默认以表数据id字段作为计算一致性hash的字段，默认整数计算
>
> - 可以通过修改分片计算规则文件，自定义以哪个字段做计算字段



## 分片计算规则配置文件：`rule.xml`

> 可以定义所有与分片计算有关的规则和代码。
>
> 在mycat的根目录conf文件夹里。

### `<tableRule>`

> 定义规则名称，通过哪一个function规则，以哪个id计算
>
> 基本格式：

```xml
<tableRule name="rule1"> <!--name表示表格分片计算规则名称 auto-sharding-long-->
    <rule>
        <columns>id</columns> <!--该规则在计算分片时使用的表格字段名称-->
        <algorithm>func1</algorithm> <!--使用这个计算规则,rule真正调用的计算逻辑算法的标签:function名称-->
    </rule>
</tableRule>
```



### `function`

> 每一个function都表示了一种算法的详细计算配置。

```xml
<function name="rang-long" //算法名称，绑定tableRule使用
          class="org.opencloudb.route.function.AutoPartitionByLong">
    <!--👆👆👆执行该算法的详细计算代码，代码在mycat的lib库中 👆👆👆-->
    
    <property name="mapFile">autopartition-long.txt</property>
    <!--👆👆👆附加的计算文件，可以根据算法不同，代码不同，属性name值不同，
	整数范围约束中mapFile定义了整数范围对应分片的规则 
	0-500w 500w-1000w 1000w-1500w-->
</function>
```

> `<property>`标签指定分片规则，该文件在`mycat/conf`目录中

```sh
# K=1000,M=10000.
#0-500M=0 #0-500万对应dataNode下标是0
#500M-1000M=1 #对应dataNode下标是1
#1000M-1500M=2 #对应dataNode下标是2
#自定义
0-30M=0	#0-30万在下标为0的dataNode下操作
30M-90M=1 #30万-90万在下标为1的dataNode下操作
```





## 练习：一致性hash分片计算

> 实现一个id值为字符串类型的表格分片计算
>
> 不能使用整数范围约束，可以使用一致性hash

### `tableRule`

```xml
<tableRule name="test1">
    <rule>
        <columns>id</columns> 
        <algorithm>murmur</algorithm> 
    </rule>
</tableRule>
```

### `function`

```xml
<function name="murmur"
          class="org.opencloudb.route.function.PartitionByMurmurHash">

    <property name="seed">0</property>
    <!-- hash桶的种子值，一个mycat中种子值必须唯一，hash数学计算的一个参数，会改变映射结果。
		默认就是0而且几乎不会发生变化 -->
    
    <property name="count">2</property>
    <!-- 利用一致性hash计算分片时，使用分片dataNode的名字做整数映射，
		这个2表示分片个数-->
    
    <property name="virtualBucketTimes">160</property>
    <!-- 一个实际的数据库节点被映射为这么多虚拟节点，默认是160倍，也就是虚拟节点数是物理节点数的160倍 -->
    
    <!-- <property name="weightMapFile">weightMapFile</property> 节点的权重，
	没有指定权重的节点默认是1。以properties文件的格式填写，
	以从0开始到count-1的整数值也就是节点索引为key，以节点权重值为值。
	所有权重值必须是正整数，否则以1代替 
		例如：weightMapFile
			0=1:0号分片权重是1
			1=5:1号分片权重是5
	-->
    
    <!-- <property name="bucketMapPath">/etc/mycat/bucketMapPath</property> 
	详细打印分片映射的整数结果到一个文件，但是没有任何效果 -->

</function>
```

> mycat可以通过权重文件：`weightMapFile`(properties格式)，实现不同的分片权重分配
>
> 0=5：0号下标的节点权重为5
>
> 1=2：1号下标的节点权重为2

![](https://gitee.com/sxhDrk/images/raw/master/imgs/一致性hash分片计算权重.png)





## *自定义分片计算规则*

> 自定义tableRule的标签，自定义计算的字段名称，例如对user_id做计算
>
> 自定义function标签，对应tableRule实现不同字段的计算逻辑。



## 案例：对easymall项目中表格分片

> `t_product`，将其配置在多个数据库中，实现一致性hash的计算
>
> - 定义一致性hash计算规则`rule`：`easymall-product`
> - 使用算法 ：一致性hash

1. 准备真实数据库，并复制到两个云主机数据库中

2. 配置mycat

   - `schema.xml`

     ```xml
     <?xml version="1.0"?>
     <!DOCTYPE mycat:schema SYSTEM "schema.dtd">
     <mycat:schema xmlns:mycat="http://org.opencloudb/" >
     
         <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
             <table name="t1_student" primaryKey="id" dataNode="dn1,dn2" rule="auto-sharding-long"/>
             <table name="t2_user" primaryKey="id" dataNode="dn1,dn2" rule="sharding-by-murmur"/>
             
             <!--案例t_product表-->
             <table name="t_product" primaryKey="product_id" dataNode="dn3,dn4" rule="easymall-product"/>
         </schema>
     
         <dataNode name="dn1" dataHost="localhost1" database="mycat"/>
         <dataNode name="dn2" dataHost="localhost2" database="mycat"/>
         
         <!--案例t_product分区表-->
         <dataNode name="dn3" dataHost="localhost1" database="easydb"/>
         <dataNode name="dn4" dataHost="localhost2" database="easydb"/>
         
         <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
                   writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
                   slaveThreshold="100">
             <heartbeat>select user()</heartbeat>
             <writeHost host="hostM1" url="10.9.104.184:3306" user="root" password="root"/>
         </dataHost>
         <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
                   writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
                   slaveThreshold="100">
             <heartbeat>select user()</heartbeat>
             <writeHost host="hostM2" url="10.9.100.26:3306" user="root" password="root"/>
         </dataHost>
     </mycat:schema>
     
     ```

     

   - `rule.xml`

     ```xml
     <tableRule name="easymall-product">
         <rule>
             <columns>product_id</columns> 
             <algorithm>murmur</algorithm> 
         </rule>
     </tableRule>
     
     <function name="murmur"
               class="org.opencloudb.route.function.PartitionByMurmurHash">
         <property name="seed">0</property>  
         <property name="count">2</property>
         <property name="virtualBucketTimes">160</property>
     </function>
     ```

     

# 跨分片数据查询--全局表

> 在配置的大量分片表格中，有很多表需要进行关联操作join。如果在多个表格的数据中，相关的数据产生了跨分片的存储---mycat没办法跨分片操作相关数据。

## 跨分片的数据

> 当多个表格中有分片表格，相关数据产生了跨分片的存储，mycat无法实现相关查询，返回不正确的数据

![](https://gitee.com/sxhDrk/images/raw/master/imgs/非分片表.png)



## 全局表

> 上述例子中，多个表格，其中有的是
>
> - 分片表格（数据量大）
> - 有的表格是非分片（数据量不大）
>
> 解决他们这种之间的跨分片配置，可以==使用全局表==。

全局表特点：

- 数据量不大
- 数据变化稳定。

常见的全局表---工具字典表。

> 例如：
>
> - 做业务查询时，可以使用工具字典表存储（200--成功，201--，202--….）
>
> - 1101199802090988 解析出来地市，时间，操作人员 关联查询地市表，和实施员工表

全局表的思路：将全局表格完全的复制到多个dataNode

全局的配置：table标签中指定多个dataNode但是没有分片计算规则。默认复制使用。

> 真实表：
>
> - t3_category
>
> - t3_product，分片做到mycat管理的2个table标签中，

其中category就是全局表，product就是正常分片表格，使用的分片计算规则 `auto-sharding-long`



## 全局表配置

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/" >

    <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
        <!--sharded product-->
        <table name="t3_product" primaryKey="id" dataNode="dn1,dn2"
               rule="auto-sharding-long"/>
        <!-- globle table category-->
        <table name="t3_category" primaryKey="id" dataNode="dn1,dn2"/>
    </schema>

    <dataNode name="dn1" dataHost="localhost1" database="mycat"/>
    <dataNode name="dn2" dataHost="localhost2" database="mycat"/>
    
    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM1" url="10.9.104.184:3306" user="root" password="root"/>
    </dataHost>
    <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM2" url="10.9.100.26:3306" user="root" password="root"/>
    </dataHost>
</mycat:schema>

```



## 总结

> 全局表解决的是==非分片表格==与==分片表格==关联的底层跨分片问题。





# 跨分片数据查询--ER分片表

> 多个分片表格有相关数据需要关联操作，更容易在没有合理的分片计算规则计划之下出现跨分片操作。

## 多个分片表跨分片查询

![](https://gitee.com/sxhDrk/images/raw/master/imgs/多个分片跨分片查询.png)

> 多个分片表格如果设计不好分片规则计算，更容易出现跨分片操作



## ER分片

> ER分片表解决思路： 相关的数据，统一  分片计算  方法。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ER分片.png)

> ER分片，可以解决1对1关系表，1对多关系表。但是无法解决多对多关系表

这种1对多关系，可以分为父表和子表

- 父表是1
- 子表是多

> 区分父表子表：明显的特征，子表中有关联字段，父表没有。
>
> 业务意义区分，子表的任何一条数据都不能独立于父表存在

利用父表进行分片计算的统一（所有子表，孙表都是根据父表的关系进行分片计算）。

- 可以使用table配置父表，定义分片，定义计算规则
- 使用table包含的childTable定义与父表的关系。

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/" >

    <schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100">
        
        <!--parent table-->
        <table name="t5_order" primaryKey="id" dataNode="dn1,dn2"
               rule="auto-sharding-long">
            <!--child table -->
            <childTable name="t6_order_item" primaryKey="id" joinKey="o_id"
                        parentKey="id">
            </childTable>
        </table>
    </schema>

    <dataNode name="dn1" dataHost="localhost1" database="mycat"/>
    <dataNode name="dn2" dataHost="localhost2" database="mycat"/>

    <dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM1" url="10.9.104.184:3306" user="root" password="root"/>
    </dataHost>
    <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  
              slaveThreshold="100">
        <heartbeat>select user()</heartbeat>
        <writeHost host="hostM2" url="10.9.100.26:3306" user="root" password="root"/>
    </dataHost>
</mycat:schema>

```

> table定义父表，使用多少分片，使用什么计算规则
>
> - childTable定义子表
>   - joinKey:表示子表中，==与父表的关联字段的名称==
>   - parentKey:子表的关联字段，==在父表中叫的名字==
>
>  



## 多对多

海量的多对多关系无法在mycat实现分片存储。（老师-学生-外键关联表格）

只能使用大数据数据存储，计算之间的关系。

- 比如hadoop。
- hdfs存储--mapreduce计算数据。


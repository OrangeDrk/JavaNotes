[TOC]



# 物理存储原理

## 概述

1. HBase里的一个Table在行的方向上分割为一个或者多个HRegion，即HBase中一个表的数据会被划分成一个或者很多的HRegion，每一个HRegion会被存储在一个节点上。这样做的目的是为了能做到数据的分布式存储

2. HRegion可以动态扩展，并且HBase保证HRegion的负载均衡

3. HRegion实际上是行键排序后，按规则分割的连续的存储空间

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegion-StoreFile.jpg)

4. 一张HBase表，可能有多个HRegion，每个HRegion达到一定大小时，进行分裂

   > ==大小默认为：10G==

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegion3.jpg)



## 拆分过程

1. HRegion是按照大小分割的，每个表一开始只有一个HRegion，随着数据不断插入表，HRegion不断增大，当增大到一个阈值时，HRegion就会等分出两个等大的HRegion。因此随着Table中的行不断增多，就会有越来越多的HRegion

2. 按照现在主流硬件的配置，每个HRegion的大小可以是1~20GB，这个大小由`hbase.hregion.max.filesize`指定，默认为10GB

3. HRegion的拆分和转移是由HBase的`HMaster`进程自动完成的，用户感知不到

4. **HRegion是HBase中分布式存储和负载均衡的最小单元**

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegion1.jpg)

5. **HRegion虽然是分布式存储的最小单元，但并不是存储的最小单元**

6. HRegion由一个或者多个`HStore`组成，每个`HStore`保存一个列族（columns family）

7. 每个`HStore`又由【一个`memStore`】以及【0个或者多个`StoreFile`】组成，`StoreFile`以`HFile`格式保存在HDFS上

   > `memStore`：写缓存，默认大小128MB

8. **总结：HRegion是分布式存储的最小单位，StoreFile(HFile)是存储的最小单位**

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegion2.jpg)



# 系统架构

## HBase基于Hadoop架构图

![](https://gitee.com/sxhDrk/images/raw/master/imgs/HBase与Hadoop架构图.png)



## HBase架构组成

> HBase采用Master/Slave（主从）架构搭建集群，它隶属于Hadoop生态系统，由一下类型节点组成

1. HMaster节点：主节点：Active；从节点：Backup

   - **管理HRegionServer，实现其负载均衡**。保证每一个HRegionServer上的数据量差别不大
   - **管理和分配HRegion**。比如在HRegion分裂时分配新的HRegion；在HRegionServer退出时迁移其内的HRegion到其他HRegionServer上
   - **实现DDL操作**。（Data Definition Language），namespace和Table的增删改，column family的增删改等
   - **管理namespace和table的元数据**。（实际存储在HDFS上）
   - **权限控制（ACL）**

2. HRegionServer节点

   - 存放和管理本地HRegion

   - 读写HDFS，管理Table中的数据

   - Client直接通过HRegionServer读写数据

     > 从HMaster中获取元数据，找到RowKey所在的HRegion/HRegionServer后读取数据

3. ZooKeeper集群

   - 存放整个HBase集群的元数据、集群的状态信息以及RS服务器的运行状态
   - 实现HMaster主备节点的failover



> HBase的数据存储于HDFS中，==RegionServer和DataNode一般会放在相同的Server上==**实现数据的本地化(避免或减少数据在网络中的传输，节省带宽)**



![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegion原理详解.jpg)



![](https://gitee.com/sxhDrk/images/raw/master/imgs/HMaster.png)





> HBase Client通过RPC方式和HMaster、HRegionServer通信；
>
> 一个HRegionServer可以存放1000个HRegion（1000个数字的由来是来自于Google的Bigtable论文）；
>
> 底层Table数据存储于HDFS中，而HRegion所处理的数据尽量和数据所在的DataNode在一起，实现数据的本地化；**数据本地化并不是总能实现，比如在HRegion移动(如因Split)时，需要等下一次Compact才能继续回到本地化**



# 架构原理详解

## HRegion

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ZooKeeper.png)



1. HBase使用RowKey将表水平切割成多个HRegion
2. 从HMaster的角度看，每个HRegion都记录了它的StartKey和EndKey
3. ==由于RowKey是排序的，因而Client可以通过HMaster快速的定位每个RowKey在哪个HRegion==
4. HRegion由HMaster分配到响应的HRegionServer中，然后由HRegionServer负责HRegion的启动、管理、与Client的通信以及负责数据的读取（使用HDFS）
5. 每个HRegionServer可以同时管理1000个左右的HRegion



## HMaster

![](https://gitee.com/sxhDrk/images/raw/master/imgs/HBase与Hadoop架构图3.png)



1. HMaster没有单点故障问题，因为HBase集群可以启动多个HMaster

2. 通过ZooKeeper的Master Election机制保证同时只有一个HMaster处于Active状态，其他的HMaster则处于热备份状态

3. 一般情况下会启动两个HMaster，BackUp的HMaster会定期的和Active HMaster通信以获取其最新状态，从而保证它是实时更新的，因而如果启动了多个HMaster反而增加了Active HMaster的负担

4. HMaster主要用于HRegion的分配和管理，DDL的实现等，既它主要有两方面的职责：

   > DDL：Data Definition Language，既Table的新建、删除、修改等

   1. 协调HRegionServer：

      - 启动时HRegion的分配，以及负载均衡和修复时HRegion的重新分配
      - 监控集群中所有HRegionServer的状态(通过Heartbeat和监听ZooKeeper中的状态)

   2. Admin职能

      - 管理创建、删除、修改Table的操作



## ZooKeeper - 协调者

![](https://gitee.com/sxhDrk/images/raw/master/imgs/HBase与Hadoop架构图2.png)



1. ZooKeeper为HBase集群提供协调服务，它管理着HMaster和HRegionServer的状态(available/alive等)

2. ZooKeeper协调集群所有节点的共享信息，在HMaster和HRegionServer连接到ZooKeeper后创建Ephemeral( 临时）节点，并使用Heartbeat机制维持这个节点的存活状态，如果某个Ephemeral节点实效，则HMaster会收到通知，并做相应的处理

3. 当HMaster宕机的时候实现HMaster之间的failover(失败恢复）

4. 在HRegionServer宕机时通知给HMaster，从而对宕机的HRegionServer中的HRegion集合的修复(将它们分配给其他的HRegionServer)

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/ZooKeeper转移HRegion到正常的HRegionServer.png)

5. 另外，HMaster通过监听ZooKeeper中的Ephemeral节点(默认：/hbase/rs/*)来监控HRegionServer的加入和宕机。

   - 在第一个HMaster连接到ZooKeeper时会创建Ephemeral节点(默认：/hbasae/master)来表示Active的HMaster
   - 其后加进来的HMaster则监听该Ephemeral节点，如果当前Active的HMaster宕机，则该节点消失，因而其他HMaster得到通知，而将自身转换成Active的HMaster，在变为Active的HMaster之前，它会创建在/hbase/back-masters/下创建自己的Ephemeral节点。



## HRegionServer

![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegionServer详解.png)

> 参考图片2：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegionServer详解2.png)



1. HRegionServer一般和DataNode在同一台机器上运行，**实现数据的本地性**

2. HRegionServer存储和管理多个HRegion，由WAL(HLog)、BlockCache、Region组成：

   1. **WAL即Write Ahead Log**

      - 作用：记录写操作，保证数据的可靠性（不丢失）
      - 当HRegionSever接收到写请求之后，会先将这个写请求记录到WAL中，如果记录成功，则将数据更新到对应HRegion中HStore中的memStore中
      - 在HBase0.94版本之前，WAL只允许串行写；从HBase0.94版本开始，引入了NIO的Channel机制，允许对WAL进行并行写
      - 当WAL达到一定条件的时候，会被自动清理
   2. **BlockCache是一个读缓存**
      - 读缓存，维系在内存中，默认是128M
      - 采用了"局部性"原理 - 本质上就是一个"猜"的过程，提高"猜"的命中率：
        - **时间局部性**：在HBase中，如果一条数据被读取过，那么会认为这条数据再次被读取的概率要高于其他没有被读取过的数据，那么会将这条数据放到读缓存中
        - **空间局部性**：在HBase中，如果一条数据被读取过，那么会认为与这条数据相邻的数据被读取的概率要高于其他不相邻的数据，那么会将这条数据相邻的数据放到读缓存中
      - 采用了LRU策略：抛弃最长时间不用的数据
   3. **HRegion是一个Table中的一个Region在一个HRegionServer中的表达**
      1. 一个HRegion包含1到多个HStore，每一个HStore对应一个列族
      2. 一个HStore中包含1个memStore以及0到多个HFile/StoreFile
      3. memStore：写缓存，将数据记录到内存中，默认大小是128M，当memStore达到一定条件的时候，冲刷出一个HFile
      4. HFile：0到多个。HFile落地到HDFS上 -> HBase中的数据最终写到HFile中，HFile存储在DataNode上
      5. memStore的冲刷条件：
         - 当memStore用满之后，会自动冲刷出一个HFil
         - 当WAL的大小达到1G的时候，会冲刷这个HRegionServer上所有的memStore，同时WAL会滚动产生一个新的WAL，原来的WAL就会变成OLD_WAL，OLD_WAL会被清理掉
         - 当所有的memStore所占用的内存之和/总内存>0.35，那么会自动冲刷几个最大的memStore - 这种方案会产生大量的小文件

3. 因为Hbase的HFile是存到HDFS上，所以Hbase实际上是具备数据的副本冗余机制的



## HBase的第一次读写

> 在HBase 0.96以前，HBase有两个特殊的Table：`-ROOT-`和`.meta.`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/-ROOT-.png)

1. `-ROOT-` 表的存储位置存储在ZooKeeper，它存储了`.META.`表的RegionInfo信息
2. `.META.` 表则存储了用户定义的Table的RegionInfo信息
3. 第一次访问Table时
   - 首先从ZooKeeper中读取`-ROOT-`表所在`HRegionServer`；
   - 然后从该HRegionServer中根据请求的TableName，RowKey读取`.META.`表所在HRegionServer；
   - 最后从该HRegionServer中读取`.META.`表的内容而获取此次请求需要访问的HRegion所在的位置，然后访问该HRegionSever获取请求的数据，这需要三次请求才能找到用户Table所在的位置，**然后第四次请求开始获取真正的数据**。
   - 当然为了提升性能，客户端会缓存`-ROOT- Table`位置以及`-ROOT-/.META. Table`的内容

![](https://gitee.com/sxhDrk/images/raw/master/imgs/HBase第一次读取0.96之前.jpg)





> 在HBase 0.96及之后，去掉了`-ROOT- table`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/HRegion宕机重新查询更新缓存.png)

1. 只剩下这个特殊的目录表叫做**Meta Table(**hbase:meta)，它存储了集群中所有用户HRegion的位置信息，而ZooKeeper的节点中(`/hbase/meta-region-server`)存储的则直接是这个`Meta Table`的位置，并且这个`Meta Table`如以前的`-ROOT- Table`一样是不可split的。**这样，客户端在第一次访问用户Table的流程是：**

   1. 从ZooKeeper(`/hbase/meta-region-server`)中获取`hbase:meta`的位置（HRegionServer的位置），缓存该位置信息
   2. 从HRegionServer中查询用户Table对应请求的RowKey所在的HRegionServer，**缓存该位置信息**
   3. 从查询到HRegionServer中读取Row
   
2. 从这个过程中，客户端会缓存这些位置信息，然而第二步它只是缓存当前RowKey对应的HRegion的位置，因而如果下一个要查的RowKey不在同一个HRegion中，则需要继续查询`hbase:meta`所在的HRegion，**然而随着时间的推移，客户端缓存的位置信息越来越多，以至于不需要再次查找`hbase:meta Table`的信息**

3. **当某个HRegion因为宕机或Split被移动，此时需要重新查询并且更新缓存。**

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/HBase第一次读写0.96及之后.png)


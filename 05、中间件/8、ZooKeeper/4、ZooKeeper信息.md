[TOC]

# 节点信息

| 名称           | 示例值                        | 说明                                                         |
| -------------- | ----------------------------- | ------------------------------------------------------------ |
| cZxid          | 0x9                           | 创建该节点所分配的全局事务id                                 |
| ctime          | Wed Mar 04 01:01:09  PST 2020 | 创建时间                                                     |
| mZxid          | 0x9                           | 更新该节点数据所分配的全局事务id                             |
| mtime          | Wed Mar 04 01:01:09  PST 2020 | 更新时间                                                     |
| pZxid          | 0x9                           | 子节点的最新事务id                                           |
| cversion       | 0                             | 子节点版本，表示对子节点的更改次数                           |
| dataversion    | 0                             | 数据版本，数据每发生一次变化，数据版本增加1                  |
| aclVersion     | 0                             | ACL的更新次数，实际上是权限更新的次数                        |
| ephemeralOwner | 0x0                           | 如果不是临时节点，则此值为0；<br />如果是临时节点，则此值是其会话id |
| dataLength     | 13                            | 数据的字节个数                                               |
| numChildren    | 0                             | 子节点的个数                                                 |



## 事务id变化图

![](https://gitee.com/sxhDrk/images/raw/master/imgs/节点的事务id信息.png)



## 节点类型

|          | 顺序节点              | 非顺序节点 |
| -------- | --------------------- | ---------- |
| 持久节点 | Persistent_Sequential | Persistent |
| 临时节点 | Ephemeral_Sequential  | Ephemeral  |



# 事务日志

## 概述

1. 事务日志指Zookeeper系统在正常运行过程中，针对所有的事务操作，在返回客户端ACK的响应前，Zookeeper会保证已经将本次更新操作的事务日志已经写到磁盘上
2. Zookeeper的事务日志为二进制文件，不能通过vim等工具直接访问。其实可以通过zookeeper自带的jar包读取事务日志文件

 

## 查看事务log

1. 进入Zookeeper的安装目录的lib目录下：

   `cd zookeeper-3.4.8/lib`

2. 将lib目录下的slf4j-api-1.6.1.jar复制到数据目录的version-2目录下：

   `cp slf4j-api-1.6.1.jar ../tmp/version-2/`

3. 回到Zookeeper的安装目录：`cd ..`

4. 将Zookeeper的安装目录下的zookeeper-3.4.8.jar复制到数据目录下的version-2目录下：

   `cp zookeeper-3.4.8.jar tmp/version-2/`

5. 进入数据目录下的version-2目录下：

   `cd tmp/version-2/`

6. 执行：

   ` java -cp .:zookeeper-3.4.8.jar:slf4j-api-1.6.1.jar org.apache.zookeeper.server.LogFormatter ./log.100000001`
   查看日志文件



# 快照

## 概述

1. 快照指Zookeeper系统在所存储的部分节点信息写到了磁盘上。
2. Zookeeper的快照文件为二进制文件，不能通过vim等工具直接访问。其实可以通过Zookeeper自带的jar包读取事务日志文件

 

## 查看快照文件

1. 进入Zookeeper的安装目录的lib目录下：

   `cd zookeeper-3.4.8/lib`

2. 将lib目录下的slf4j-api-1.6.1.jar复制到数据目录的version-2目录下：

   `cp slf4j-api-1.6.1.jar ../tmp/version-2/`

3. 回到Zookeeper的安装目录：cd ..

4. 将Zookeeper的安装目录下的zookeeper-3.4.8.jar复制到数据目录下的version-2目录下：

   `cp zookeeper-3.4.8.jar tmp/version-2/`

5. 进入数据目录下的version-2目录下：

   `cd tmp/version-2/`

6. 执行：

    `java -cp .:zookeeper-3.4.8.jar:slf4j-api-1.6.1.jar org.apache.zookeeper.server.SnapshotFormatter ./log.100000001`
   查看快照文件



# session

客户端与Zookeeper服务端连接后，会有一个session。

1. 这个session会同步到其他的follower，防止leader宕机。新的leader选举完成后，客户端和服务端依然会持有一个相同的session，这样客户端创建的临时节点依然会有效
2. session在同步的时候，也会消耗一个事务id。（可以通过kill掉leader，然后到新的leader上创建节点，会发现事务id跳过了一个，跳过的id就是session同步的事务）



# 节点的连接情况

![image-20210322153521829](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210322153521829.png)

Zookeeper采用上图连接方式，让每个节点都可以通信

2888端口：用于client连接

3888端口：用于server(follower)选举leader的通信端口



# Watch

![image-20210329140235274](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210329140235274.png)

Zookeeper作为分布式协调服务，具有watch机制，用来监控服务是否正常。

客户端连接zkServer后，会有session信息和对应节点信息，并对节点有响应的事件监听。当监听的事件发生时，会调用回调函数，对应的Client会接收响应的信息
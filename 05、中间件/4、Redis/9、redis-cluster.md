[TOC]



# 1、redis-cluster结构特点

![](https://gitee.com/sxhDrk/images/raw/master/imgs/redis-cluster结构.png)

- ==集群中所有节点是两两互联的（不区分角色）==。他是redis-cluster结构的基础。底层可以实现通过二进制的方式优化传输速度，形成通信关系。 
- ==哨兵进程消失了，但是哨兵的高可用监听逻辑没有消失==。由于两两互联，哨兵逻辑整合到了`redis-master`主节点中，由集群中所有主节点实现对集群的监听。
  - 最少要求一个集群里有3个master(实现监听最小结构，可以允许一个master宕机)。 
- ==集群的分布式由内部计算逻辑维护的hash  slot实现==。客户端的分布式计算方法也要遵循这个hash槽的逻辑。redis-cli  可以只需要连接一个redis-cluster的节点，就可以将所有数据发送过去，实现内部的重定向跳转 
- ==分布式计算逻辑：hash槽==。引入了`16384`个槽道的概念。
  - 这些操作分片下标号0-16383。
  - 所有的key-value数据，到集群中任意一个节点上，都会计算槽道，key值和槽道是强耦合的hash散列计算（不是hashCode(),这里用到传统计算CRC16()）
  - 任何一个key计算hash散列都会得到==[0,16383]==区间的整数。底层就是hash取余（大范围的正整数%16384）--hash取模计算。
  - master负责维护数据的读写，每一个master都可以设置管理的槽道数量。由节点与槽道的这种松耦合关系形成==【key-->slot-->node】==，所以 ，key和节点对应关系是松耦合，可以进行任意的调试。 



# 2、槽道分布式计算概念

## 2.1、槽道使用逻辑

> 客户端登录其中一个redis服务端，set数据后
>
> - 由登录的该服务端对key进行槽道计算、hash取模计算
> - 计算出【0-16383】之间的某一个值，判断是否有该服务端管理
> - 如果是：执行操作
> - 如果不是：转发给管理该值的服务端进行操作
>   - 转发后执行相同的逻辑

![](https://gitee.com/sxhDrk/images/raw/master/imgs/槽道使用逻辑.png)

## 2.2、数据和节点松耦合

> key值经过hash取模计算得到固定的槽道号。key值不变，槽道号不变
>
> slot与节点管理关系是可以进行迁移，变换的。key经过了slot的计算，可以对应不同的节点

![](https://gitee.com/sxhDrk/images/raw/master/imgs/redis-cluster集群结构.png)



# 3、redis-cluster集群搭建

## 3.1、redis-cluster集群结构

> 三主，各自有一从的redis-cluster，需要六个节点启动。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/数据和节点松耦合.png)

## 3.2、开始搭建

### 准备redis-cluster配置模板

1. 复制redis.conf，重命名为：`redis-cluster.conf`.使用该redis基本模板

   ```c
   [root@10-9-104-184 redis-3.2.11]# cp redis.conf redis-cluster.conf
   ```

2. 使用新的模板文件`redis-cluster.conf`修改集群需要的配置

   - **721行：打开Redis的集群模式**

     > 开启了cluster 就是将redis-server进程中添加槽道的计算逻辑。没有槽道无法实现处理数据。

     ![](https://gitee.com/sxhDrk/images/raw/master/imgs/ruby连接集群节点.png)

   - **729行：打开集群节点配置文件nodes**

     > 集群节点有很多，nodes文件记录了当前集群所有节点状态。

     ![](https://gitee.com/sxhDrk/images/raw/master/imgs/redis-cluster配置-打开集群模式.png)



### 创建集群节点的文件夹

> 和前面一样，复制配置文件，修改端口号，就能启动多个redis-cluster进程。==后面会提供一个脚本文件--重启集群。==
>
> 按照脚本中的配置文件夹结构，创建管理这些配置文件。
>
> - 搭建集群 8000 8001  8002 8003 8004 8005
>
> - 创建一批管理配置文件的文件夹。文件夹以==端口号命名==，就在redis根目录做

1. 创建多个文件夹

   ```c
   [root@10-9-104-184 redis-3.2.11]# mkdir 8000 8001 8002 8003 8004 8005
   ```

2. 将配置拷贝到每一个文件夹

   ```c
   [root@10-9-104-184 redis-3.2.11]# cp redis-cluster.conf 8000
   
   [root@10-9-104-184 redis-3.2.11]# cp redis-cluster.conf 8001
   
   [root@10-9-104-184 redis-3.2.11]# cp redis-cluster.conf 8002
   
   [root@10-9-104-184 redis-3.2.11]# cp redis-cluster.conf 8003
   
   [root@10-9-104-184 redis-3.2.11]# cp redis-cluster.conf 8004
   
   [root@10-9-104-184 redis-3.2.11]# cp redis-cluster.conf 8005
   
   ```



### 修改对应节点配置文件

> 分别打开配置文件，对每一个节点配置进行修改
>
> - 因为全是模板文件拷贝的，把模版文件6379改为该节点对应端口号即可

```sh
[root@10-9-104-184 redis-3.2.11]# vim 8000/redis-cluster.conf
:%s/6379/8000/

[root@10-9-104-184 redis-3.2.11]# vim 8001/redis-cluster.conf
:%s/6379/8001/

[root@10-9-104-184 redis-3.2.11]# vim 8002/redis-cluster.conf
:%s/6379/8002/

[root@10-9-104-184 redis-3.2.11]# vim 8003/redis-cluster.conf
:%s/6379/8003/

[root@10-9-104-184 redis-3.2.11]# vim 8004/redis-cluster.conf
:%s/6379/8004/

[root@10-9-104-184 redis-3.2.11]# vim 8005/redis-cluster.conf
:%s/6379/8005/

```



### 启动所有节点

1. 启动所有集群中的节点进程

   ```sh
   [root@10-9-104-184 redis-3.2.11]# redis-server 8000/redis-cluster.conf 
   
   [root@10-9-104-184 redis-3.2.11]# redis-server 8001/redis-cluster.conf 
   
   [root@10-9-104-184 redis-3.2.11]# redis-server 8002/redis-cluster.conf 
   
   [root@10-9-104-184 redis-3.2.11]# redis-server 8003/redis-cluster.conf 
   
   [root@10-9-104-184 redis-3.2.11]# redis-server 8004/redis-cluster.conf 
   
   [root@10-9-104-184 redis-3.2.11]# redis-server 8005/redis-cluster.conf 
   ```

   

2. ps查看启动进程

   > 确定是否8000-8005全部启动；确定是否全部启动在cluster模式下
   >
   > ==6个进程，末尾以`[cluster]`表示集群模式。==

   ```sh
   [root@10-9-104-184 redis-3.2.11]# ps -ef|grep redis
   
   root     11428     1  0 11:22 ?        00:00:00 redis-server *:8000 [cluster]       
   
   root     12536     1  0 11:22 ?        00:00:00 redis-server *:8001 [cluster]       
   
   root     12540     1  0 11:22 ?        00:00:00 redis-server *:8002 [cluster]       
   
   root     12544     1  0 11:22 ?        00:00:00 redis-server *:8003 [cluster]       
   
   root     12548     1  0 11:22 ?        00:00:00 redis-server *:8004 [cluster]       
   
   root     12554     1  0 11:22 ?        00:00:00 redis-server *:8005 [cluster]
   
   ```

   > 哪个没有启动成功，说明最大的问题，可能就是端口号修改不正确。检查一下对应端口号的配置文件。



### 集群基本命令

1. ==登录==集群客户端，访问redis-cluster服务端节点

   > redis-cli 使用选项多一个`-c`表示以`cluster模式`调用服务端进程，支持cluster一些特性。

   ```sh
   [root@10-9-104-184 redis-3.2.11]# redis-cli -c -p 8000 -h 10.9.104.184
   ```

2. `cluster info`

   > 命令调用会返回集群运行的各种状态信息，其中包含了slot状态

   ```sh
   127.0.0.1:8000> cluster info
   cluster_state:fail #运行状态 fail
   cluster_slots_assigned:0 #槽道分配个数0
   cluster_slots_ok:0
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:1 #集群节点个数
   cluster_size:0 #集群大小（体现在槽道被分成了几份，数据分片个数）
   ```

   

3. `cluster noeds`

   > 查看当前集群，所有节点信息，由于两两互联，在集群搭建完成后，通过任何一个节点，都可以获取集群的所有节点信息。

   当前没有搭建完成，只能看到自己本省节点信息

   ```sh
   10.9.104.184:8000> cluster nodes
   f8bea038f94f5a6dc316a67c2e545bd9693c01c6 :8000 myself,master - 0 0 0 connected
   ```

   

### 连接集群节点

> 启动完了6个节点之后，集群中所有节点还没有添加到一起（让之间两两互联，分配槽道管理范围）。集群是不能使用的。
>
> redis的安装环境中，提供了一个ruby语言编写的脚本文件。专门用来操作集群，执行集群搭建，转移，数据迁移，槽道分片等功能
>
> 这个脚本在redis根目录的src下。叫做`redis-trib.rb`

1. 利用ruby脚本连接集群节点

   > 命令结构：redis-trib.rb 子命令 各种参数
   >
   > - 子命令
   >   - `create host1:port1 ... hostN:portN`
   >   - `create  -- replicas <arg> host1:port1 ... hostN:portN`
   >
   > create作为ruby语言脚本的子命令，就是用来创建连接集群的，可以直接后面跟着所有想加入集群的节点，并且可以利用选项实现搭建初始化过程就完成从节点的分配。

   使用create命令将8000 8001  8002作为三个主节点，创建一个最小结构的集群。

   ```sh
   [root@10-9-104-184 redis-3.2.11]# src/redis-trib.rb create 10.9.104.184:8000 10.9.104.184:8001 10.9.104.184:8002
   Can I set the above configuration? (type 'yes' to accept):yes
   ```

   > 提示后输入：yes，连接成功后后显示下图结果

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/redis-cluster配置-打开集群节点配置文件.png)
   
2. 登录任意节点，测试set/get命令

   ```sh
   [root@10-9-104-184 redis-3.2.11]# redis-cli -c -p 8000
   127.0.0.1:8000> set name wanglaoshi
   -> Redirected to slot [5798] located at 10.9.104.184:8001
   OK
   10.9.104.184:8001>
   ```

   

## 3.3、添加主节点

### 添加

redis-cluster支持动态添加节点，已有的集群结构不用发生任何改变，直接在运行的集群中调用命令就可以做到动态扩展集群（节点变得越来越多，对已有节点不影响）

> ==子命令语法==：
> add-node	new_host:new_port existing_host:existing_port
> 						--slave
> 						--master-id <arg>

调用ruby脚本中的add-node自命令。选项`--slave --master-id`与添加从节点有关。

提示参数：`新节点ip:端口(8003) 已存在节点ip:port(8000|8001|8002)`

> 刚好8003 8004 8005还没有使用。==将8003动态添加到当前集群。==

```sh
[root@10-9-104-184 redis-3.2.11]# src/redis-trib.rb add-node 10.9.104.184:8003 10.9.104.184:8000
```

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ruby添加从节点后的状态.png)



### 添加完后的节点状态

> 可以登录8003调用命令`cluster nodes`命令查看集群节点状态信息。

```sh
127.0.0.1:8003>cluster nodes
```

| `0730f6c324fa3400aeb24e6da23a54368acf110f` | `10.9.104.184:8001` | `master`                                              | `-`                                          | 0 1581920977396              | `2`                                            | `connected`           | 5461-10922   |
| ------------------------------------------ | ------------------- | ----------------------------------------------------- | -------------------------------------------- | ---------------------------- | ---------------------------------------------- | --------------------- | ------------ |
| 节点id ：唯一标识                          | 节点host和端口号    | 节点角色，当前登录人 `slave`,`master`,`master myself` | 表示这个节点的主节点id `-`表示没有上层主节点 | 时间 如果正在登录客户端显示0 | 序号 集群节点会从1开始 `2`,`3`,`4`,`5`分配序号 | 连接状态 disconnected | 槽道管理范围 |

> 默认添加进来的节点都是主节点，由于槽道已经分配完毕。肯定没有新槽道交给他去维护。新的节点即使是主节点，不会存储处理数据。
>
> ==但是作为主节点可以实现投票，已经是集群中可以投票选举的一员了。==



## 3.4、添加从节点

### 添加从节点

> `add-node`调用这个子命令。参数、新节点、已存在节点不变，只要在其中使用2个选项
>
> - `--slave`：表示节点以从节点身份添加到集群
> - ` --master-id`：表示新节点要添加到的主节点的id值。
>
> 就可以。

使用8004作为新节点添加到集群成为8001的从节点。

```sh
[root@10-9-104-184 redis-3.2.11]# src/redis-trib.rb add-node --slave --master-id 0730f6c324fa3400aeb24e6da23a54368acf110f 10.9.104.184:8004 10.9.104.184:8001
```

通过登录查看节点状态。就可以看到不一样的一个nodes信息。

> `cluster nodes`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ruby动态添加主节点.png)



### 方式二：在创建集群时通过命令创建主从关系

可以通过`redis-trib.rb`的脚本命令，在`create`执行时，就指定当前集群主从结构。但是没办法实现一对一的指定。

create 命令下有一个选项  `--replicas <arg> ` 选项表示要将现在创建集群时，提供的所有节点，以主从形式创建完成，`<arg>`对应一个整数值，例如：1，表示集群在保证完整搭建正常运行结构下（最小三个主），会为每一个主节点都分配至少1个从节点。所以，提供的节点个数满足这个计算。例如：

- `--replicas 1 `： 至少给6个节点，实现3主各有1从，
- `--replicas 2`：至少给9个节点，实现3主各有2从。



### 高可用测试

> 验证当前看不到哨兵进程时，是否有故障转移。
>
> 将主节点8001宕机，等待一段时间进行选举。实现主从替换。8001恢复回来就变成了从节点。

1. 将8001宕机

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/高可用测试-旧主节点变从节点.png)

2. 选举完成后，新的主节点出现，8004顶替为新的主节点，数据和槽道一并交给8000维护

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/迁移槽道数量.png)

3. 将8001恢复上线，该节点就会变成从节点挂接在8004下

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/ruby删除节点.png)



## 3.5、删除节点

> `del-node`。可以实现对集群中某个节点的删除工作。
>
> 只能在集群中删除没有槽道管理权的主节点和从节点。==特点：不处理槽道逻辑==，因为槽道和数据是绑定的，一旦删除了有槽道管理范围的节点，集群有一批槽道不可用，集群down状态。
>
> 命令结构：`del-noed host:port node_id`
>
> - `host:port`：表示一个已存在的节点
> - `node_id`：表示删除的节点的id

目前8001为从节点，8003为没有槽道的主节点，这两个节点可以删除

```sh
[root@10-9-104-184 redis-3.2.11]# src/redis-trib.rb del-node 10.9.104.184:8003 0730f6c324fa3400aeb24e6da23a54368acf110f
```

![](https://gitee.com/sxhDrk/images/raw/master/imgs/高可用测试-将主节点宕机.png)



## 3.6、重新分配槽道

> 已经创建集群，由脚本语言命令ruby将槽道分配完毕了，所以当有新的主节点加入时，是没有槽道的，一旦想对槽道进行重新分片，可以调用子命令，控制操作空槽道（没有key对应的槽道）进行迁移操作。`reshard` 重新分片
>
> 命令结构：`reshard host:port --from <arg> --slots <arg> --yes  --timeout <arg>`
>
> - `host:port `: 对哪个==集群==实现重新分配操作的操作

1. 调用重新分片命令

   > 8003作为主节点没有槽道管理，新定义一批槽道: 200个（确定没有绑定任何key值）

   ```sh
   [root@10-9-104-184 redis-3.2.11]# src/redis-trib.rb reshard 10.9.104.194:8004
   ```

   

2. 提示：`How many slots do you wang to move (from 1 to 16384)?`

   > 要从1到16384个槽道数量迁移几个槽道？ ==200个==

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/高可用测试-新主节点顶替.png)

3. 提示：`What is the receiving node ID?`

   > 迁移到哪个节点ID？ ==迁移到8003中==

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/迁移槽道目标ID.png)

4. 确定槽道来源

   > 当前集群中已有的16384个槽道，全部都已经分配完毕，在迁移的时候一定从一个源节点迁移到目标节点（8003）。源节点可以是1个，也可以是多个，甚至是全部其他拥有槽道的节点。
   >
   > - ==具体一个==：输入源节点ID
   >
   >   将节点id依次填写到#1 #2序号后 
   >
   >   ```sh
   >   Source node #1:d3ec4c56a741b1f60833f685e53cccb4cee5b7de
   >   Source node #2:done
   >   ```
   >
   > - ==多个==：分别出入源节点ID，没输入完回车
   >
   > - ==全部==：输入`all`
   >
   > 结束该步骤：输入`done`

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/迁移槽道源节点.png)

5. 提示：确定迁移槽道？yes

   `Do  you want to proceed with the proposed reshard plan (yes/no)?`

   同意yes ；不同意no

6. 登录节点，观察集群槽道变化

   > 8003节点多出200个槽道



# 4、脚本文件的使用

## 4.1、编写脚本

> shell脚本包含了所有重新搭建集群的过程：==停删启建==，只要保证环境配置文件能够正常启动一个8000-8005的节点环境就可以使用脚本（注意文件夹结构）
>
> ==该脚本存放在Redis根目录，名称为：`recreate.sh`==

```sh
#stop redis
for port in {8000..8005}
do redis-cli -c -p $port -h 10.42.0.33 shutdown
done;

#delete file dump nodes
rm -f dump*
rm -f nodes*

#start redis
for port in {8000..8005}
do redis-server $port/redis-cluster.conf
done;

#create cluster
echo yes|src/redis-trib.rb create --replicas 1 10.42.0.33:8000 10.42.0.33:8001 10.42.0.33:8002 10.42.0.33:8003 10.42.0.33:8004 10.42.0.33:8005

```



## 4.2、运行

> 运行脚本，就会自动创建一个3主，各1从的集群

```sh
[root@10-9-104-184 redis-3.2.11]# sh recreate.sh
```



# 5、常见问题

## 5.1、提示添加节点失败

### 问题提示

> 10.42.47.192:8000  is not empty. Either the  node already knows other nodes (check with CLUSTER NODES) or contains some key  in database 0.

创建集群时，向集群添加节点，发现节点非空。要不然当前集群已经知道该节点，要不然就是该节点中的redis内存中有已存在的数据



### 原因

1. 节点非空，可能是读取了其他文件的dump持久化文件，导致新节点启动时就存在了内存数据 
2. 集群已知，该节点已经加入过这个集群。



### 解决思路

1. 登录到该节点，执行`keys * `检查节点中是否有数据 
   - 如果没有数据，说明是第二个原因。
   - 如果有数据，就将数据清空操作。调用命令`flushall`。冲刷掉所有数据，内存数据和持久化文件数据一并都消失了。
2. 不能在已经创建的集群节点上再去调用`redis-trib.rb  create`的命令。 删除集群对该节点的记录，就将对应节点的nodes文件删除
   - 关闭对应进程，删除对应的`nodes-port.conf`文件。重新启动，重新执行命令。



## 5.2、集群操作各种意外解决

> 操作集群的各种步骤，各种检测功能，如果由于误操作，网络意外导致集群不可恢复，在测试环境中，可以执行以下几步，重新恢复集群的状态。==停-删-启-建==

1. 停

   > 停止进程，将集群所有进程停掉
   >
   > redis-cli  -c -p 8000 shutdown
   >
   > 8001  …

2. 删

   > 删除掉当前集群中所有节点对应持久化文件
   >
   > - `dump*` 每个节点对应持久化文件。不能保证这些文件中，在重新创建集群之前是空，就把他删掉
   >
   > - `nodes*` 对应节点集群状态持久化文件。每个文件肯定都保存了已经创建好的集群节点信息。

   ```sh
   #rm -f dump*
   #rm -f nodes* 
   ```

   

3. 启

   > 启动所有集群节点。
   >
   > 当停和删做完了，当前准备所有节点环境就和最初创建的环境是一致的，可以启动所有节点等待重新搭建

   `#redis-server  8000/conf`

   `#redis-server  8001/conf`

4. 建

   > 重新对已经启动的多个节点按照想要的结构创建出集群。
   >
   > `src/redis-trib.rb  create node1 node2 node3 node4 node5 node6..`



# 6、Redis 5.0版本

1. 先启动所有的redis服务

2. 需要在每一个节点的配置文件中配置`masterauth`和`requirepass`，而且密码要一致

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210301211230853.png)![](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210301211243181.png)

3. 建立集群

   ```shell
   redis-cli --cluster create 182.92.95.61:8000 182.92.95.61:8001 182.92.95.61:8002 182.92.95.61:8003 182.92.95.61:8004 182.92.95.61:8005 --cluster-replicas 1 -a sxh123+123
   ```

4. 如果是外网redis，需要将防火墙的端口开放

   > 需要注意：redis集群不仅需要开通redis客户端连接的端口，而且需要开通集群总线端口。
   >
   > 集群总线端口为redis客户端连接的端口 + 10000
   >
   > 如redis端口为6379。 则集群总线端口为16379

   ![image-20210301211536008](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210301211536008.png)

5. 如果是云服务器，需要将安全组开放

   ![image-20210301211436048](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210301211436048.png)

6. 重新创建集群

   ![image-20210301211612637](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210301211612637.png)

## 重新构建脚本

```shell
# stop redis
for port in {8000..8005}
do redis-cli -c -p $port -a 'sxh123+123' -h 182.92.95.61 shutdown
done;

# copy dump
vdate=`date +%Y%m%d`
echo $vdate
mkdir -p copydump/$vdate/
cp dump* copydump/$vdate/

#delete file dump nodes
rm -f dump*
rm -f nodes*

#start redis
for port in {8000..8005}
do redis-server $port/redis_cluster.conf
done;

# create cluster
echo yes | redis-cli --cluster create 182.92.95.61:8000 182.92.95.61:8001 182.92.95.61:8002 182.92.95.61:8003 182.92.95.61:8004 182.92.95.61:8005 --cluster-replicas 1 -a 'sxh123+123'

```

## 重启脚本

```shell
# stop redis
for port in {8000..8005}
do redis-server $port/redis_cluster.conf
done;

```



## 停止脚本

```shell
# stop redis
for port in {8000..8005}
do redis-cli -c -p $port -a 'sxh123+123' shutdown save
done;
```



## 重启恢复数据

1. 关闭每个节点中的aof模式

   ![image-20210301214159128](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210301214159128.png)

2. 重启集群后，每个节点进行check，恢复集群数据

   ![image-20210301214237157](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210301214237157.png)
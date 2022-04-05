# Hadoop简介

## 概述

> Hadoop：开源分布式存储、计算矿框架。免费试用
>
> CHD：Hadoop的一个发行版本，可以一键式部署集群，但是是收费的，非常昂贵

1. Hadoop是Yahoo 开发，后来贡献给了Apache的一套开源的、可靠的、可扩展的（可伸缩的）用于进行分布式计算的框架

2. Hadoop之父：Doug Cutting（道格·卡丁）

3. Hadoop的版本管理非常混乱（最新更新版本2.10，版本最新确实3.2）

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/Hadoop完全分布式-3个节点.png)

## 模块

1. Hadoop Common：基本模块，支撑其他模块运行
2. Hadoop Distributed File System (HDFS™)：分布式存储
3. Hadoop YARN：任务调度和集群资源管理
4. Hadoop MapReduce：分布式计算
5. Hadoop Ozone：对象存储

## 版本

1. Hadoop 1.0：包含Common、HDFS、MapReduce
2. Hadoop 2.0：包含Common、HDFS、MapReduce和YARN
   - 从Hadoop 2.7开始包含了Ozone
   - 从Hadoop 2.9开始包含了Submarine
   - ==Hadoop 1.0和Hadoop 2.0不兼容==
3. Hadoop 3.0：包含Common、HDFS、MapReduce和YARN
   - 从Hadoop 3.1开始包含了Ozone
   - 从Hadoop 3.1最新版本中包含了Submarine



## 相关组件

1. HBase: 类似Google BigTable的分布式NoSQL列数据库。（HBase和Avro已经于2010年5月成为顶级     Apache 项目）
2. Hive：数据仓库工具，由Facebook贡献
3. Zookeeper：分布式锁设施，提供类似Google  Chubby的功能，由Facebook贡献
4. Avro：新的数据序列化格式与传输工具，将逐步取代Hadoop原有的IPC机制。
5. Pig:     大数据分析平台，为用户提供多种接口
6. Ambari：Hadoop管理工具，可以快捷的监控、部署、管理集群
7. Sqoop：用在Hadoop与传统的数据库间进行数据的传递





# 安装

> 开启Hadoop：start-all.sh
>
> 关闭Hadoop：stop-all.sh

## 单机模式

只能启动MapReduce，所以不使用单机模式

## 伪分布式

### 环境配置

> 能启动HDFS、MapReduce、Yran的大部分功能

1. 关闭防火墙

   ```sh
   service iptables stop
   chkconfig iptables off
   ```

2. 修改云主机的主机名，Hadoop集群中要求主机名中不能出现`-`或者`_`

   ```sh
   cd /etc/sysconfig
   vim network
   ```

   修改

   ```
   HOSTNAME=hadoop01
   ```

   保存退出，重新生效

   ```sh
   source network
   ```

3. 需要将IP和主机名进行映射

   ```sh
   cd /etc
   vim hosts
   ```

   添加host映射

   ```sh
   10.9.162.133 hadoop01
   ```

4. 重启

   ```sh
   reboot
   ```

5. 配置免密登录

   产生密钥：`ssh-keygen`

   拷贝密钥：`ssh-copy-id root@hadoop01`

   测试是否能够免密登录：`ssh hadoop01`

   如果成功，则退出：`logout`

6. 下载Hadoop的安装包

   ```sh
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/hadoop-alone.tar.gz
   ```

7. 解压安装包

   ```sh
   tar -xvf hadoop-alone.tar.gz
   ```

8. 配置环境变量

   ```sh
   vim /etc/profile
   ```

   文件末尾添加

   ```sh
   export HADOOP_HOME=/home/software/hadoop-2.7.1
   export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
   ```

   保存退出，重新生效

   ```sh
   source /etc/profile
   ```

9. 格式化`namenode`

   ```sh
   hadoop namenode -format
   ```

   如果出现

   `Storage directory /home/software/hadoop-2.7.1/tmp/dfs/name has been successfully formatted.`

   表示格式化成功

10. 启动hadoop

    ```sh
    start-all.sh
    ```

    通过jps查看，有6个进程

    ```
    Jps
    NameNode
    DataNode
    SecondaryNameNode
    ResourceManager
    NodeManager
    ```

11. 通过网页浏览

    可以通过浏览器访问HDFS的页面，访问地址为：IP地址:50070

    可以通过浏览器访问Yarn的页面，访问地址为：IP地址:8088



### 配置文件的修改

> 进入：`cd hadoop-2.7.1/etc/hadoop`

1. 配置`hadoop-env.sh`

   - 编辑hadoop-env.sh:`vim hadoop-env.sh`

   - 修改JAVA_HOME的路径：

     `export JAVA_HOME=/home/software/jdk1.8`

   - 修改HADOOP_CONF_DIR的路径

     `export HADOOP_CONF_DIR=/home/software/hadoop-2.7.1/etc/hadoop`

   - 保存退出

   - 重新生效：`source hadoop-env.sh`

2. 配置`core-site.xml`

   - 编辑core-site.xml：`vim core-site.xml`

   - 指定如下内容

     ```xml
     <property>
         <!-- 指定HDFS中的主节点 - namenode -->
         <name>fs.defaultFS</name>               
         <value>hdfs://hadoop01:9000</value>
     </property>
     <property>
         <!-- 执行Hadoop运行时的数据存放目录 -->
         <name>hadoop.tmp.dir</name>
         <value>/home/software/hadoop-2.7.1/tmp</value>
     </property>
     
     ```

   - 保存退出

3. 配置`hdfs-site.xml`

   - 编辑文件：`vim hdfs-site.xml`

   - 添加如下内容

     ```xml
     <property>
         <!-- 设置HDFS中的副本数量 -->
         <!-- 在伪分布式下，值设置为1 -->
         <name>dfs.replication</name>
         <value>1</value>
     </property>
     ```

   - 保存退出

4. 配置`mapred-site.xml`

   - 将`mapred-site.xml.template`复制为`mapred-site.xml`

     ```sh
     cp mapred-site.xml.template mapred-site.xml
     ```

   - 编辑新文件：`vim mapred-site.xml`

   - 添加如下配置

     ```xml
     <property>
         <!-- 指定将MapReduce在Yarn上运行  -->
         <name>mapreduce.framework.name</name>
         <value>yarn</value>
     </property>
     ```

   - 保存退出

5. 配置`yarn-site.xml`

   - 编辑yarn-site.xml：`vim yarn-site.xml`

   - 添加如下内容：

     ```xml
     <!-- 指定Yarn的主节点 - resourcemanager -->
     <property>
         <name>yarn.resourcemanager.hostname</name>
         <value>hadoop01</value>
     </property>
     <!-- NodeManager的数据获取方式 -->
     <property>
         <name>yarn.nodemanager.aux-services</name>
         <value>mapreduce_shuffle</value>
     </property>
     ```

   - 保存退出

6. 配置`slaves`

   - 编辑slaves：`vim slaves`
   - 添加从节点信息，例如：hadoop01
   - 保存退出



## 完全分布式

> 能启动Hadoop的所有功能



![](https://gitee.com/sxhDrk/images/raw/master/imgs/完全分布式结构.png)



> 如上图所示，完全分布式的Hadoop的结构，最少需要13个节点，才能保证高可用，也就是最少需要13台服务器

只有三台服务器，在三台服务器上搭建Hadoop完全分布式集群

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Hadoop版本混乱.png)



### 环境配置

> 因为第一台 服务器搭建过伪分布式，所以需要将伪分布式Hadoop文件重命名

```sh
cd /home/software/
mv hadoop-2.7.1/ hadoop-alone
```



1. 三台云主机关闭防火墙

   ```sh
   service iptables stop
   chkconfig iptables off
   ```

2. 修改三台云主机的主机名

   ```sh
   vim /etc/sysconfig/network
   ```

   修改HOSTNAME属性，依次改为hadoop01、hadoop02、hadoop03

   保存退出，并且重新生效

   ```sh
   source /etc/sysconfig/network
   ```

3. 三台云主机添加IP映射

   > 编辑hosts文件`vim /etc/hosts`，添加如下内容

   ```sh
   10.9.162.133 hadoop01
   10.9.152.65 hadoop02
   10.9.130.83 hadoop03
   ```

4. 三台云主机重启：`reboot`

5. 三台云主机开启Zookeeper

   ```sh
   cd /home/software/zookeeper-3.4.8/bin 
   sh zkServer.sh start	#开启Zookeeper
   sh zkServer.sh status	#检查Zookeeper的状态，应该为2个follower1个leader
   ```

6. 返回第一个节点software目录下：`cd /home/software`

7. 下载Hadoop完全分布式的安装包

   ```sh
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/hadoop-dist.tar.gz
   ```

8. 三台机器进行免密登录

   ```sh
   ssh-keygen
   ssh-copy-id	root@hadoop01
   ssh-copy-id	root@hadoop02
   ssh-copy-id	root@hadoop03
   ```

   三台集器测试：

   ```sh
   ssh hadoop01 #如果不需要密码logout
   ssh hadoop02 #如果不需要密码logout
   ssh hadoop03 #如果不需要密码logout
   ```

9. 解压安装包

   ```sh
   tar -xvf hadoop-dist.tar.gz
   ```

10. 将Hadoop的安装包拷贝到其他两个节点上

    ```sh
    scp -r hadoop-2.7.1/ root@hadoop02:/home/software/
    scp -r hadoop-2.7.1/ root@hadoop03:/home/software/
    ```

11. 三台节点配置环境变量

    > 修改文件：`vim /etc/profile`

    ```sh
    export HADOOP_HOME=/home/software/hadoop-2.7.1
    export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
    ```

    保存退出，并且重新生效：`source /etc/profile`

12. 在任意一个节点格式化Zookeeper

    > 实际上就是在Zookeeper中注册Hadoop节点

    ```sh
    hdfs zkfc -formatZK
    ```

    如果成功，日志会显示：`Successfully created /hadoop-ha/ns in ZK`

    如果出现：`HA is not available`，说明系统兼容性不够，重装系统解决

13. 三台云主机启动`JournalNode`

    ```sh
    hadoop-daemon.sh start journalnode
    ```

14. 在第一个节点上格式化`NameNode`

    ```sh
    hadoop namenode -format
    ```

    如果格式化成功，则出现：`Storage directory /home/software/hadoop-2.7.1/tmp/hdfs/name has been successfully formatted. `

15. 在第一个节点上启动NameNode

    ```sh
    hadoop-daemon.sh start namenode
    ```

16. 在第二个节点上格式化NameNode

    ```sh
    hdfs namenode -bootstrapStandby
    ```

    如果格式化成功，则出现：`Storage directory /home/software/hadoop-2.7.1/tmp/hdfs/name has been successfully formatted.`

17. 在第二个节点启动NameNode

    ```sh
    hadoop-daemon.sh start namenode
    ```

18. 在==三个节点上都启动==DataNode

    ```sh
    hadoop-daemon.sh start datanode
    ```

19. 在==第三个节点==上启动YARN

    ```sh
    start-yarn.sh
    ```

20. 在第一个节点上启动ResourceManager

    ```sh
    yarn-daemon.sh start resourcemanager
    ```

21. 在第一个和第二个节点上启动zkfc

    ```sh
    hadoop-daemon.sh start zkfc
    ```



> 如果都启动成功，通过jps查看：
>
> - 第一个节点出现8个进程
>
>   ```
>   - Jps
>   - NameNode
>   - DataNode
>   - JournalNode
>   - ResourceManager
>   - NodeManager
>   - DFSZKFailoverController
>   - QuorumPeerMain
>   ```
>
> - 第二个节点出现7个进程
>
>   ```
>   - Jps
>   - NameNode
>   - DataNode
>   - JournalNode
>   - NodeManager
>   - DFSZKFailoverController
>   - QuorumPeerMain
>   ```
>
> - 第三个节点出现6个进程
>
>   ```
>   - Jps
>   - DataNode
>   - JournalNode
>   - ResourceManager
>   - NodeManager
>   - QuorumPeerMain
>   ```
>
> 如果少了QuorumPeerMain，表示Zookeeper启动出错
>
> 如果少了NameNode/DataNode/JournalNode/DFSZKFailoverController
>
> - 可以通过`hadoop-daemon.sh start namenode/datanode/journalnode/zkfc`
>
> 如果少了ResourceManager/NodeManager
>
> - 可以通过`yarn-daemon.sh start resourcemanager/nodemanager`
>
> ==Hadoop完全分布式开启，需要先启动ZooKeeper，再开Hadoop==



### 配置文件的修改

> 上述下载的Hadoop的jar包中已经配置好了

1. 编辑core-site.xml，添加如下内容：

   ```xml
   <!--指定hdfs的nameservice，为整个集群起一个别名-->
   <property>
       <name>fs.defaultFS</name>                
       <value>hdfs://ns</value>
   </property>
   <!--指定Hadoop数据临时存放目录-->
   <property>
       <name>hadoop.tmp.dir</name>
       <value>/home/software/hadoop-2.7.1/tmp</value>
   </property> 
   <!--指定zookeeper的存放地址-->
   <property>
       <name>ha.zookeeper.quorum</name>
       <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
   </property> 
   ```

2. 编辑hdfs-site.xml

   ```xml
   <!--执行hdfs的nameservice为ns，注意要和core-site.xml中的名称保持一致-->
   <property>
       <name>dfs.nameservices</name>
       <value>ns</value>
   </property>
   <!--ns集群下有两个namenode，分别为nn1, nn2-->
   <property>
       <name>dfs.ha.namenodes.ns</name>
       <value>nn1,nn2</value>
   </property>
   <!--nn1的RPC通信-->
   <property>
       <name>dfs.namenode.rpc-address.ns.nn1</name>
       <value>hadoop01:9000</value>
   </property>
   <!--nn1的http通信-->
   <property>
       <name>dfs.namenode.http-address.ns.nn1</name>
       <value>hadoop01:50070</value>
   </property>
   <!-- nn2的RPC通信地址 -->
   <property>
       <name>dfs.namenode.rpc-address.ns.nn2</name>
       <value>hadoop02:9000</value>
   </property>
   <!-- nn2的http通信地址 -->
   <property>
       <name>dfs.namenode.http-address.ns.nn2</name>
       <value>hadoop02:50070</value>
   </property>
   <!--指定namenode的元数据在JournalNode上存放的位置，这样，namenode2可以从journalnode集群里的指定位置上获取信息，达到热备效果-->
   <property>
       <name>dfs.namenode.shared.edits.dir</name>
       <value>qjournal://hadoop01:8485;hadoop02:8485;hadoop03:8485/ns</value>
   </property>
   <!-- 指定JournalNode在本地磁盘存放数据的位置 -->
   <property>
       <name>dfs.journalnode.edits.dir</name>
       <value>/home/software/hadoop-2.7.1/tmp/journal</value>
   </property>
   <!-- 开启NameNode故障时自动切换 -->
   <property>
       <name>dfs.ha.automatic-failover.enabled</name>
       <value>true</value>
   </property>
   <!-- 配置失败自动切换实现方式 -->
   <property>
       <name>dfs.client.failover.proxy.provider.ns</name>
       <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
   </property>
   <!-- 配置隔离机制 -->
   <property>
       <name>dfs.ha.fencing.methods</name>
       <value>sshfence</value>
   </property>
   <!-- 使用隔离机制时需要ssh免登陆 -->
   <property>
       <name>dfs.ha.fencing.ssh.private-key-files</name>
       <value>/root/.ssh/id_rsa</value>
   </property>
   <!--配置namenode存放元数据的目录，可以不配置，如果不配置则默认放到hadoop.tmp.dir下-->
   <property>    
       <name>dfs.namenode.name.dir</name>    
       <value>file:///home/software/hadoop-2.7.1/tmp/hdfs/name</value>    
   </property>    
   <!--配置datanode存放元数据的目录，可以不配置，如果不配置则默认放到hadoop.tmp.dir下-->
   <property>    
       <name>dfs.datanode.data.dir</name>    
       <value>file:///home/software/hadoop-2.7.1/tmp/hdfs/data</value>    
   </property>
   <!--配置副本数量-->    
   <property>    
       <name>dfs.replication</name>    
       <value>3</value>    
   </property>   
   <!--设置用户的操作权限，false表示关闭权限验证，任何用户都可以操作-->                                                                    
   <property>    
       <name>dfs.permissions</name>    
       <value>false</value>    
   </property>  
   ```

3. 编辑mapred-site.xml

   ```xml
   <property>    
       <name>mapreduce.framework.name</name>    
       <value>yarn</value>    
   </property> 
   ```

4. 编辑yarn-site.xml

   ```xml
   <!--配置yarn的高可用-->
   <property>
       <name>yarn.resourcemanager.ha.enabled</name>
       <value>true</value>
   </property>
   <!--指定两个resourcemaneger的名称-->
   <property>
       <name>yarn.resourcemanager.ha.rm-ids</name>
       <value>rm1,rm2</value>
   </property>
   <!--配置rm1的主机-->
   <property>
       <name>yarn.resourcemanager.hostname.rm1</name>
       <value>hadoop01</value>
   </property>
   <!--配置rm2的主机-->
   <property>
       <name>yarn.resourcemanager.hostname.rm2</name>
       <value>hadoop03</value>
   </property>
   <!--开启yarn恢复机制-->
   <property>
       <name>yarn.resourcemanager.recovery.enabled</name>
       <value>true</value>
   </property>
   <!--执行rm恢复机制实现类-->
   <property>
       <name>yarn.resourcemanager.store.class</name>
       <value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
   </property>
   <!--配置zookeeper的地址-->
   <property>
       <name>yarn.resourcemanager.zk-address</name>
       <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
   </property>
   <!--执行yarn集群的别名-->
   <property>
       <name>yarn.resourcemanager.cluster-id</name>
       <value>ns-yarn</value>
   </property>
   <!-- 指定nodemanager启动时加载server的方式为shuffle server -->
   <property>    
       <name>yarn.nodemanager.aux-services</name>    
       <value>mapreduce_shuffle</value>    
   </property>  
   <!-- 指定resourcemanager地址 -->
   <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>hadoop03</value>
   </property>
   ```

   





# Docker安装




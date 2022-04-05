[TOC]

# 单机安装

> 不依赖于Hadoop的HDFS，配置完即可使用
>
> 好处是便于测试；坏处是不具备分布式存储数据的能力

1. 安装JDK

2. 上传或者下载HBASE的安装包

3. 解压安装包：tar -xvf hbase-0.98.17-hadoop2-bin.tar.gz

4. 进入HBASE安装目录下的子目录conf：cd hbase-0.98.17-hadoop2/conf

5. 修改配置文件hbase-site.xml：vim hbase-site.xml

6. 添加如下配置：

   ```xaml
   <!--指定HBASE的数据存储目录-->
   <property>
       <name>hbase.rootdir</name>
       <value>file:///home/software/hbase/tmp</value>
   </property>
   ```

7. 进入HBASE的安装目录的子目录bin下：cd ../bin

8. 启动服务器端，执行sh start-hbase.sh

   > 启动完成之后可以通过jps命令查看是否有HMaster进程

9. 启动客户端 ./hbase shell





# 伪分布式安装

1. 安装JDK

2. 安装Hadoop的伪分布式或者完全分布式集群

3. 上传或者下载HBASE的安装包

4. 解压安装包：`tar -xvf hbase-0.98.17-hadoop2-bin.tar.gz`

5. 进入HBASE安装目录下的子目录conf：`cd hbase-0.98.17-hadoop2/conf`

6. 修改conf/hbase-env.sh：`vim hbase-env.sh`

7. 添加JAVA_HOME：`export JAVA_HOME=JDK的实际安装路径`

8. 重新生效：`source hbase-env.sh`

9. 修改配置文件hbase-site.xml：`vim hbase-site.xml`

10. 配置使用hdfs:
	
	```xml
	<property>
    <name>hbase.rootdir</name>
	    <value>hdfs://hadoop01:9000/hbase</value>
	</property>
	<property>
	    <name>dfs.replication</name>
	    <value>1</value>
	</property>
	```
	
11. 启动Hadoop。如果是使用的Hadoop完全分布式集群，则还需要启动Zookeeper

12. 进入HBASE的安装目录的子目录bin下：`cd ../bin`

13. 启动服务器端，执行`sh start-hbase.sh`
	启动完成之后可以通过jps命令查看是否有HMaster进程

15. 启动客户端 `./hbase shell`







# ==完全分布式安装==

1. 安装和配置：Hadoop+JDK+ZooKeeper

2. 三台云主机开启ZooKeeper

   ```sh
   cd /home/software/zookeeper-3.4.8/bin
   sh zkServer.sh start	#启动Zookeeper
   sh zkServer.sh status	#查看Zookeeper状态
   ```

   > 开启成功后，3个节点中，有2个follower和1和leader

3. 在第一个节点启动Hadoop

   ```sh
   start-all.sh
   ```

4. 在第一个节点下载HBase安装包并解压

   ```sh
   cd /home/software/
   #下载
   wget http://bj-yzjd.ufile.cn-north-02.ucloud.cn/hbase-1.3.1-bin.tar.gz
   #解压
   tar -xvf hbase-1.3.1-bin.tar.gz
   ```

5. 修改HBase目录下的`/conf/hbase-env.sh`

   ```sh
   #修改JAVA_HOME：
   export JAVA_HOME=xxxx
   #修改Zookeeper和Hbase的协调模式，hbase默认使用自带的zookeeper，如果需要使用外部zookeeper，需要先关闭：
   export HBASE_MANAGES_ZK=false
   ```

   将`export HBASE_MASTER_OPTS`和`export HBASE_REGIONSERVER_OPTS`注释掉

   保存，重新生效：`source hbase-env.sh`

6. 修改HBase目录下的`/conf/hbase-site.xml`

   > 在`<configuration>`标签中添加如下内容

   ```xml
   <property>
       <name>hbase.rootdir</name>
       <value>hdfs://hadoop01:9000/hbase</value>
   </property>
   <property>
       <name>hbase.cluster.distributed</name>
       <value>true</value>
   </property>
   <property>
       <name>hbase.zookeeper.quorum</name>
       <value>hadoop01:2181,hadoop02:2181,hadoop03:2181</value>
   </property>
   ```

7. 编辑`/conf/regionservers`

   > 添加云主机的名字

   ```
   hadoop01
   hadoop02
   hadoop03
   ```

8. 远程拷贝到`Hadoop02`,`Hadoop03`主机上

   ```sh
   cd /home/software
   
   scp -r hbase-1.3.1 root@hadoop02:/home/software/
   scp -r hbase-1.3.1 root@hadoop03:/home/software/
   ```

9. 进入HBase的bin目录，启动HBase

   ```sh
   cd /home/software/hbase-1.3.1/bin
   
   sh start-hbase.sh
   ```

   > 启动后，该节点会出现HMaster、HRegionServer两个进程
   >
   > 其他两个节点都有HRegionServer进程

10. 在HBase的bin目录，启动HBase的客户端

    ```sh
    sh hbase shell
    ```



> 1. 可以通过浏览器访问：http://xxxx:60010访问web界面，管理HBase
> 2. bin目录执行：`stop-hbase.sh`关闭HMaster
> 3. bin目录执行：`sh hbase-daemon.sh stop regionserver`关闭RegionServer
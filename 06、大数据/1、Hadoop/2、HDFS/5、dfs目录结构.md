# 概述
1. dfs目录表示HDFS的存储目录
	- `dfs/name`表示NameNode的持久化目录
    - `dfs/data`表示DataNode的存储目录
    - `dfs/namesecondary`表示SecondaryNameNode的存储目录

2. 实际过程中，由于NameNode、DataNode以及SecondaryNameNode应该分布在不同的节点上，所以name、data、nameseconda三个目录也应该出现在不同的节点上

3. 当执行格式化指令时，会在指定的数据目录下，生成`dfs/name`目录

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/NameNode格式化生成dfs目录.png)

4. 当格式化后，启动HDFS前，会生成一个最初的`fsimage_0000000000000000000`文件，该文件中存储的根节点的信息

5. `dfs/name/in_use.lock`文件的作用是防止在同一台服务器上启动多个NameNode，避免管理紊乱

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/is_use.lock文件.png)

6. 当启动HDFS时，会生成edits文件

7. HDFS中有事务id的概念，当HDFS每接收一个写操作（比如：mkdir put mv），都会分配全局递增的事务id(txid)，然后写到edits文件中

8. 每生成一个新的edits文件

   - edits文件中都会以`OP_START_LOG_SEGMENT`开头
   - 当一个edits文件写完后，会以`OP_END_LOG_SEGMENT`结尾。

   > 即在`OP_START_LOG_SEGMENT`——`OP_END_LOG_SEGMENT`存储的是这个edits文件所有的事务记录

9. `edits_inprogress`文件的作用是记录当前正在执行的事务文件。后面的编号是以上一次`Txid+1`来命名的

10. 初次使用HDFS时，有一个默认的`edits`和`fsimage`的合并周期（==1分钟==），以后在使用HDFS的过程中，达到条件`edits_inprogress会和fsimage`进行合并

    > 条件是自定义的，比如3600s合并一次

11. 上传文件底层会拆分成如下的事务过程：
    - `OP_ADD` 将文件加入到指定的HDFS目录下，并以._Copyging_结尾，表示此文件还未写完
    - `ALLOCATE_BLOCK_ID`：为文件分配块ID
    - `SET_GENSTAMP_V2`：为块生成时间戳版本号，是全局唯一的
    - `ADD_BLOCK`：写块数据
    - `OP_CLOSE`：表示块数据写完
    - `OP_RENAME_OLD`：将文件重命名，表示写完

12. 当停止HDFS后再次启动HDFS时，当执行一个事务之后，底层会触发一次合并，然后生成一个新的edits文件

13. `seen_txid`记录是的最新的`edits_inprogress`文件末尾的数字

14. `VERSION`文件中的重要内容：

    - clusterID：集群编号
    - storageType：节点类型
    - blockpollID：块池编号

15. `fsimage_N`文件存储的N号事务前的所有的元数据信息

16. `fsimage_N.md5`存储的是fsimage文件的md5校验码，可以通过MD5的校验和来校验文件的完整性：`md5sum fsimage_n`



# edits文件和fsimage文件

1. 查看edits文件：`hdfs oev -i edits文件 -o xxx.xml`。例如：
	- `hdfs oev -i edits_0000000000000000001-0000000000000000003 -o edits.xml`
2. 查看fsimage文件：`hdfs oiv -i fsimage文件 -o xxx.xml -p XML`。例如：
	- `hdfs oiv -i fsimage_0000000000000000012 -o fsimage.xml -p XML`
3. fsimage介绍：fsimage是一个二进制文件，当中记录了HDFS中所有文件和目录的元数据信息。
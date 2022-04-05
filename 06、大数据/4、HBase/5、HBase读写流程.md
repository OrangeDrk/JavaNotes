[TOC]



# 写流程

1. 当HRegionServer收到写请求的时候，会默认先将这个写操作记录到WAL中，然后会将数据更新到memStore中

2. 数据在memStore中会进行排序：行键字典升序-> 列族名字典升序 -> 列名字典升序 -> 时间戳倒叙

3. 当memStore达到一定条件之后，会自动进行冲刷，冲刷出HFile

   > 因为数据在memStore中已经排序，所以HFile是局部有序整体无序的，各个HFile把持起始行键和结束行键，最后落地到HDFS上

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/写流程-HFile.png)

4. HFile的格式V1：

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/写流程-HFile格式.png)

   - DataBlock：存储数据

     - DataBlock是HBase中数据存储的最小单位

     - DataBlock默认大小是64KB。小的DataBlock利于查询(get)，大的DataBlock利于遍历(scan)

     - BlockCache的空间局部性是以DataBlock为单位存储：只要DataBlock中有一条数据被读取，那么整个DataBlock都会放到BlockCache中

     - 每一个DataBlock都包含1个Magic和多个KeyValue

       > **Magice**：魔术，本质上是一个随机数。**作用**：对数据进行校验，只要改变过数据，这个随机数就会发生变化
       >
       > **KeyValue**：用于存储数据
       >
       > ![](https://gitee.com/sxhDrk/images/raw/master/imgs/写流程-HFile-DataBlock-KeyValue.png)

   - MetaBlock：存储元数据，绝大部分HFile不包含这一部分，只有`.meta.`文件中会包含

   - FileInfo：对文件信息的描述

   - DataIndex：记录DataIBlock在文件中的存储位置

   - MetaIndex：记录MetaIBlock在文件中的存储位置

   - Trailer：在文件末尾，固定4个字节大小，其中前2个字节记录DataIndex的起始位置，后2个字节记录MetaIndex的起始位置

5. HFile在第二个版本中新添了==布隆过滤器==

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/布隆过滤器.jpg)



# 读流程

1. 当HRegionServer接收到读请求之后，会先从BlockCache中读取
   - 如果BlockCache读不到，则会从memStore中读取
   - 如果memStore也都不到，需要从HFile中读取
2. 会现根据HFile的行键范围进行筛选，筛选完成之后会再利用布隆过滤器来进行筛选，被筛掉的文件中一定是没有要查询的数据的，但是剩下的文件中不能保证一定有要查询的数据



# Compaction机制

1. HBase提供了2种Compaction机制

   - minor compact

     > 将相邻的几个小的HFile合并成一个大的HFile，如果HFile本身很大就不合并

   - major compact

     > 将所有的HFile，不管大小，合并成一个HFile。合并完成之后存在一个HFile

2. 在HBase中，如果不指定，默认使用`minor compact`

   > 实际开发中，一般会在周末的凌晨进行`major compact`
   >
   > `major compact`合并一般是几个小时

3. 在合并过程中，会舍弃被标记删除的数据或者过时的数据

   > 例如一个数据指定保留3个版本的数据，但是在合并过程中发现5个版本，就会舍弃最早的2个版本的数据


[TOC]

# 常见参数


| **配置所在文件** | **参数**                                                     | **参数默认值**                  | **作用**                                                     |
| ---------------- | ------------------------------------------------------------ | ------------------------------- | ------------------------------------------------------------ |
| hdfs-site.xml    | dfs.namenode.support.allow.format                            | true                            | 表示设置NameNode是否允许被格式化。  在生产系统，把它设置为false，阻止任何格式化操作在一个运行的DFS上。  建议初次格式化后，修改配置禁止，改成false |
| hdfs-site.xml    | dfs.heartbeat.interval                                       | 3                               | DataNode的心跳间隔，默认单位为秒  在集群网络通信状态不好的时候，适当调大 |
| hdfs-site.xml    | dfs.blocksize                                                | 134217728                       | 块大小，默认是128MB  必须得是1024(page size)的整数倍         |
| hdfs-site.xml    | dfs.namenode.checkpoint.period  或者：  fs.checkpoint.period | 3600                            | edits和fsimage文件合并周期阈值，默认单位为s                  |
| hdfs-site.xml    | dfs.stream-buffer-size                                       | 4096                            | 文件流缓存大小。需要是硬件page大小的整数倍。在读写操作时，数据缓存大小。  注意：是1024的整数倍  注意和core-default.xml中指定文件类型的缓存是不同的，这个是dfs共用的 |
| mapred-site.xml  | mapreduce.task.io.sort.mb                                    | 100                             | 任务内部排序缓冲区大小，默认单位是MB  此参数调大，能够减少Spil溢写次数，减少磁盘I/O     建议：250MB~400MB |
| mapred-site.xml  | mapreduce.map.sort.spill.percent                             | 0.8                             | Map阶段溢写文件的阈值。不建议修改此值                        |
| mapred-site.xml  | mapreduce.reduce.shuffle.parallelcopies                      | 5                               | ReduceTask  启动的并发拷贝数据的线程数（fetch线程数）  建议，尽可能等于或接近于Map任务数量，达到并行抓取的效果 |
| mapred-site.xml  | mapreduce.job.reduce.slowstart.completedmaps                 | 0.05                            | 当Map任务数量完成率在5%时，Reduce任务启动，这个参数建议不要轻易改动，如果Map任务总量非常大时，可以将此参数调低，让reduce更早开始工作。 |
| mapred-site.xml  | io.sort.factor                                               | 10                              | 文件合并（Merge）因子，如果文件数量太多，可以适当调大，从而减少I/O次数 |
| mapred-site.xml  | mapred.**compress**.map.output                               | false                           | 是否对Map的输出结果文件进行压缩，默认是不压缩。但是如果Map的结果文件很大，可以开启压缩，在Reduce的远程拷贝阶段可以节省网络带宽。 |
| mapred-site.xml  | **mapred.map.tasks.speculative.execution**                   | true  <br />==一般设置为false== | 启动map任务的推测执行机制       推测执行是Hadoop对“拖后腿”的任务的一种优化机制，当一个作业的某些任务运行速度明显慢于同作业的其他任务时，Hadoop会在另一个节点上为“慢任务”启动一个备份任务，这样两个任务同时处理一份数据，而Hadoop最终会将优先完成的那个任务的结果作为最终结果，并将另一个任务杀掉。     启动推测执行机制的目的是更快的完成job，     但是在集群计算资源紧张时，比如同时在运行很多个job，启动推测机制可能会带来相反效果。如果是这样，就改成false。     对于这个参数的控制，轻易不要改动。 |
| mapred-site.xml  | mapred.reduce.tasks.speculative.execution                    | true<br />==一般设置为false==   | 启动reduce任务的推测执行机制                                 |




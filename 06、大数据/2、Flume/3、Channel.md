[TOC]

# Memory Channel

1. 内存通道，将接收到的数据临时存储在内存中
2. 读写速度比较快，但是不可靠



## 可配置选项说明

> 高亮属性为必选项

| 配置项                  | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| ==type==                | memory                                                       |
| ==capacity==            | 默认：100    事件存储在信道中的最大数量  <br />建议实际工作调节：10万     <br />首先估算出每个event的大小，然后再服务的内存来调节 |
| ==transactionCapacity== | 默认：100    每个事务中的最大事件数  <br />建议实际工作调节：1000~3000 |

> capacity：10000-30000	对应transactionCapacity为：1000-3000（1%）

## 示例

```sh
a1.sources = s1
a1.channels = c1
a1.sinks = k1

# 配置Source
a1.sources.s1.type = netcat
a1.sources.s1.bind = 0.0.0.0
a1.sources.s1.port = 8090

# 配置Channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

# 配置Sink
a1.sinks.k1.type = logger

# 将Source和Channel绑定
a1.sources.s1.channels = c1

# 将Sink和Channel绑定
a1.sinks.k1.channel = c1
```



# File Channel

1. 文件通道，将接收到的数据临时存储在磁盘上
2. 读写速度比较慢，但是可靠



## 可配置选项说明

> 高亮属性为必选项

| 配置项       | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| ==type==     | file                                                         |
| ==dataDirs== | 指定存放的目录，逗号分隔的目录列表，用以存放日志文件。使用单独的磁盘上的多个目录可以提高文件通道效率。 |



## 示例

1. 创建数据存目录

   ```sh
   cd /home
   mkdir flumedata	
   ```

   

2. 在flume的data目录下创建：`filechannel.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   
   # 配置FileChannel
   a1.channels.c1.type = file
   # 指定临时存储的目录
   a1.channels.c1.dataDirs = /home/flumedata/
   
   # 配置Sink
   a1.sinks.k1.type = logger
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   ```

3. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f filechannel.conf -Dflume.root.logger=INFO,console
   ```

4. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

5. 此时flume控制台会打印信息

   ```sh
   2020-03-18 12:29:31,359 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 68 65 6C 6C 6F 2C 69 27 6D 20 66 6C 75 6D 65    hello,i'm flume }
   2020-03-18 12:29:40,706 (Log-BackgroundWorker-c1) [INFO - org.apache.flume.channel.file.EventQueueBackingStoreFile.beginCheckpoint(EventQueueBackingStoreFile.java:227)] Start checkpoint for /root/.flume/file-channel/checkpoint/checkpoint, elements to sync = 1
   2020-03-18 12:29:40,727 (Log-BackgroundWorker-c1) [INFO - org.apache.flume.channel.file.EventQueueBackingStoreFile.checkpoint(EventQueueBackingStoreFile.java:252)] Updating checkpoint metadata: logWriteOrderID: 1584505750768, queueSize: 0, queueHead: 0
   2020-03-18 12:29:40,745 (Log-BackgroundWorker-c1) [INFO - org.apache.flume.channel.file.Log.writeCheckpoint(Log.java:1052)] Updated checkpoint for file: /home/flumedata/log-1 position: 172 logWriteOrderID: 1584505750768
   ```

   

6. 查看`/home/flumedata`目录，会生成一个`log-x.meta`，该文件记录的是传输的信息







# 其他Channel

## JDBC Channel

1. 事件会被持久化（存储）到可靠的数据库里
2. 目前只支持嵌入式Derby数据库。但是Derby数据库不太好用，所以JDBC Channel目前仅用于测试，不能用于生产环境。



## 内存溢出通道

1. 优先把Event存到内存中，如果存不下，在溢出到文件中
2. 目前处于测试阶段，还未能用于生产环境
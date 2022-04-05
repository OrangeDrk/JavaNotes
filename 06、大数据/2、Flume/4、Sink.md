[TOC]

# Logger Sink

1. 将数据以日志形式打印
2. 在打印的时候为了防止屏幕显示的内容过多，每次打印body内容不超过16个字节
3. 打印的时候需要指定级别，例如：INFO、DEBUG、WANGN、ERROR等



## 可配置项说明

> 高亮属性为必选项

| 配置项      | 说明     |
| ----------- | -------- |
| ==channel== | 绑定通道 |
| ==type==    | logger   |



## 示例

1. 在flume的data目录下创建：`loggersink.conf`

   ```sh
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的r1
   a1.sources.r1.type=netcat
   a1.sources.r1.bind=0.0.0.0
   a1.sources.r1.port=8090
   
   #描述a1的s1
   a1.sinks.s1.type=logger
   #描述a1的c1
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   ```

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f loggersink.conf -Dflume.root.logger=INFO,console
   ```

3. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

   



# File_roll Sink

1. 将收集的数据写到指定的目录下
2. 需要指定文件滚动事件的时间，不指定会默认30s生成一个小文件



## 可配置项说明

> 高亮属性为必选项

| 配置项             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| ==channel==        | 绑定通道                                                     |
| ==type==           | file_roll                                                    |
| ==sink.directory== | 文件被存储的目录                                             |
| sink.rollInterval  | 30 记录日志到文件里，每隔30秒生成一个新日志文件。如果设置为0，则禁止滚动，从而导致所有数据被写入到一个文件中。 |



## 示例

1. 创建数据收集目录

   ```sh
   cd /home
   mkdir flumedata
   ```

   

2. 在flume的data目录下创建：`filerollsink.conf`

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
   a1.sinks.k1.type = file_roll
   a1.sinks.k1.sink.directory = /home/flumedata
   # 如果不指定，默认每隔30s生成一个文件
   a1.sinks.k1.sink.rollInterval = 3600
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   
   ```

3. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f filerollsink.conf -Dflume.root.logger=INFO,console
   ```

4. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

5. 查看数据数收集目录：`/home/flumedata`，会生成文件，内容为传输的数据

   ```sh
   cat 1584506172023-1 
   hello
   ```

   




# HDFS Sink

1. 将数据收集来之后写到HDFS中
2. 数据往HDFS中写的时候只支持三种格式：
   - 序列：`SequenceFile`
   - 文本：`DataStream`
   - 压缩：`CompressedStream`
3. ==使用的时候最好指定滚动时间==，不指定默认30s生成一个小文件，导致HDFS上产生大量的小文件



## 可配置选项说明

> 高亮属性为必选项

| 配置项                | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| ==channel==           | 绑定的通道                                                   |
| ==type==              | hdfs                                                         |
| ==hdfs.path==         | HDFS 目录路径  （hdfs://namenode/flume/webdata/)             |
| hdfs.inUseSuffix      | .tmp    Flume正在处理的文件所加的后缀                        |
| ==hdfs.rollInterval== | 文件生成的间隔事件，默认是30，单位是秒                       |
| hdfs.rollSize         | 生成的文件大小，默认是1024个字节 ，0表示不开启此项           |
| hdfs.rollCount        | 每写几条数据就生成一个新文件，默认数量为10  每写几条数据就生成一个新文件， |
| ==hdfs.fileType==     | SequenceFile/DataStream/CompressedStream                     |
| hdfs.retryInterval    | 80    Time  in seconds between consecutive attempts to close a file. Each close call  costs multiple RPC round-trips to the Namenode, so setting this too low can  cause a lot of load on the name node. If set to 0 or less, the sink will not  attempt to close the file if the first attempt fails, and may leave the file  open or with a ”.tmp” extension. |



## 示例

1. 启动Hadoop伪分布式

   ```sh
   start-all.sh
   ```

2. 在flume的data目录下创建：`hdfssink.conf`

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
   a1.sinks.k1.type = hdfs
   a1.sinks.k1.hdfs.path = hdfs://hadoop01:9000/flume
   a1.sinks.k1.hdfs.fileType = DataStream
   a1.sinks.k1.hdfs.rollInterval = 3600
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   ```
   
3. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f hdfssink.conf -Dflume.root.logger=INFO,console
   ```

4. 新窗口通过nc发送信息

   ```sh
   nc hadoop01 8090
   lunch
   OK
   breakfast
   OK
   ```

5. 查看HDFS文件，在`/flume`目录下会生成一个数据文件，内容为输入的数据

   ```
   lunch
   breakfast
   ```

   



# Avro Sink

1. 将源数据利用AVRO序列化后写到指定的节点上
2. 将数据序列化之后写出，结合AVRO Source来实现**多级流动**、**扇入流动**、**扇出流动**的效果

## 可配置选项说明

> 高亮属性为必选项

| 配置项       | 说明           |
| ------------ | -------------- |
| ==channel==  | 绑定的通道     |
| ==type==     | avro           |
| ==hostname== | 要发送的主机   |
| ==port==     | 要发往的端口号 |

## 示例

### 多级流动

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Sink-多级流动.png)

> 多级流动案例。hadoop01   ->  hadoop02  ->  hadoop03 --日志打印输出

1. 在flume的data目录下创建`duoji.conf`配置文件

   - hadoop01的`duoji.conf`

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
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop02
     a1.sinks.k1.port = 8090
     
     # 将Source和Channel绑定
     a1.sources.s1.channels = c1
     
     # 将Sink和Channel绑定
     a1.sinks.k1.channel = c1
     ```

     

   - hadoop02的`duoji.conf`

     ```sh
     a1.sources = s1
     a1.channels = c1
     a1.sinks = k1
     
     # 配置Source
     a1.sources.s1.type = avro
     a1.sources.s1.bind = 0.0.0.0
     a1.sources.s1.port = 8090
     
     # 配置Channel
     a1.channels.c1.type = memory
     a1.channels.c1.capacity = 10000
     a1.channels.c1.transactionCapacity = 100
     
     # 配置Sink
     a1.sinks.k1.type = avro
     a1.sinks.k1.hostname = hadoop03
     a1.sinks.k1.port = 8090
     
     # 将Source和Channel绑定
     a1.sources.s1.channels = c1
     
     # 将Sink和Channel绑定
     a1.sinks.k1.channel = c1
     ```

     

   - hadoop03的`duoji.conf`

     ```sh
     a1.sources = s1
     a1.channels = c1
     a1.sinks = k1
     
     # 配置Source
     a1.sources.s1.type = avro
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

2. 倒序启动flume

   ```sh
   #hadoop03
   ../bin/flume-ng agent -n a1 -c ../conf -f duoji.conf -Dflume.root.logger=INFO,console
   #hadoop02
   ../bin/flume-ng agent -n a1 -c ../conf -f duoji.conf -Dflume.root.logger=INFO,console
   #hadoop01
   ../bin/flume-ng agent -n a1 -c ../conf -f duoji.conf -Dflume.root.logger=INFO,console
   ```

3. 在hadoop01节点通过nc向hadoop01发送信息

   ```sh
   nc hadoop01 8090
   hello
   ```

4. 会在hadoop03节点收到传输的avro信息

   ```sh
   2020-03-18 12:01:40,835 (SinkRunner-PollingRunner-DefaultSinkProcessor) 
   	[INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] 
   	Event: { headers:{} body: 68 65 6C 6C 6F                                  hello }
   ```

   

### 扇入流动

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Sink-扇入流动.png)

### 扇出流动

![](https://gitee.com/sxhDrk/images/raw/master/imgs/事务.png)



# 自定义Sink

需要去写一个类实现Sink、Configurable，在Sink完成过程中需要注意事务的问题。在这个过程中需要覆盖configure、start、process、stop方法



## Sink

```java
import org.apache.flume.*;
import org.apache.flume.conf.Configurable;
import org.apache.flume.sink.AbstractSink;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.PrintStream;
import java.util.Map;

// 模拟File_roll Sink
public class AuthSink extends AbstractSink implements Sink, Configurable {

    private File file;
    private PrintStream ps;

    @Override
    public void configure(Context context) {
        // 获取属性值
        String dir = context.getString("dir");
        // 判断是否指定了输出的目录
        if (dir == null) {
            throw new NullPointerException("存储的目录路径不能为空！！！");
        }
        file = new File(dir);
    }

    @Override
    public synchronized void start() {
        System.out.println("Sink已经启动~~~");
        // 判断路径是否存在
        if (!file.exists())
            // 如果路径不存在，试着创建这个路径
            file.mkdirs();
        // 初始化流对象
        try {
            ps = new PrintStream(file.getPath() + "/" + System.currentTimeMillis());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        super.start();
    }

    @Override
    public Status process() {

        // 获取Channel
        Channel c = super.getChannel();
        // 获取Transaction
        Transaction t = c.getTransaction();
        // 开启事务
        t.begin();
        // 创建Event
        Event e;
        try {
            while ((e = c.take()) != null) {
                Map<String, String> headers = e.getHeaders();
                byte[] body = e.getBody();
                ps.println("headers:");
                for (Map.Entry<String, String> entry : headers.entrySet()) {
                    ps.println("\t" + entry.getKey() + ":" + entry.getValue());
                }
                ps.println("body:");
                ps.println("\t" + new String(body));
            }
            t.commit();
        } catch (Exception ex) {
            t.rollback();
            ex.printStackTrace();
            return Status.BACKOFF;
        } finally {
            t.close();
        }
        return Status.READY;
    }

    @Override
    public synchronized void stop() {
        ps.close();
        System.out.println("Sink已经执行完了");
        super.stop();
    }
}
```



## Flume配置文件

```sh
a1.sources = s1
a1.channels = c1
a1.sinks = k1

a1.sources.s1.type = netcat
a1.sources.s1.bind = 0.0.0.0
a1.sources.s1.port = 8090

a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

a1.sinks.k1.type = cn.tedu.sink.AuthSink
a1.sinks.k1.sink.dir = /home/flumedata
a1.sinks.k1.sink.rollInterval = 3600

a1.sources.s1.channels = c1
a1.sinks.k1.channel = c1
```





# 事务

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Sink-扇出流动.png)
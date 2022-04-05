[TOC]

# AVRO Source

## 概述

1. 监听Avro 端口来接收外部avro客户端的事件流
2. avro-source接收到的是经过avro序列化后的数据，然后反序列化数据继续传输。
3. 源数据必须是经过avro序列化后的数据
4. 利用Avro source可以实现**多级流动、扇出流、扇入流**等效果
5. 可以接收通过flume提供的avro客户端发送的日志信息

## 可配置选项说明

> 高亮属性为必选项

| **配置项**     | **说明**             |
| -------------- | -------------------- |
| ==channels==   | 绑定通道             |
| ==type==       | avro                 |
| ==bind==       | 需要监听的主机名或IP |
| ==port==       | 要监听的端口         |
| threads        | 工作线程最大线程数   |
| selector.*     | 选择器配置           |
| interceptors.* | 拦截器配置           |



## 示例

1. 在Flume目录下的data目录下创建`avrosource.conf`文件

2. 根据指定的配置文件启动Flume，配置文件内容如下：

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的source1
   a1.sources.r1.type=avro  
   a1.sources.r1.bind=0.0.0.0
   a1.sources.r1.port=8090
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sinka
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

3. 执行启动命令

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f avrosource.conf  -Dflume.root.logger=INFO,console
   ```

   如果出现`Avro source source1 started`，说明启动成功

4. 新窗口进入Flume的bin目录，创建文件`a.txt`

   ```sh
   cd /home/software/apache-flume-1.7.0-bin/bin
   vim a.txt
   	#文件内容：hello,i'm flume
   ```

5. 将文件序列化之后发送给Flume

   ```sh
   sh flume-ng avro-client -H hadoop01 -p 8090 -c ../conf -F a.txt
   ```

6. Flume日志就会打印该文件的内容日志



# Exec Source

## 概述

可以将命令产生的输出作为源来进行传递

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明           |
| -------------- | -------------- |
| ==channels==   | 绑定的通道     |
| ==type==       | exec           |
| ==command==    | 要执行的命令   |
| selector.*     | 选择器配置     |
| interceptors.* | 拦截器列表配置 |

## 示例

1. 在flume的data目录下创建文件：`execsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的source1
   a1.sources.r1.type=exec  
   a1.sources.r1.command=ls /home
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 启动Flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f execsource.conf  -Dflume.root.logger=INFO,console
   ```

3. flume日志信息

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/execsource打印日志.png)



# Spooling Directory Source

## 概述

1. flume会持续监听指定的目录，把放入这个目录中的文件当做Source来处理
2. 注意：一旦文件被放到“自动收集”目录中后，便必能修改，如果修改，flume会报错
3. 此外，也不能有重名的文件，如果有，flume也会报错

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明                         |
| -------------- | ---------------------------- |
| ==channels==   | 绑定通道                     |
| ==type==       | spooldir                     |
| ==spoolDir==   | 读取文件的路径，即"搜集目录" |
| selector.*     | 选择器配置                   |
| interceptors.* | 拦截器配置                   |

## 示例

1. 在flume的data目录中创建文件：`spooldirsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.channels=c1
   a1.sinks=s1
   
   #描述/配置a1的source1
   a1.sources.r1.type=spooldir
   a1.sources.r1.spoolDir=/home/data
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 创建搜集目录

   ```sh
   cd /home/
   mkdir data
   ```

3. 启动flum

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f spooldirsource.conf  -Dflume.root.logger=INFO,console
   ```

4. 向搜集目录添加文件

   ```sh
   [root@hadoop01 home]# cp a.txt data 
   ```

5. 查看flume控制台日志打印信息



# NetCat Source

## 概述

1. 一个NetCat Source用来监听一个指定端口，并接收监听到的数据
2. 接收的数据是字符串形式

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明                 |
| -------------- | -------------------- |
| ==channels==   | 绑定通道             |
| ==type==       | netcat               |
| ==port==       | 指定要绑定到的端口号 |
| selector.*     | 选择器配置           |
| interceptors.* | 拦截器配置           |

## 示例

1. 在flume的data目录下创建文件：`ncsource.conf`

   ```sh
   #配置Agent a1 的组件
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
   ../bin/flume-ng agent -n a1 -c ../conf -f ncsource.conf -Dflume.root.logger=INFO,console
   ```

3. 新窗口通过nc发送消息

   ```sh
   nc hadoop01 8090
   hello
   ```

4. 查看flume控制台打印的日志信息



# Sequence Generator Source

## 概述

1. 一个简单的序列发生器，不断的产生时间，值从0开始，每次递增1
2. ==主要用来测试==

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明               |
| -------------- | ------------------ |
| ==channels==   | 绑定的通道         |
| ==type==       | seq                |
| selector.*     | 选择器配置         |
| interceptors.* | 拦截器配置         |
| batchSize      | 递增步长， 默认是1 |

## 示例

1. 在flume的data目录下创建文件：`seqsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.sinks=s1
   a1.channels=c1
   
   #描述/配置a1的source1
   a1.sources.r1.type=seq
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 启动flume

   ```sh
   ../bing/flume-ng agent -n a1 -c ../conf -f seqsource.conf -Dflume.root.logger=INFO,console
   ```



# HTTP Source

## 概述

1. 此Source接收HTTP的GET和POST请求作为Flume的事件
2. GET方式只用于试验，所以实际使用过程中以POST请求居多
3. 如果想让flume正确解析HTTP协议信息，比如解析出请求头、请求体等信息，需要提供一个可插拔的“处理器”来将请求转换为事件对象，这个处理器必须实现HTTPSourceHandler接口。
4. 这个处理器接受一个HttpServletRequest对象，并返回一个Flume Event对象集合

> 常用Handler

1. JSONHandler
   - 可以处理JSON格式的数据，并支持UTF-8     UTF-16 UTF-32字符集

   - 该handler接受Event数组，并根据请求头中指定的编码将其转换为Flume     Event

   - 如果没有指定编码，默认编码为UTF-8

   - 格式：

     ```json
     [
         {
             "headers" : {
                 "timestamp" : "434324343",
                 "host" : "random_host.example.com"
             }
             "body" : "random_body"
         },
         {
             "headers" : {
                 "namenode" : "namenode.example.com",
                 "datanode" : "random_datanode.example.com"
             },
             "body" : "really_random_body"
         }
     ]
     ```

     

2. BlobHandler

   - BlobHandler是一种将请求中上传文件信息转化为event的处理器
   - BlobHandler适合大文件的传输

## 可配置选项说明

> 高亮属性为必选项

| 配置项         | 说明       |
| -------------- | ---------- |
| ==channels==   | 绑定的通道 |
| ==type==       | http       |
| selector.*     | 选择器配置 |
| interceptors.* | 拦截器配置 |
| ==port==       | 端口       |

## 示例

1. 在flume的data目录下创建文件：`httpsource.conf`

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.sinks=s1
   a1.channels=c1
   
   #描述/配置a1的source1
   a1.sources.r1.type=http
   a1.sources.r1.port=8090
   
   #描述sink
   a1.sinks.s1.type=logger
   
   #描述内存channel
   a1.channels.c1.type=memory
   a1.channels.c1.capacity=1000
   a1.channels.c1.transactionCapacity=100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   
   ```

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f httpsource.conf -Dflume.root.logger=INFO,console
   ```

3. 新窗口执行curl命令,模拟HTTP的post请求

   ```sh
   curl -X POST -d '[{"headers":{"a":"a1","b":"b1"},"body":"hello http-flume"}]'  http://10.42.0.33:8090
   ```

4. 查看flume控制台日志打印



# 自定义Source

1. Flume中的Source的顶级接口：Source
2. 如果需要自定义Source，那么需要写一个类实现EventDrivenSource或者PollableSource
   - EventDrivenSource：被动型Source。需要自定义线程来获取数据
   - PollableSource：主动型Source。主动提供线程来获取数据
3. 实际场景中，需要根据不同的场景来提供不同的Source
4. 自定义的Source需要打成jar包放到flume安装目录的lib目录下，然后再格式文件中需要给定类的全路径名



## Source类

```java
import org.apache.flume.Context;
import org.apache.flume.Event;
import org.apache.flume.EventDrivenSource;
import org.apache.flume.channel.ChannelProcessor;
import org.apache.flume.conf.Configurable;
import org.apache.flume.event.EventBuilder;
import org.apache.flume.source.AbstractSource;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class AuthSource extends AbstractSource implements EventDrivenSource, Configurable {
    private int step = 1;
    private ExecutorService es;

    /*
        获取格式文件中指定属性的值
        a1.sources.s1.step = xxx
     */
    @Override
    public void configure(Context context) {
        //用户指定的自增的步长
        String step = context.getString("step");
        //  如果用户指定了步长，就要按照用户指定的数值自增。如果用户没有指定步长，默认每次自增1
        if (step != null) {
            this.step = Integer.parseInt(step);
        }
    }

    /*
        启动source
     */
    @Override
    public synchronized void start() {
        System.out.println("Source开始执行");
        es = Executors.newFixedThreadPool(5);
        //获取Channel处理器
        ChannelProcessor cp = super.getChannelProcessor();

        //提交任务，开始执行任务
        es.submit(new Add(step,cp));
    }

    @Override
    public synchronized void stop() {
        es.shutdownNow();
        System.out.println("source执行结束");
    }


}

class Add implements Runnable {

    private int step;
    private ChannelProcessor cp;

    public Add(int step,ChannelProcessor cp) {
        this.cp = cp;
        this.step = step;
    }

    @Override
    public void run() {
        int i = 0;
        while (true) {
            //flume会将收集的数据封装成event
            //构建一个headers
            Map<String, String> headers = new ConcurrentHashMap<>();
            headers.put("tmie",System.currentTimeMillis()+"");
            // 第一个参数：body
            // 第二个参数：headers
            byte[] body = (i + "").getBytes();
            Event e = EventBuilder.withBody(body, headers);

            //将event传递给Channel
            cp.processEvent(e);
            i += step;
        }
    }
}

```



## 打成jar包

![](https://gitee.com/sxhDrk/images/raw/master/imgs/自定义Source-打jar包2.png)

> Build - Build Artifacts，生成jar包

![](https://gitee.com/sxhDrk/images/raw/master/imgs/自定义Source-打jar包3.png)

> 生成的jar包存在`out -> artifacts -> G_Flume_jar`下

![](https://gitee.com/sxhDrk/images/raw/master/imgs/自定义Source-打jar包.png)

## Flume配置文件

```sh
a1.sources = s1
a1.channels = c1
a1.sinks = k1

# 自定义source的类型要写类的全路径名
a1.sources.s1.type = cn.tedu.source.AuthSource

a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

a1.sinks.k1.type = logger

a1.sources.s1.channels = c1
a1.sinks.k1.channel = c1
```


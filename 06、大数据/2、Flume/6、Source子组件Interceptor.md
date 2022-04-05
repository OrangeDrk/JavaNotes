

[TOC]



# Interceptor

1. 本身是Source的子组件，需要配置在Source上
2. 可以配置多个，按照配置顺序形成拦截链



## Timestamp Interceptor

1. 这个拦截器在事件头中插入以毫秒为单位的当前处理时间
2. 头的名字为`timestamp`，值为当前处理的时间戳
3. 如果在之前已经有这个时间戳，则保留原有的时间戳

> 结合HDFS Sink使用，可以实现按天收集的效果

### 可配置项

| 配置项           | 说明                                |
| ---------------- | ----------------------------------- |
| ==type==         | timestamp                           |
| preserveExisting | false    如果时间戳已经存在是否保留 |

### 示例

1. 配置文件：`timestampin.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   a1.sources.s1.interceptors = i1
   a1.sources.s1.interceptors.i1.type = timestamp
   
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

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f timestampin.conf -Dflume.root.logger=INFO,console
   ```

3. 通过nc发送数据

   ```sh
   nc hadoop01 8090
   hello
   OK
   ```

4. flume控制台日志打印

   ```sh
   Event: { headers:{timestamp=1584548779108} body: 68 65 6C 6C 6F                                  hello }
   ```



> 结合HDFS Sink使用，实现按天收集的效果
>
> 当天收集的数据会放到一个文件夹内，00:00过后会放入新的文件夹

```sh
a1.sources = s1
a1.channels = c1
a1.sinks = k1

# 配置Source
a1.sources.s1.type = netcat
a1.sources.s1.bind = 0.0.0.0
a1.sources.s1.port = 8090
a1.sources.s1.interceptors = i1
a1.sources.s1.interceptors.i1.type = timestamp

# 配置Channel
a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

# 配置Sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.hdfs.path = hdfs://hadoop01:9000/log/reportTime=%Y-%m-%d
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.rollInterval = 3600

# 将Source和Channel绑定
a1.sources.s1.channels = c1

# 将Sink和Channel绑定
a1.sinks.k1.channel = c1

```



## Host Interceptor

> 在日志信息的headers中添加一个host字段

1. 这个拦截器插入当前Agent的主机名或者IP
2. 头的名字为host或者配置的名称
3. 值是主机名或者ip地址，基于配置



### 可配置项

| 配置参数         | 说明                                                |
| ---------------- | --------------------------------------------------- |
| ==type==         | host                                                |
| preserveExisting | false    如果主机名已经存在是否保留                 |
| useIP            | true    如果配置为true则用IP，配置为false则用主机名 |
| hostHeader       | host    加入头时使用的名称                          |



### 示例

1. 配置文件：`hostin.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   a1.sources.s1.interceptors = i2
   a1.sources.s1.interceptors.i2.type = host
   
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

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f hostin.conf -Dflume.root.logger=INFO,console
   ```

3. 通过nc发送数据

   ```sh
   nc hadoop01 8090
   hello
   OK
   ```

4. flume控制台打印的信息

   ```sh
   Event: { headers:{host=10.42.0.33} body: 68 65 6C 6C 6F      hello }
   ```

   



## Static Interceptor

> 在日志信息的headers中添加一个指定的字段

1. 此拦截器允许用户增加静态头信息，使用静态的值到所有事件
2. 目前的实现中不允许一次指定多个头
3. 如果需要增加多个静态头，可以指定多个Static Interceptors



### 可配置项

| 配置项           | 说明                  |
| ---------------- | --------------------- |
| ==type==         | static                |
| preserveExisting | true                  |
| ==key==          | key    要增加的头名   |
| ==value==        | value    要增加的头值 |



### 示例

1. flume配置文件：`staticin.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   a1.sources.s1.interceptors = i3
   a1.sources.s1.interceptors.i3.type = static
   a1.sources.s1.interceptors.i3.key = subject
   a1.sources.s1.interceptors.i3.value = bigdata
   
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

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f staticin.conf -Dflume.root.logger=INFO,console
   ```

3. 通过nc发送数据

   ```sh
   nc hadoop1 8090
   hi
   OK
   ```

4. flume控制台打印信息

   ```sh
   Event: { headers:{subject=bigdata} body: 68 69        hi }
   ```

   





## UUID Interceptor

> 在日志信息的headers中添加一个id，作用往往是标记数据的唯一性，保证数据不重复

1. 这个拦截器在所有事件头中增加一个全局一致性的标志，就是UUID



### 可配置项

| 配置项           | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| ==type==         | `org.apache.flume.sink.solr.morphline.UUIDInterceptor$Builder  ` |
| headerName       | `id`：头名称                                                 |
| preserveExisting | `true`：如果头已经存在，是否保留                             |
| prefix           | `""`：在UUID前拼接的字符串前缀                               |



### 示例

1. flume配置文件：`uuidin.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   a1.sources.s1.interceptors = i4
   a1.sources.s1.interceptors.i4.type = org.apache.flume.sink.solr.morphline.UUIDInterceptor$Builder
   
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

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f uuidin.conf -Dflume.root.logger=INFO,console
   ```

3. 通过nc发送数据

   ```sh
   nc hadoop01 8090
   nihao
   OK
   ```

4. flume控制台打印信息

   ```sh
   Event: { headers:{id=e230e125-56de-4e15-a8ee-a02b408d698a} body: 6E 69 68 61 6F       nihao }
   ```

   



## Search And Replace Interceptor

> 会根据指定的正则表达式去搜索Body中的数据，将符合正则表达式的数据进行替换

1. 这个拦截器提供了简单的基于字符串的正则搜索和替换功能



### 可配置项

| 配置项            | 说明                           |
| ----------------- | ------------------------------ |
| ==type==          | search_replace                 |
| ==searchPattern== | 要搜索和替换的正则表达式       |
| ==replaceString== | 要替换为的字符串               |
| charset           | UTF-8    字符集编码，默认utf-8 |



### 示例

1. flume的配置文件：`searchreplace.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   a1.sources.s1.type = http
   a1.sources.s1.port = 8090
   a1.sources.s1.interceptors = i1
   a1.sources.s1.interceptors.i1.type = search_replace
   a1.sources.s1.interceptors.i1.searchPattern = [0-9]
   a1.sources.s1.interceptors.i1.replaceString = *
   
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   a1.sinks.k1.type = logger
   
   a1.sources.s1.channels = c1
   a1.sinks.k1.channel = c1
   ```

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f searchreplace.conf -Dflume.root.logger=INFO,console
   ```

3. 通过curl发送http请求

   ```sh
   curl -X POST -d '[{"headers":{"class":"bigdata"},"body":"data123456"}]' http://hadoop01:8090
   ```

4. flume控制台打印信息

   ```sh
   Event: { headers:{class=big1910} body: 62 69 67 2A 2A 2A 2A    data****** }
   ```

   





## Regex Filtering Interceptor

> 需要制定一个正则表达式，只要符合正则表达式的数据就会被拦截掉

1. 此拦截器通过解析事件体去匹配给定正则表达式来筛选事件
2. 所提供的正则表达式即可以用来包含或剔除事件



### 可配置项

| 配置项        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| ==type==      | `regex_filter`                                               |
| ==regex==     | `".*"`： 所要匹配的正则表达式                                |
| excludeEvents | ` false`：如果是true则刨除匹配的事件，false则包含匹配的事件。 |



### 示例

1. flume配置文件：`regexfilter.conf`

   ```sh
   a1.sources = s1
   a1.channels = c1
   a1.sinks = k1
   
   # 配置Source
   a1.sources.s1.type = netcat
   a1.sources.s1.bind = 0.0.0.0
   a1.sources.s1.port = 8090
   a1.sources.s1.interceptors = i1
   a1.sources.s1.interceptors.i1.type = regex_filter
   a1.sources.s1.interceptors.i1.regex = .*[0-9].*
   a1.sources.s1.interceptors.i1.excludeEvents = true
   
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

2. 启动flume

   ```sh
   ../bin/flume-ng agent -n a1 -c ../conf -f regexfilter.conf -Dflume.root.logger=INFO,console
   ```

3. 通过curl发送HTTP请求

   ```sh
   curl -X POST -d '[{"headers":{"class":"big1910"},"body":"big1910abc"}]' http://hadoop01:8090
   ```

4. body含有数字的都被过滤掉了，所以没有日志打印

   ```sh
   2020-03-19 01:33:07,894 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 41 63 63 65 70 74 3A 20 2A 2F 2A 0D             Accept: */*. }
   2020-03-19 01:33:07,894 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 43 6F 6E 74 65 6E 74 2D 54 79 70 65 3A 20 61 70 Content-Type: ap }
   2020-03-19 01:33:07,894 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:95)] Event: { headers:{} body: 0D                                              . }
   ```

   






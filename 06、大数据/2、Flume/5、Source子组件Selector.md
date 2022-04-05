# Selector

1. Selector本身是Source的子组件，需要配置在Source上

2. 模式：

   - replication：复制模式

     > 当第一个节点收到数据的时候，会将这个数据复制然后发送给每一个节点，所以每一个节点的数据是相同的

   - multiplexing：路由模式/多路复用模式

     > 当第一个节点收到数据的时候，会根据这个数据中的指定字段来进行判断，然后转发给不同的节点。所以在这种模式下，每一个节点收到的数据是不同的

3. 如果不指定，默认使用的是复制模式

4. 在实际生产中，如果对数据进行备份，那么使用复制模式；如果对数据分类，那么使用路由模式





## 复制模式

> Selector默认是复制模式（replicating），即把Source复制，然后分发给多个sink

### 可配置选项

| 配置项            | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| selector.type     | replicating   表示复制模式，source的selector如果不配置，默认就是这种模式  在复制模式下，当source接收到数据后，会复制多分，分发给每一个avro sink |
| selector.optional | 标志通道为可选                                               |

### 示例

1. hadoop01配置

   ```sh
   a1.sources = s1
   a1.channels = c1 c2
   a1.sinks = k1 k2
   
   # 配置Source
   a1.sources.s1.type = http
   a1.sources.s1.port = 8090
   a1.sources.s1.selector.type = replicating
   a1.sources.s1.selector.optional = c2
   
   # 配置Channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 10000
   a1.channels.c2.transactionCapacity = 100
   
   
   # 配置Sink
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop03
   a1.sinks.k1.port = 8090
   
   a1.sinks.k2.type = avro
   a1.sinks.k2.hostname = hadoop02
   a1.sinks.k2.port = 8090
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1 c2
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   ```

2. hadoop02和hadoop03配置

   ```sh
   #配置Agent a1 的组件
   a1.sources=r1
   a1.sinks=s1
   a1.channels=c1
   
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
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   ```

   



## 多路复用模式

> 在这种模式下，用户可以指定转发的规则。selector根据规则进行数据的分发



### 可配置选项

| 配置项             | 说明                                     |
| ------------------ | ---------------------------------------- |
| selector.type      | `multiplexing` 表示路由模式              |
| selector.header    | 指定要监测的头的名称                     |
| selector.mapping.* | 匹配规则                                 |
| selector.default   | 如果未满足匹配规则，则默认发往指定的通道 |



### 示例

> 01机利用HTTP Source接收数据，根据路由规则，发往02，03机。02，03通过avro source接收数据，通过logger sink打印数据

1. hadoop01机配置

   ```sh
   a1.sources = s1
   a1.channels = c1 c2
   a1.sinks = k1 k2
   
   # 配置Source
   a1.sources.s1.type = http
   a1.sources.s1.port = 8090
   a1.sources.s1.selector.type = multiplexing
   a1.sources.s1.selector.header = class
   a1.sources.s1.selector.mapping.big1910 = c1
   a1.sources.s1.selector.mapping.big1911 = c2
   a1.sources.s1.selector.default = c2
   
   # 配置Channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 10000
   a1.channels.c1.transactionCapacity = 100
   
   a1.channels.c2.type = memory
   a1.channels.c2.capacity = 10000
   a1.channels.c2.transactionCapacity = 100
   
   
   # 配置Sink
   a1.sinks.k1.type = avro
   a1.sinks.k1.hostname = hadoop03
   a1.sinks.k1.port = 8090
   
   a1.sinks.k2.type = avro
   a1.sinks.k2.hostname = hadoop02
   a1.sinks.k2.port = 8090
   
   # 将Source和Channel绑定
   a1.sources.s1.channels = c1 c2
   
   # 将Sink和Channel绑定
   a1.sinks.k1.channel = c1
   a1.sinks.k2.channel = c2
   
   ```

2. hadoop02和hadoop03机配置

   ```sh
   #配置Agent a1 的组件
   a1.sources = r1
   a1.sinks = s1
   a1.channels = c1
   
   #描述/配置a1的source1
   a1.sources.r1.type = avro
   a1.sources.r1.bind = 0.0.0.0
   a1.sources.r1.port = 8090
   
   #描述sink
   a1.sinks.s1.type = logger
   
   #描述内存channel
   a1.channels.c1.type = memory
   a1.channels.c1.capacity = 1000
   a1.channels.c1.transactionCapacity = 100
   
   #为channel 绑定 source和sink
   a1.sources.r1.channels=c1
   a1.sinks.s1.channel=c1
   ```

   
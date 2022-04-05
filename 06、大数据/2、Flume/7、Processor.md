[TOC]



> Processor：控制器

# 概述

1. 在Flume中，如果不指定，那么默认一个Sink就对应一个Sinkgroup，允许将一个或者多个Sink绑定在一个Sinkgroup
2. Flume Sink Processor 可以通过切换组内Sink用来实现负载均衡的效果，或在一个Sink故障时切换到另一个Sink

![](https://gitee.com/sxhDrk/images/raw/master/imgs/sinkgroup.jpeg)

# Default Sink Processor

1. 只接受一个Sink
2. 这是默认的策略，即如果==不配置Processor用的是这个策略==



## 可配置选项

| 配置项             | 说明                 |
| ------------------ | -------------------- |
| ==sinks==          | 用空格分隔的Sink集合 |
| ==processor.type== | default              |





# Failover Sink Processor

> 使用的时候需要指定优先级，只要优先级高的Sink存活，那么优先级低的Sink就收不到数据

1. 维护一个sink们的优先表。确保只要一个Sink是可用的就能正常处理

2. 失败处理原理：为失效的Sink指定一个冷却时间，在冷却时间到达后再重新使用

3. Sink们可以被配置一个优先级，==数字越大优先级越高==

4. 如果Sink发送事件失败，则下一个最高优先级的Sink将会尝试接着发送该事件

5. 如果没有指定优先级，则优先级顺序取决于Sink们的配置顺序，先配置的默认优先级高

6. 在配置的过程中，设置一个`group processor`，并未每一个Sink都指定一个优先级

   > 优先级必须是唯一的

7. 另外可以设置`maxpenalty`属性指定限定失败时间



## 可配置项

| 配置项               | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| ==sinks==            | 绑定的sink                                                   |
| ==processor.type==   | failover                                                     |
| processor.priority   | 设置优先级，注意，每个sink的优先级必须是唯一的               |
| processor.maxpenalty | 30000    The  maximum backoff period for the failed Sink (in millis) |



## 示例

```sh
a1.sources = s1
a1.channels = c1 c2
a1.sinks = k1 k2

a1.sinkgroups = g1
a1.sinkgroups.g1.sinks = k1 k2
a1.sinkgroups.g1.processor.type = failover
a1.sinkgroups.g1.processor.priority.k1 = 7
a1.sinkgroups.g1.processor.priority.k2 = 3
a1.sinkgroups.g1.processor.maxpenalty = 100000

a1.sources.s1.type = netcat
a1.sources.s1.bind = 0.0.0.0
a1.sources.s1.port = 8090

a1.channels.c1.type = memory
a1.channels.c1.capacity = 10000
a1.channels.c1.transactionCapacity = 100

a1.channels.c2.type = memory
a1.channels.c2.capacity = 10000
a1.channels.c2.transactionCapacity = 100

a1.sources.s1.channels = c1 c2
a1.sinks.k1.channel = c1
a1.sinks.k2.channel = c2
```








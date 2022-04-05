[TOC]

# Java的客户端JedisCluster

## 介绍

jedis客户端为redis-cluster准备的一个客户端代码，底层封装了连接池。

只要创建出一个JedisCluster对象，就可以实现连接池的使用。也能实现客户端代码的高可用



## 测试使用

```java
//测试连接redis-cluster集群
@Test
public void test06(){
    //准备创建对象JedisCluster的参数，
    //集群的连接信息的收集，由于集群两两互联，相互可以获取信息，只需要提供一部分节点
    //8000 8001
    Set<HostAndPort> info=new HashSet<>();
    info.add(new HostAndPort("10.9.104.184",8000));
    info.add(new HostAndPort("10.9.104.184",8001));
    
    //连接池属性
    JedisPoolConfig config=new JedisPoolConfig();
    config.setMaxIdle(8);
    config.setMinIdle(3);
    config.setMaxTotal(200);
    
    //创建最终对象
    JedisCluster cluster=new JedisCluster(info,config);//从8000 8001拿到第一个可连接

    //获取整个集群节点信息 cluster nodes封装槽道逻辑
    for(int i=0;i<100;i++){
        cluster.set("key_"+i,"name");
    }
}
```



# SpringBoot整合JedisCluster

## 配置类

```java
@Configuration
@ConfigurationProperties(prefix = "redis.cluster")
public class ClusterConfig {
    //准备几个为bean对象创建做的属性
    private List<String> nodes;//redis.cluster.nodes
    private Integer maxTotal;//redis.cluster.maxTotal
    private Integer maxIdle;//redis.cluster.maxIdle
    private Integer minIdle;//redis.cluster.minIdle
    
    //get/set方法

    //初始化对象的方法
    @Bean
    public JedisCluster initCluster(){
        //收集节点信息。解析nodes属性
        Set<HostAndPort> info=new HashSet<>();
        for (String node:nodes){
            //node="10.9.104.184:800*";
            String host=node.split(":")[0];
            int port=Integer.parseInt(node.split(":")[1]);
            info.add(new HostAndPort(host,port));
        }

        //连接池的属性JedisPoolConfig
        JedisPoolConfig config=new JedisPoolConfig();
        config.setMaxTotal(maxTotal);
        config.setMaxIdle(maxIdle);
        config.setMinIdle(minIdle);
        return new JedisCluster(info,config);
    }

}
```



## application.properties

> ==这里的nodes节点信息要写2个以上的主节点，以防其中一个节点宕机导致无法获取集群信息==

```properties
redis.cluster.maxTotal=200
redis.cluster.maxIdle=8
redis.cluster.minIdle=3
redis.cluster.nodes=10.9.104.184:8000,10.9.104.184:8001,10.9.104.184:8002
```



## 测试结果

系统启动后，相当于内存就具备了一个框架维护的jedisCluster对象，能够连接8000-8005集群实现分布式计算。当集群中某一个master宕机之后，只要集群技术端，没有崩溃（没有失效）。代码端永远可以正常操作集群，实现数据的读写。

（只要给出的主节点有一个没有宕机，就能通过该节点获取其他节点信息，恢复整个集群的槽道信息和节点信息）



# JedisCluster的槽道逻辑

> redis-cluster中分布式计算就是槽道。
>
> jedisCluster就是将槽道原理和数据结构对应封装了起来，实现==通过槽道计算==每次命令==直接发送给对应节点==。
>
> 例如：操作 `set name wanglaoshi`,代码中计算槽道号`5798`，找到当前redis-cluster 的5798槽道对应的节点：`8001`，将会使用底层`8001`的jedis对象执行`set name wanglaoshi`。



## JedisCluster初始化封装数据

> 使用2个map对象
>
> - 一个保存了节点和连接池对应关系
> - 一个保存了槽道和连接池对应关系。



### 初始化流程

![](https://gitee.com/sxhDrk/images/raw/master/imgs/LRU最久未使用策略.png)

1. 获取3个节点的属性

2. 使用其中一个能连接的节点收集集群数据：cluster nodes，application.properties给多个是为了保证一定能拿到一个能连接的节点，去获取整个集群的信息

3. 利用获取集群的信息包装2个map对象

   - nodes：

     - key：节点名称（ip：port）
     - value：该节点的连接池（JedisPool）

   - slots：

     - key：槽道号（slot）
     - value：槽道管理的节点连接池（JedisPool，通过nodes赋值实现）

     

     

### 方法调用高可用

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Redis淘汰策略.png)

1. 每次调用计算槽道号
2. 从slots中获取pool连接池
3. 调用连接资源，执行方法
4. 成功则结束
5. 不成功进去重试
   - 重试成功，结束方法
   - 重试不成功，进入重新连接构造集群信息逻辑
     - 重新构造，服务端顶替完成，数据变动，抛异常。下次调用就会执行成功
     - 重新构造，服务端没有顶替完成，数据不变，抛异常，下次调用依然进入重新构造集群信息逻辑，直到成功顶替

> 结论：JedisCluster利用槽道的结构，封装了代码高可用逻辑，只要redis-cluster服务端不崩溃，客户端代码就无须重启、重新配置





# 面试题

数据库2000万条数据，但是redis只能存储最大缓存容量200万条。如何保证200万条绝大部分是热点数据？热点数据--热门数据。例如:商品热门数据，小米10pro。。冷门数据相对访问频率不大，不太会需要存储在缓存。例如：诺基亚1110（考察的就是淘汰策略使用）

>  可以通过redis的配置实现 绝大部分数据存储的都是热点数据。
>
> 常用的LRU：最近最久未使用删除策略

redis中可以通过设置缓存内存容量，达到内存上限后，可以根据设置删除淘汰数据策略实现数据淘汰，==默认情况下，没有淘汰策略==，一旦达到内容上限，将会抛出错误。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/JedisCluster初始化流程.png)

> 没有设置maxmemory ，默认最大内存占用物理内存上限。一旦到达上限，系统不可用了。配合淘汰策略，达到内存上限进行数据删除。

# **所有的淘汰策略**

```
volatile-lru -> remove the key with an expire set using an LRU algorithm

allkeys-lru -> remove any key according to the LRU algorithm

volatile-random -> remove a random key with an expire set

allkeys-random -> remove a random key, any key

volatile-ttl -> remove the key with the nearest expire time (minor TTL)

noeviction -> don't expire at all, just return an error on write operations

```

> `volatile-lru`：对于已经设置了有超时时间的数据，利用LRU计算算法实现淘汰
>
> `allkeys-lru`：所有数据包括永久数据，采用lru算法淘汰
>
> `volatile-random`：对于设置了超时时间的数据，采用随机淘汰
>
> `allkeys-random`：所有数据随机淘汰
>
> `volatile-ttl`：对于设置了超时时间的数据，采用到达了内存上限之后，计算谁的剩余时间少先淘汰谁。
>
>  `noeviction`：没有淘汰策略  到达了上限，返回error

`LRU`：可以保证绝大部分数据在淘汰发生时，是热点数据（频繁访问）

![](https://gitee.com/sxhDrk/images/raw/master/imgs/JedisCluster高可用图解.png)

> 频繁访问的热点数据，绝大多数都会在达到内存上限时，不满足最近最久未使用。
>
> 【可以根据缓存存储的数据特点--超时，没超时，指定`volatile-lru`还是`allkeys-lru`】
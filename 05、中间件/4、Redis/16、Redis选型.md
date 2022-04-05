

[TOC]



> redis作为数据库/缓存的区别

# 缓存

作为缓存，Redis因为内存瓶颈的存在，所以最好只保留热数据

![image-20210307200030363](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210307200030363.png)

Redis的`Redis.conf`配置文件中，有相关配置，来规定Redis占用多大内存。最好是==1G-10G==，再大对以后迁移数据造成不方便

![image-20210307195718628](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210307195718628.png)



> redis中可以通过设置缓存内存容量，达到内存上限后，可以根据设置删除淘汰数据策略实现数据淘汰，==默认情况下，没有淘汰策略==，一旦达到内容上限，将会抛出错误。

当内存达到设定的内存大小时，可以通过淘汰策略进行冷数据的淘汰

`redis.conf`配置文件中同样有对应配置。通过`maxmemory-policy noeviction`来指定淘汰策略

```shell
#  Redis 5.0.x
# volatile-lru -> Evict using approximated LRU among the keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key among the ones with an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
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
> `noeviction`：没有淘汰策略  到达了上限，返回error

`LFU`：最多用到的key

`LRU`：可以保证绝大部分数据在淘汰发生时，是热点数据（频繁访问）— 最多没用到的key

![](https://gitee.com/sxhDrk/images/raw/master/imgs/LRU最久未使用策略.png)

## Redis中key的有效期

1. 对key设置有效时间，当对key进行get时，不会延长有效时间

2. 可以通过`EXPIREAT key timestamp `来指定固定时间，timestamp是Unix时间戳

   ```shell
   127.0.0.1:6379> set k1 name ex 50
   OK
   127.0.0.1:6379> EXPIREAT k1 20210307000000
   (integer) 1
   127.0.0.1:6379> TTL k1
   (integer) 20208691880325
   ```

3. 对key设置有效时间，当对key进行重新set时，会剔除有效时间，变为永久数据

==所以，key有效期每次操作都要设置，由业务逻辑决定==



## 过期判定原理

1. 被动访问时，判定是否过期
2. 周期轮询判定
   - 每次判定20个key，如果超过25%key失效，清除失效key，继续轮询。
   - 如果小于25%失效，清除key即可

**目的：稍微牺牲内存，但是保住了Redis性能为王的准则**







# 数据库

数据不能丢，不能设置淘汰策略，必须要持久化存储

Redis+MySQL的情况，中间添加消息队列，保证数据一致性
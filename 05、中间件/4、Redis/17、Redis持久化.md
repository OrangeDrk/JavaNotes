[TOC]

# 1、RDB

## 1.1、介绍

RDB持久化有时点性

​	在某一时刻，进行持久化，持久化文件中只有当前时刻的key-value数据。持久化过程中变更的数据不会在这次持久化文件中出现。

​	Redis通过创建子进程来进行持久化，另一个进程还可以继续处理Redis的操作

![image-20210308134801768](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210308134910016.png)

Redis配置文件`redis.conf`中有RDB的相关配置

1. 保存频率

   第一个参数：多长时间

   第二个参数：多少操作变化

   当60s内发生10000会持久化一次

   ![image-20210308134910016](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210308145954477.png)

2. 文件名和存放路径

   ![image-20210308134938056](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210308134801768.png)

## 1.2、触发持久化

save：客户端阻塞，服务端开始持久化。使用场景：关机维护，明确知道必须要保存数据

bgsave：客户端非阻塞，服务端fork创建子线程开始持久化

配置文件配置规则：配置save规则，这个save是bgsave

## 1.3、优缺点

> 缺点

1. 不支持拉链存储，从始至终只有一个dump.db文件，每次都会覆盖，区分版本需要人为来做
2. 丢失数据相对多一些：如果第一次持久化后，第二次持久化时挂机，这个时间段内的数据就丢失了

> 优点

类似Java中的序列化，恢复的速度相对要快





# 2、AOF

会把写操作记录到日志文件中，而且会进行覆盖（set k1 a,set k1 b，最后日志只有set k1 b）。

在第一次持久化时，会将内存数据以RDB的方式方式写出到aof文件中

然后每次写操作都会在aof文件中追加日志

![image-20210308145954477](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210308134938056.png)

追加有3种配置:

- always每次都追加
- everysec每秒追加一次
- no不追加



第一次`BGREWRITEAOF`时，会在aof文件中生成REDIS开头的内容，这是RDB数据

```shell
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
```

> aof文件

```shell
REDIS0009ú      redis-ver^E5.0.3ú
redis-bitsÀ@ú^EctimeÂ^XÓE`ú^Hused-memÂ¸^D^M^@ú^Laof-preambleÀ^Aþ^@û^B^@^@^Bk2^Ab^@^Bk1^AcÿWùtTüMÃÞ*2^M
```

因为配置的是每次都追加日志，所以每次写操作，aof中都会新增新的日志内容

```shell
127.0.0.1:6379> BGREWRITEAOF
Background append only file rewriting started
127.0.0.1:6379> keys *
1) "k2"
2) "k1"
127.0.0.1:6379> set k1 v
OK
```

> aof文件

```shell
REDIS0009ú      redis-ver^E5.0.3ú
redis-bitsÀ@ú^EctimeÂ^XÓE`ú^Hused-memÂ¸^D^M^@ú^Laof-preambleÀ^Aþ^@û^B^@^@^Bk2^Ab^@^Bk1^AcÿWùtTüMÃÞ*2^M
$6^M
SELECT^M
$1^M
0^M
*3^M
$3^M
set^M
$2^M
k1^M
$1^M
v^M
```


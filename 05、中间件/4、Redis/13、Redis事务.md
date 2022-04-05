[TOC]

Redis初衷是快速读取，所以事务不会特别的强调一致性

# 介绍

redis通过一个命令来开启一个事务

开启事务后，所发送的所有命令会放到一个队列中

当Redis收到执行事务的时候，按照顺序执行队列中的命令

> 注：Redis先收到谁发送的执行，就会先执行谁队列中的命令，这时就不保证其他队列的命令能正确执行

# 事务命令

![image-20210305162847636](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210305162847636.png)

## ① multi

开启一个事务

```shell
127.0.0.1:6379> MULTI  # 开启事务
OK
127.0.0.1:6379> keys *   # 发送命令
QUEUED
127.0.0.1:6379> set name sxh  # 发送命令
QUEUED
127.0.0.1:6379> EXEC  # 执行事务
1) 1) "unkey"
   2) "unkey1"
   3) "k1"
   4) "k2"
   5) "unkey2"
2) OK
```

## ②exec

执行事务

将事务队列中的命令按照顺序依次执行

```shell
127.0.0.1:6379> MULTI  # 开启事务
OK
127.0.0.1:6379> keys *   # 发送命令
QUEUED
127.0.0.1:6379> set name sxh  # 发送命令
QUEUED
127.0.0.1:6379> EXEC  # 执行事务
1) 1) "unkey"
   2) "unkey1"
   3) "k1"
   4) "k2"
   5) "unkey2"
2) OK
```

## ③ watch

对指定的key进行监控，当key被其他事务更改，该事务所有命令不在执行

> client1

```shell
127.0.0.1:6379> WATCH k1 # 对k1进行监听
OK
127.0.0.1:6379> MULTI  # 开启事务
OK
127.0.0.1:6379> get k1  # 发送命令，get k1
QUEUED

```

> 此时client1没有提交事务，client2开启了事务，操作了k1，并提交了事务
>
> client2

```shell
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 nnnnnnname
QUEUED
127.0.0.1:6379> EXEC
1) OK

```

> 此时client1才提交事务，因为提前监听了k1，此时发现k1被修改过，所以client1的所有命令都不会被执行了

```shell
127.0.0.1:6379> WATCH k1
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> get k1
QUEUED
127.0.0.1:6379> EXEC   # client1 最后提交的事务，因为k1被更改过，所以上面的命令都不执行
(nil)
```

# 为什么Redis不支持回滚（roll back）

以下是这种做法的优点：

- Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
- 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， 回滚并不能解决编程错误带来的问题。 举个例子， 如果你本来想通过 [INCR](http://redis.cn/commands/incr.html) 命令将键的值加上 1 ， 却不小心加上了 2 ， 又或者对错误类型的键执行了 [INCR](http://redis.cn/commands/incr.html) ， 回滚是没有办法处理这些情况的。
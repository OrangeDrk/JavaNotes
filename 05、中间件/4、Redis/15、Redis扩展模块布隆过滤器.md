[TOC]

> Redis可以通过module模块进行功能扩展

# 下载

在redis国外官网，可以从上方导航栏看到modules信息

![image-20210305171108362](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210305171108362.png)

从页面下方就能发现RedisBloom

![image-20210305171153424](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210305171153424.png)

点击小房子会进入github，进行下载

![image-20210305171305047](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210305171305047.png)



# Redis加载Bloom

因为Redis是C语言开发的，所以扩展模块也是C语言开发的，下载完需要解压，进入目录

执行`make`命令进行编译，会生成一个`.so`的扩展文件

![image-20210305171852556](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210305171852556.png)

然后启动redis的时候，用`--loadmodule path`来加载布隆过滤器，启动好redis后，即可在Redis中使用bloom的相关命令

> 加载Bloom一定要用全路径

```shell
redis-server ../redis-5.0.3/redis.conf --loadmodule /home/software/RedisBloom-master/redisbloom.so
```

> 也可以写在配置文件中，启动默认加载

![image-20210307195315068](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210305173621018.png)

# Bloom过滤器原理

![image-20210305173621018](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210307195315068.png)

预先将会存在的值通过3个不同的hash函数进行计算，得出哪个bit位是1

查询的参数同样进行3个不同的hash函数计算，得出3个bit位下标，看3个位置是不是都是1，如果是说明命中，可以进行数据查询。如果有一个为0，说明该参数不能进行查询

> 存在问题：当预设值越多，bit位中1的个数也就越多，就会出现即使要查询的参数不存在，那么也会出现出3个bit位都是1的情况（概率问题）
>
> 但是一定概率会减少放行（减少缓存穿透），而且成本低



# Redis命令

## ① bf.add

向布隆过滤器添加数据

```shell
127.0.0.1:6379> BF.ADD store apple
(integer) 1
```

## ② bf.exists

```shell
127.0.0.1:6379> BF.ADD store apple
(integer) 1
127.0.0.1:6379> BF.EXISTS store apple
(integer) 1
127.0.0.1:6379> BF.EXISTS store banana
(integer) 0
```

## ③ cf.count

> cuckoo 布谷鸟过滤器相关命令



# 扩展

除了Bloom过滤器

还有counting bloom、cuckoo（布谷鸟过滤器）
[TOC]

# 设置密码

## 持久化密码

> 配置文件中配置一个key-value 

```c
requirepass 123456
```

![](https://gitee.com/sxhDrk/images/raw/master/imgs/没有输入密码的Redis客户端.png)



## 临时密码

需要在本地客户端登录 ,通过`config set requirepass `设置密码。

重新启动的redis有没有密码取决于配置文件



## 客户端使用密码

> 使用客户端登录后，进行命令的发送，会提示没有提供操作密码

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Redis设置持久化密码.png)

> Linux客户端使用密码连接Redis
>
> -a : 指定密码
>
> -h : 指定ip

```shell
[root@sxh ~]# redis-cli -a sxh123+123
```



### Java客户端调用

> 所有客户端在操作redis之前需要先调用命令或者方法

```c
jedis auth(密码)
jedis.auth("123456");
```



### Linux中Redis客户端

```c
127.0.0.1:6380> auth 123456
OK
127.0.0.1:6380> set name wanglaoshi
OK
127.0.0.1:6380> get name
"wanglaoshi"
127.0.0.1:6380>
```



# ==企业的网络环境==

很多企业具备内部网络，这些网络中运行的redis服务器==可以省略提供密码==安全保证。
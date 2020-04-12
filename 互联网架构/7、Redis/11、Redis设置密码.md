[TOC]

# 设置密码

## 持久化密码

> 配置文件中配置一个key-value 

```c
requirepass 123456
```

![](https://note.youdao.com/yws/api/personal/file/3558A9FEDC7241F3A0D696AB3B477530?method=download&shareKey=3feb265f1347c137945b2972ac67f5ad)



## 临时密码

需要在本地客户端登录 ,通过`config set requirepass `设置密码。

重新启动的redis有没有密码取决于配置文件



## 客户端使用密码

> 使用客户端登录后，进行命令的发送，会提示没有提供操作密码

![](https://note.youdao.com/yws/api/personal/file/C56042F4E0334974BDE5A6B22CE764CF?method=download&shareKey=5a5993ff2db1e9243fbd41854b1adddd)



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
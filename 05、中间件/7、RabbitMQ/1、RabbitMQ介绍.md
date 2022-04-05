[TOC]

# 消息队列历史

> 早期消息队列是解决通信解耦问题的

A、B两个系统，进行数据通信，A要将数据传递给B实现计算，B要将计算结果返回给A使用

![](https://gitee.com/sxhDrk/images/raw/master/imgs/早期通信形式.png)

无论A是传递数据，还是B返回计算结果，有任何一步失败，都会导致整个通信流程重新来。引入消息队列后将有效改善这个问题

![](https://gitee.com/sxhDrk/images/raw/master/imgs/长连接和短连接.png)



# RabbitMQ

rabbitmq是企业级的消息队列。它实现了SUN公司提出的AMQP（advanced message queue protocol）协议，是一个高级的消息队列

> 消息队列产品：
>
> - rabbitmq
> - rocketmq
> - kafka
> - JMS：性能比较低



# rabbitmq的结构



## 客户端角色

rabbitmq支持很多语言，可以连接rabbitmq成为客户端，角色不同取决于客户端使用rabbitmq实现的功能

1. ==生产端==：发送消息的客户端
2. ==消费端==：连接消息队列，接收消费者发送的消息的客户端



## 核心组件

> 连接组件：长连接，短连接

![](https://gitee.com/sxhDrk/images/raw/master/imgs/rabbitmq和erlang语言版本的对应.png)

### 长连接

长连接基于TCP/IP协议，创建长连接比较消耗资源，所以不能频繁创建、销毁。

==一般一个进程都会绑定一个长连接去使用==



### 短连接

短连接基于长连接，消耗资源非常有限。客户端使用完后可以关闭、销毁



### 交换机：`exchange`

rabbitmq要在内存中保存大量的队列，交换机完成对发送过来的消息的高并发处理。

并发稳定性取决于客户端代码

exchange组件是使用erlang语言实现的

![](https://gitee.com/sxhDrk/images/raw/master/imgs/早期通信引入消息队列.png)



### 队列：`queue`

是rabbitmq存储消息的组件，可以在内存中存储，也可以实现持久化。

队列从生产端发送到的交换机中获取消息

消费端绑定监听队列，实现消费



# 安装和启动

## 安装

> rabbitmq的exchange组件是erlang语言开发的，安装rabbitmq之前，要安装对应高版本的erlang环境

![](https://gitee.com/sxhDrk/images/raw/master/imgs/交换机类型.png)

```sh
[root@10-9-104-184 resources]# rpm -ivh rabbitmq-server-3.7.7-1.el6.noarch.rpm 
```

> rpm安装后，安装文件在`/usr/lib/rabbitmq/`中



## 启动

> rabbitmq默认用户：guest；密码：guest
>
> 默认只允许本地登录使用这个用户



### 调整环境配置

1. 开启web控制台插件

   > rabbitmq提供了一个web控制台，需要在启动rabbitmq之前开启控制台插件

   ```sh
   [root@10-9-104-184 bin]# rabbitmq-plugins enable rabbitmq_management
   ```

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/开启user的权限.png)

   

2. 将用户权限打开

   > rabbitmq有一个模板配置文件
   >
   > `/usr/share/doc/rabbitmq-server-3.7.7/rabbitmq.config.example`
   >
   > 将这个文件拷贝到`/etc/rabbitmq`，并重命名`rabbitmq.config`

   ```sh
   [root@10-9-104-184 rabbitmq-server-3.7.7]# cp 
   	/usr/share/doc/rabbitmq-server-3.7.7/rabbitmq.config.example 
   	/etc/rabbitmq/rabbitmq.config
   ```

   > 修改`/etc/rabbitm1/rabbitmq.config`配置文件的61行，将user限制打开，参数为空，表示没有限制

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/通过rabbitmq的命令脚本启动服务.png)



### 启动

> 方式一：通过服务的方式启动
>
> - 打开：start
> - 关闭：stop
> - 重启：restart

```sh
[root@10-9-104-184 resources]# service rabbitmq-server start 
```



> 方式二：通过命令脚本启动
>
> - 启动命令后加`&`号，后台运行，通过ps查看PID，使用kill关闭
> - 直接启动，CTRL+C关闭

先进入rabbitmq根目录的bin文件夹

`/usr/lib/rabbitmq/bin`

```sh
[root@10-9-104-184 bin]# ./rabbitmq-server start
[root@10-9-104-184 bin]# ./rabbitmq-server -detached
```

![](https://gitee.com/sxhDrk/images/raw/master/imgs/开启web控制台插件.png)



### 登录web控制台页面

> rabbitmq服务成功启动后，通过浏览器：`服务器ip:15672`访问rabbitmq的控制台页面

![](https://gitee.com/sxhDrk/images/raw/master/imgs/rabbitmq的控制台页面.png)
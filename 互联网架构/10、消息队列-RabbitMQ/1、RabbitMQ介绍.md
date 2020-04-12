[TOC]

# 消息队列历史

> 早期消息队列是解决通信解耦问题的

A、B两个系统，进行数据通信，A要将数据传递给B实现计算，B要将计算结果返回给A使用

![](https://note.youdao.com/yws/api/personal/file/97753B5BC9514F10B789258DD501DD66?method=download&shareKey=8756de7c20a88be00e23a0e1f68580e3)

无论A是传递数据，还是B返回计算结果，有任何一步失败，都会导致整个通信流程重新来。引入消息队列后将有效改善这个问题

![](https://note.youdao.com/yws/api/personal/file/357FAE59404D40748E68008F41597BD6?method=download&shareKey=966ca153aa9e3638a6ea4560a79f5b54)



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

![](https://note.youdao.com/yws/api/personal/file/A7671D5D82CE438AB7CCB636BF2AD800?method=download&shareKey=1668abd40b50783598207f62f0644ca3)

### 长连接

长连接基于TCP/IP协议，创建长连接比较消耗资源，所以不能频繁创建、销毁。

==一般一个进程都会绑定一个长连接去使用==



### 短连接

短连接基于长连接，消耗资源非常有限。客户端使用完后可以关闭、销毁



### 交换机：`exchange`

rabbitmq要在内存中保存大量的队列，交换机完成对发送过来的消息的高并发处理。

并发稳定性取决于客户端代码

exchange组件是使用erlang语言实现的

![](https://note.youdao.com/yws/api/personal/file/F4B32D7440F14B22AFC0BDD0755331D4?method=download&shareKey=955620469b9d82a740c219763d0f1d63)



### 队列：`queue`

是rabbitmq存储消息的组件，可以在内存中存储，也可以实现持久化。

队列从生产端发送到的交换机中获取消息

消费端绑定监听队列，实现消费



# 安装和启动

## 安装

> rabbitmq的exchange组件是erlang语言开发的，安装rabbitmq之前，要安装对应高版本的erlang环境

![](https://note.youdao.com/yws/api/personal/file/5D1A302CFC29438890C4F71C22F3CF5C?method=download&shareKey=2daeb5f5d40dedae21746353d7760e02)

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

   ![](https://note.youdao.com/yws/api/personal/file/F9839F4647B84CA190E371D930ADE8C3?method=download&shareKey=3e97812447c40b4fa5686447745e5bc3)

   

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

   ![](https://note.youdao.com/yws/api/personal/file/35082F031BC042EBB6758615EA850115?method=download&shareKey=d7393965f48eb9a768bd5855c743c567)



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

![](https://note.youdao.com/yws/api/personal/file/232E2516AB684ADEBD877278DD2DD98A?method=download&shareKey=aa17939c7161c70bf1957a1e09dc96b9)



### 登录web控制台页面

> rabbitmq服务成功启动后，通过浏览器：`服务器ip:15672`访问rabbitmq的控制台页面

![](https://note.youdao.com/yws/api/personal/file/74D05FD2C1FF4904B9C8AC6B64760FA5?method=download&shareKey=6d67403006ccbeda7675284dcdb51884)
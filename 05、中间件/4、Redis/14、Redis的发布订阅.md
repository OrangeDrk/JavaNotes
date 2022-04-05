[TOC]



> 类似于聊天室，所有人能看到聊天内容



# 发布

命令：`PUBLISH channel message`

向一个通道中推送一个消息

```shell
127.0.0.1:6379> PUBLISH sxh hello
(integer) 1
```



# 订阅

命令：`SUBSCRIBE channel [channel ...]`

监听一个通道里的消息

> 只能收到开始监听之后的消息，之前的消息无法收到

```shell
127.0.0.1:6379> SUBSCRIBE sxh 
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "sxh"
3) (integer) 1
1) "message"
2) "sxh"
3)
```


[TOC]

# 下载

http://nginx.org/en/download.html



# 安装

将下载好的安装包解压到一个没有中文没有空格的目录下即可。



# 命令

> 进入Nginx根目录，使用cmd窗口运行

1. 验证配置是否正确: `nginx -t`
2. 查看Nginx的版本号：`nginx -V`
3. 启动Nginx：`start nginx`
4. 快速停止或关闭Nginx：`nginx -s stop`
5. 正常停止或关闭Nginx：`nginx -s quit`
6. 配置文件修改重装载命令：`nginx -s reload`

> nginx是否在运行可以在windows的任务管理器中查看，正在运行ngxin会由两个进程显示。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx启动后.png)



# 配置

> nginx的工作是基于[`conf/nginx.conf`]配置文件来进行的。

`nginx.conf`的配置结构：

```shell
http{ #代表处理http请求

      #配置一个虚拟服务器
      server{
      
      	#此虚拟服务器接收对80端口的访问
      	listen 80; 
      	
     	#此虚拟服务器接收对localhost主机名的访问
      	server_name localhost; 
      	
      	#当访问/user资源时由此配置处理
      	location /user{
      		规则
      	}
      	
      	#当访问/order资源时由此配置处理
      	location /order{
      		规则
      	}
      	...
      }

      #其他Server配置
      server ...
      ...
}

```


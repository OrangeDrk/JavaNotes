[TOC]

# 入门案例：通过Nginx实现请求转发

1. 当客户端访问http://www.pq.com时，由nginx转发给http://127.0.0.1:8080端口进行处理

   - 配置hosts文件：`127.0.0.1 www.pq.com`

2. 在nginx.conf中配置

   ```sh
   http{
         #为nginx配置一个虚拟服务器,
         server {
             #监听本机80端口
             listen 80;
             #接收对www.pq.com主机名的访问
             server_name www.pq.com;
             #对/即任意路径的访问进行处理
             location / {
             	#转发到指定地址
             	proxy_pass http://127.0.0.1:8080;
             } 
             #可以配置多个location
             ...
         }
         #可以配置多个server
         ...
   }
   
   ```

   

3. 启动tomcat：`startup.bat`

4. 启动nginx：`start nginx`

5. 通过浏览器访问地址：`www.pq.com`,发现确实可以跳转到指定页面

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx实现请求转发.png)

# `location`路径配置和匹配规则

## `location`路径的写法

在配置虚拟服务器时，可以配置多个location，指定不同路径采用不同的处理方案，location支持多种写法，规则如下：

| **规则** | **实例**      | **作用**                                 |
| -------- | ------------- | ---------------------------------------- |
| `=`      | `=/aaa/1.jpg` | 路径严格匹配，路径必须一模一样才会匹配到 |
| `^~`     | `^~/aaa`      | 只要是指定路径开头的路径都可以匹配       |
| `~`      | `~.png$`      | 区分大小写按正则匹配路径                 |
| `~*`     | `~*.png$`     | 不区分大小写按正则匹配路径               |
| `/`      | `/`           | 通用匹配，所有路径都可以匹配到           |



## `location`路径配置的优先级

由于location的路径配置非常灵活，所有有可能一个路径被多个location所匹配，此时按照如下规则判断匹配优先级：

- 首先匹配 `=`
- 其次匹配 `^~`
- 其次是按文件中顺序的==正则==匹配
- 最后是交给 `/` 通用匹配
- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

==**总的规律是，精度越高优先级越高**==



> 案例

```shell
location = / {
   #规则A
}

location = /login {
   #规则B
}

location ^~ /static/ {
   #规则C
}

location ~ \.(gif|jpg|png|js|css)$ {
   #规则D
}

location ~* \.png$ {
   #规则E
}

location / {
   #规则F
}
```

> 结果：
>
> - 访问根目录 /， 比如 http://localhost/ 将匹配规则 A
> - 访问 http://localhost/login 将匹配规则 B
> - http://localhost/register 则匹配规则 F
>
> -  访问 http://localhost/static/a.html 将匹配规则 C
> -  访问 http://localhost/a.png, http://localhost/b.png 将匹配规则 D和规则 E，但是规则 D 顺序优先，规则 E不起作用
> - http://localhost/static/c.png则优先匹配到规则 C
> - 访问 http://localhost/a.PNG 则匹配规则 E，而不会匹配规则 D，因为规则 E 不区分大小写
> -   访问 http://localhost/category/id/1111 则最终匹配到规则 F





# Nginx的负载均衡实现

## 负载均衡原理

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx负载均衡原理.png)

## 实现案例

### 配置三台tomcat

> 分别监听不同端口，并启动tomcat

1. tomcat1
   - 第22行：8015
   - 第69行：8081
   - 第91行：8019
2. tomcat2
   - 第22行：8025
   - 第69行：8082
   - 第91行：8029
3. tomcat3
   - 第22行：8035
   - 第69行：8083
   - 第91行：8039

## 修改`nginx.conf`配置，启动nginx

```shell
#upstream是nginx配置文件中的关键字，用来配置一组服务器地址供后续使用
upstream big1907{
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
    server 127.0.0.1:8083;
}

server {
    listen 80; #对80端口的访问
    server_name www.pq.com; #对此主机名的访问
    location / {
    	#转发到上面配置的服务器组
    	proxy_pass http://big1907;
    }
}
```



## 测试

> 经过测试，发现确实可以通过nginx负载均衡访问到tomcat中的资源

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx负载均衡1.png)

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx负载均衡2.png)

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx的动静分离原理.png)



# 负载均衡策略

> nginx在分发资源到后端服务器时，如何分配请求是可以配置的，称之为nginx的负载均衡策略

## 轮询

**配置**：默认不配置就是轮询

**作用**：连接请求轮流分配给后端服务器

```shell
http{ 
    upstream sampleapp { 
        server ip:端口;
        server ip:端口;
    } 
    .... 
    server{ 
        listen 80; 
        ... 
        location / { 
            proxy_pass http://sampleapp; 
        }  
	}
} 
```



## ip哈希

**配置**：`ip_hash;`

**作用**：`abs(客户端ip.hash())%服务器数量`，（abs：绝对值）根据余数决定连接请求去往哪个服务器

```cobol
http{ 
    upstream sampleapp { 
      ip_hash;
      server ip:端口;
      server ip:端口;
    } 
    .... 
    server{ 
      listen 80; 
      ... 
      location / {
          proxy_pass http://sampleapp; 
      }  
	}
} 
```



## 最少链接

**配置**：`least_conn;`

**作用**：将连接请求分配给目前连接数最少的服务器

```cobol
http{ 
    upstream sampleapp { 
      least_conn;
      server ip:端口;
      server ip:端口;
    } 
    .... 
    server{ 
      listen 80; 
      ... 
      location / {
          proxy_pass http://sampleapp; 
      }  
	}
} 
```



## 基于权重

**配置**：直接在地址后配置`weight=num`

**作用**：

- 根据权重进行分配，权重值越大，被分配的连接越多。
- 可以直接配置为`down`，则不再分配连接。

```shell
http{ 
    upstream sampleapp { 
      server ip:端口 weight=1;
      server ip:端口 weight=3;
      server ip:端口 weight=6;
    } 
    .... 
    server{ 
      listen 80; 
      ... 
      location / {
          proxy_pass http://sampleapp; 
      }  
	}
} 
```



# Nginx的动静分离实现

## 动静分离原理

动 --> 动态资源 --> servlet jsp --> 程序 

静 --> 静态资源 --> jpg mp3 mp4 html css js --> 文件 

> tomcat能够处理动态和静态资源，但本质上是为处理动态资源而设计的服务器，过多静态资源交由tomcat管理会降低tomcat处理动态资源的能力，得不偿失。

nginx本身无法处理动态资源，但可以处理静态资源，而且性能优良。

因此可以将静态资源和动态资源拆分，将静态资源交由ngin处理，动态资源仍由tomcat处理，从而解放了tomcat对动态资源的处理能力，整体上实现动静分离，提升了效率。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx负载均衡3.png)



## 动静分离实现

配置方式

```shell
server {
    listen 80;
    server_name www.staticfile.com;
    location ~*\.(html|css|js|jpg|png)$ {
        #root可以指向nginx服务器中的本地磁盘地址
        #静态文件就放置在这个磁盘地址中
        #之后对server中资源的访问会被转换到对本地磁盘资源的访问
        #www.staticfile.com/aaa/bbb/1.html-->d://html/aaa/bbb/1.html
        root D://html;
        #默认访问的首页配置
        index index.html;
    } 
}
```



## 案例

> 对http://www.pq.com/html路径及其子路径的访问会被转到对磁盘d://html文件夹的静态资源的访问。

```shell
server {
    listen 80;
    server_name www.pq.com;
    location ^~ /html {
        root d://;
        index index.html;
    }
}
```



# 综合案例：用户订单的Nginx动静分离，负载均衡

1. 将用户订单积分案例的静态资源存储到磁盘

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/Nginx动静分离的静态资源位置.png)

2. 将项目打成war包，发布到每一个tomcat

   - 通过maven打war包：`mvn clean`  `mvn package`

   - 放置到tomcat1、tomcat2、tomcat3的`webapps`目录下，自动解压为web应用

3. 配置Nginx

   ```shell
   #练习案例 用户积分案例
   upstream userorder{
       server 127.0.0.1:8081;
       server 127.0.0.1:8082;
       server 127.0.0.1:8083;
   }
   server {
       listen 80;
       server_name www.pq.com;
       location ~* \.(html|css|js|jpg|png)$ {#动静分离
           root d://static;
           index index.html;
       }
       location / {#负载均衡
       	proxy_pass http://userorder;
       }
   }
   ```

   
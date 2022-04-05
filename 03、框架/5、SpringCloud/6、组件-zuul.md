[TOC]

# zuul网关组件

> 使用eureka和ribbon可以实现一个复杂的，多种多样的微服务集群的搭建，并且可以实现一个功能调用过程中，复杂的调用链，但是并不能解决外接调用（因为前端没有SpringCloud技术）
>
> 引入网关组件`zuul`，是微服务集群唯一对外访问的入口工程
> 

## zuul网关的实现功能

- 路由：zuul接收到外界请求后，根据请求地址url判断该请求访问后端哪个微服务
- 过滤：不是所有的请求都是合法请求，在过滤过程中，可以根据校验逻辑，判断哪些请求来到网关可以向后调用微服务，不合法的就拒绝访问



# 搭建zuul网关系统

## 创建maven工程

选择quickstart骨架创建工程

> 没有controller、service等功能，只作为网关转发作用

## 修改pom.xml配置

1. 导入SpringCloud依赖
2. ==引入zuul组件==
3. 添加eureka-client依赖
4. 添加ribbon依赖

```xml
<dependencies>
    <!--eureka client-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>

    <!--ribbon依赖-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-ribbon</artifactId>
    </dependency>
    
    <!--zuul组件-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zuul</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <!--实现使用SpringCloud所有依赖的声明式导入-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Edgware.RELEASE</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
```



## 配置application.properties文件

1. 端口使用：8103

2. 服务名称：`gateway`，用户在注册中心注册

3. 注册中心指向：`8761`

4. 设置ip优先

5. 配置`路由规则/微服务配置`

   ```properties
   zuul.routes.api-a.path=/zuul-a/**
   zuul.routes.api-a.service-id=server-hi
   ```

   - `zuul.routes`：路由属性前缀，固定写法
   - `api-a`：该路由的一个路由名称（代号），一般都会使用业务功能起名。例如：order、user、cart、product等等
   - `path`：表示到达zuul网关的访问地址==匹配逻辑==
   - `service-id`：表示，同一个路由名称，只要满足了`path`匹配，就会调用后端服务

   > 上述这一对路由匹配：==在zuul当中，自定义了一个路由规则，叫做`api-a`，凡是到达`zuul`网关的请求，`url`满足`path`路径匹配，就会被zuul调用`server-hi`来处理==

   - `path`匹配规则满足`ANT`规范

     | **匹配规则** | **解释**                                                     |
     | ------------ | ------------------------------------------------------------ |
     | `？`         | 表示一个字符。`/zuul-a/?`<br />可以匹配到的路径：`/zuul-a/a`、`/zuul-a/b`<br />匹配不到的路径：`/zuul-a/abc` |
     | `*`          | 表示一个字符串。`/zuul-a/*`<br />可以匹配到的路径：`/zuul-a/a`、`/zuul-a/abc`<br />匹配不到多级路径：`/zuul-a/a/b/c` |
     | `**`         | 表示任意多级路径，任意字符串长度。`/zuul-a/**`<br />可以匹配到所有以`/zuul-a/`开头的url地址 |

   

```properties
#端口8103
server.port=8103
spring.application.name=gateway

#指向注册中心的地址，实现注册发现
eureka.client.service-url.defaultZone=http://127.0.0.1:8761/eureka
#ip优先
eureka.instance.prefer-ip-address=true


#配置路由规则
#server-hi server-ribbon做测试
#routes：路由属性前缀 固定
#该路由的一个路由名称（代号）
zuul.routes.api-a.path=/zuul-a/**
zuul.routes.api-a.service-id=server-hi

```



## 启动类

使用注解：

`@SpringBootApplication`：本身是一个springboot项目

`@EnableEurekaClient`：是一个eureka客户端

`@EnableZuulProxy`：==启动网关代理==

```java
@SpringBootApplication
@EnableEurekaClient
@EnableZuulProxy
public class StartGateway {
    public static void main(String[] args) {
        SpringApplication.run(StartGateway.class, args);
    }
}
```



## 访问网关

### 访问zuul理解路由匹配规则，理解流转逻辑

1. 起始访问zuul网关：http://localhost:8103/zuul-a/client/hello
   - 到达网关，接收到url域名端口，剩余`/zuul-a/client/hello`
   - 做路由的path匹配，`/zuul-a/**`去掉匹配规则字符串`/zuul-a/`，剩余`/client/hell`，拼接到`server-hi`服务名称后面
   - 最后得到的请求：http://server-hi/client/hello，再通过ribbon逻辑进行负载均衡
2. zuul网关中转发调用后端服务使用的是ribbon组件
   - 拦截请求：解析服务名称：`server-hi`
   - 根据抓取的服务信息，server-hi，拿到服务列表：9001/9002做负载均衡
   - 地址：http://127.0.0.1:9001/client/hello



### 实现第二个路由规则

> path满足：`/zuul-b/**`
>
> 路由名称：`api-b`
>
> 访问到server-ribbon：`/ribbon/hello`

```properties
zuul.routes.api-b.path=/zuul-b/**
zuul.routes.api-b.service-id=server-ribbon
```

访问URL：http://loaclhost:8103/zuul-b/ribbon/hello?name=王

通过网关后：http://server-ribbon/ribbon/hello?name=王

负载均衡后：http://127.0.0.1:9004/ribbon/hello?name=王



## 路由匹配相关的问题

如果路由匹配的path规则具备包含关系

- `a.path=/zuul-a/*`
- `b.path=/zuul-a/**`

当请求地址为：http://localhost:8103/zuul-a/haha时，到底将给a路由处理，还是交给b路由处理？？？

> 当程序加载路由规则时，根据加载的顺序，判断优先级高低
>
> 先加载的配置在内存中的key-value数据位置靠上，优先级就高
>
> - properties：加载到内存，没有根据先后上下顺序去加载，底层是无序map：`HashMap`
> - yml：格式配置在上的属性，可以先加载到内存，底层是有序map：`LinkedHashMap`
>
> 所以企业中常用的配置文件格式为：`yml`





# order-user系统整合zuul网关

> 在网关中配置不同的请求地址，访问不同服务的路由规则
>
> order规则、user规则
>
> ==nginx访问发送出来的请求地址满足zuul网关定义的path路径==

## 配置网关规则

1. order路由规则

   ```properties
   zuul.routes.t-order.path=/zuul-o/**
   zuul.routes.t-order.service-id=test-order
   ```

   

2. user路由规则

   ```properties
   zuul.routes.t-user.path=/zuul-u/**
   zuul.routes.t-user.service-id=test-user
   ```

   

## nginx配置

1. js发起的请求地址

   > http://www.pq.com/orderusers/order/pay.action

   - ip映射找到了nginx，进入nginx
   - nginx匹配端口域名，剩余`/orderusers/order/pay.action`
   - 匹配规则`location ^~ /orderusers/order`，剩余`/pay.action`
   - 转发到：`proxy_pass http://loaclhost:8103/zuul-o/order;`

2. zuul网关接收到请求地址

   > http://localhost:8103/zuul-o/order/pay.action

   - 匹配`/zuul-o/**`的path规则，剩余：`/order/pay.action`

   - 找到路由规则：`t-order`，拼接`test-order`服务

   - 拼接后的URL：`http://test-order/order/pay.action`

   - 通过负载均衡后，可能访问到的地址为：

     http://127.0.0.1:8091/order/pay.action



​	所以说，后端某一个微服务的具体访问路径，经过nginx匹配过滤，经过zuul网关匹配，可以做到完全对外界是屏蔽的。要想调用服务功能必须通过nginx，必须通过网关



**Nginx完整的配置**

```
server {
	listen 80;
	server_name www.pq.com;
	#静态文件访问
	location / {
		root webapp;
		index index.html;
	}
	location ^~ /orderusers/order {
		proxy_pass http://localhost:8103/zuul-o/order/;
	}
	localhost ^~ /orderusers/user {
		proxy_pass http://localhost:8103/zuul-u/user/;
	}
}
```



## 完整结构图解

![](https://gitee.com/sxhDrk/images/raw/master/imgs/order-user整合zuul和nginx.png)





# zuul网关敏感头

> zuul网关会对cookie默认进行过滤，认为cookie等头信息是不安全的 。==默认删掉了==
>
> - request的cookie到达不了服务器
> - 服务器response的cookie也到达不了浏览器

**需要在zuul网关中关闭敏感头**

> `=`后面什么也不写，代表对任何信息都不敏感
>
> 如果需要对某一信息敏感，在`=`后添加即可

```properties
#关闭敏感头
zuul.sensitive-headers=
```





# zuul网关常见问题

> Forwarding error
>
> load balancer dose not have avalibal service：服务名称

**原因**：

1. zuul启动时先要去抓取注册中心的所有微服务，根据路由调用某个微服务。一旦zuul启动顺序在某个微服务之后去启动，启动时无法抓取到该服务信息，前面调用到达网关之后，找不到该微服务。==可以等一段时间==
2. 路由配置内容错误的。例如：
   - 服务名称：`service-ribbon` 
   - 路由规则写成了：`service-ribon`
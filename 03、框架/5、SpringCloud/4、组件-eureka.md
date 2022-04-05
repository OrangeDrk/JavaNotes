[TOC]



> `eureka`：服务治理组件
>
> [juˈriːkə] ：you'rui'ka

# 概括

1. 中文原意：==我发现了，我找到了==

2. eureka是SpringCloud支持的微服务框架中实现服务治理的核心组件。

3. 服务治理就是管理微服务集群中的所有微服务

   > 例如：order-user系统拆分后的order和user就可以在这个组件中实现管理结构

# eureka的实现逻辑

1. eureka实现了2个核心的功能：

   - 一个是==注册==
   - 一个是==发现==

2. eureka的角色

   - **eureka服务端**：提供管理服务的功能
     - 拆分后的每一个==服务提供者==（小系统），都要==注册在eureka服务端==
   - **eureka客户端**：就是拆分之后的每一个服务提供者
     - 所有eureka客户端都是==服务提供者==（拆分之后被调用的服务==实例==，也就是拆分之后的每一个小系统）
     - 可以根据需求，==发现==所有注册信息

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112506.png)



# 使用

## 搭建注册中心

> 配置SpringCloud项目环境

1. 选择quickstart骨架创建项目

2. 修改pom.xml配置文件

   - 通过继承springboot-parent标准的springboot环境（SpringCloud必须依赖SpringBoot搭建）

     - 服务端和客户端同时作同一个父级工程的子工程就可以同时继承springboot环境

     ```
     parent
     |
     |--eureka_server
     |--eureka_client
     ```

     

   - 依赖管理引入SpringCloud

     > 因为继承了springboot-parent，只能单一继承，所以SpringCloud的依赖需要使用`<dependencyManagement>`标签导入依赖
     >
     > 继承的springboot为1.5.9，所以springcloud版本需要使用：`Edgware`
     >
     > `<scope>`标注是导入依赖：import
     >
     > 类型为：`pom`

     ```xml
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

     

   - 依赖eureka服务端资源

     > eureka需要在`<dependencies>`标签中添加依赖
     >
     > - eureka服务端：`spring-cloud-starter-eureka-server`
     >
     >- eureka客户端：`spring-cloud-starter-eureka`
     
     ```xml
     <dependencies>
         <!--添加eureka服务端依赖-->
         <dependency>
             <groupId>org.springframework.cloud</groupId>
             <artifactId>spring-cloud-starter-eureka-server</artifactId>
         </dependency>
     </dependencies>
     ```
     
     

3. 准备配置文件：`application.properties`

   - 端口：8761
   - 配置eureka注册特性
     - 关闭注册
     - 关闭发现
   - 提供一个注册中心的连接地址
   - 让当前工程在eureka治理组件的底层使用ip记录注册信息

   ```properties
   #端口 8761
   server.port=8761
   
   #eureka 客户端配置 默认情况下，服务端进程包含一个客户端
   #关闭注册
   eureka.client.fetch-registry=false
   #关闭发现
   eureka.client.register-with-eureka=false
   
   #底层优先使用ip通信
   eureka.instance.prefer-ip-address=true
   
   #提供注册中心访问的具体地址
   eureka.client.service-url.defaultZone=http://127.0.0.1:8761/eureka
   ```

4. 配置启动类:`@EnableEurekaServer`

   ```java
   @SpringBootApplication
   /*
    *	eureka server开启注解，
    *	可以启动当前工程时，利用已经配置好的资源，启动内部的eureka-server的进程
    */
   @EnableEurekaServer
   public class StartEs01 {
       public static void main(String[] args) {
           SpringApplication.run(StartEs01.class, args);
       }
   }
   ```

   

## 访问注册中心控制页面

> 全部配置正确后，启动注册中心项目，
>
> 通过网址：http://localhost:8761访问注册中心控制台页面，观察当前注册中心状态

看到如下内容，显示当前注册在eureka中是实例信息（某个服务集群的一员）

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112507.png)

## 搭建客户端

> 使用客户端资源`eureka client`搭建一个工程
>
> 以`service-hi`服务名称注册在`eureka server`端

1. 选择quickstart骨架创建项目

2. 修改pom.xml配置文件

   - 继承springboot-parent标准的springboot环境（SpringCloud必须依赖SpringBoot搭建）

     - 服务端和客户端同时作同一个父级工程的子工程就可以同时继承springboot环境

       ```
       parent
       |
       |--eureka_server
       |--eureka_client
       ```

       

   - 导入SpringCloud依赖

     ```xml
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

     

   - 依赖eureka客户端资源

     ```xml
     <dependencies>
         <!--eureka客户端依赖-->
         <dependency>
             <groupId>org.springframework.cloud</groupId>
             <artifactId>spring-cloud-starter-eureka</artifactId>
         </dependency>
     </dependencies>
     ```

3. 准备配置文件：`application.properties`

   - 端口：9001
   - 配置服务名称，以便在注册中心注册
   - 配置eureka注册特性
     - 开启注册：==默认开启==
     - 开启发现：==默认开启==
   - 让当前工程在eureka治理组件的底层使用ip记录注册信息
   - 指向注册中心地址，实现注册发现

   ```properties
   #端口9001
   server.port=9001
   
   #提供该功能的服务名称，注册在注册中心 serviceId
   spring.application.name=server-hi
   
   #开启注册，开启抓取的功能:默认开启
   #开启IP地址使用优先
   eureka.instance.prefer-ip-address=true
   
   #指向注册中心的地址，实现注册发现
   eureka.client.service-url.defaultZone=http://127.0.0.1:8761/eureka
   
   ```

4. 配置启动类:`@EnableEurekaClient`

   ```java
   @SpringBootApplication
   //开启eureka客户端进程注解
   @EnableEurekaClient
   public class StartEC01 {
       public static void main(String[] args) {
           SpringApplication.run(StartEC01.class, args);
       }
   }
   ```

5. 简单写一个controller功能

   ```java
   @RestController
   public class HelloController {
       //模仿该客户端工程的具体功能
       @RequestMapping("/client/hello")
       public String sayHello(String name){
           return "hello "+name+", i am from 9001";
           //通过端口号的标识，区分一下服务被调用时的响应来源
       }
   }
   ```

6. 再开发一个eureka客户端：`client02`

   - 可以复制第一个已经创建好多的`client01`客户端

   - 复制后需要修改pom.xml文件

     ```xml
     <artifactId>spring-cloud-eureka-client02</artifactId>
     <name>spring-cloud-eureka-client02</name>
     ```

   - 在父级工程添加`<module>`，添加一个指向复制的新工程

   - 修改`client02`中的代码

     - 启动类
     - controller返回值。。
     - application.properties中的端口号9002

7. 启动两个eureka客户端，查看注册中心控制中

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112508.png)

   - 区别于nginx.conf中的upstream
     - upstream是静态文件
     - 注册信息是动态数据

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112509.png)





# eureka治理组件的管理逻辑

> eureka治理组件，不同角色的详细功能

## 服务端

### 接收请求

客户端访问：http://loaclhost:8761/eureka，接收客户端参数，实现管理客户端信息的功能

服务器端会在底层创建当前注册中心所有服务注册信息的数据结构：==双层Map==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112510.png)

> 1. 每个服务的提供者，作为eureka治理组件的客户端进程，携带自身的详细信息，访问注册中心的接口地址：/eureka
> 2. 注册中心接受到请求，开始在内存的map对象存储双层map的数据
>    - 第一层map的key值：服务器名称
>    - 第一层map的value值：第二层map对象
>      - 第二层map的key值：服务提供者的实例名称，默认结构：（系统名称：服务名称：端口号）
>      - 第二层map的value值：该实例的详细信息：（ip,port,时间戳数据等等）

双层map

```java
Map<String,String> map2=new HashMap<String,String>();

Map<String,Map<String,String>> map1=new HashMap<String,Map<String,String>>();

map2.put("实例1","10.9.9.9…");表示map2里保存了一个实例对象，有一个eureka client启动注册

map1.put("service-hi",map2);
```



### 定时判断失效的客户端

服务端根据客户端发送的==心跳检测==的最后的时间戳，每隔60秒执行一次定时任务，判断双层map中的第二层map里携带的时间戳的值。

如果检测时，时间戳超过90秒，就会认为client客户端节点失效了，就会从双层map记录的数据中剔除这个节点



### 服务端保护模式

> 正常情况下，服务提供者（服务实例）是非常庞大的集群，按照故障概率，应该经常性的有一部分机器故障导致无法满足心跳检测的要求，将会被剔除。
>
> 但是异常情况下，例如网络波动，导致心跳检测的服务实例大量达到超时时间条件，在这种情况下，不能将大量实例剔除。所以`eureka-server`默认开始了一个应对这种场景的非正常剔除的保护模式，防止错误的剔除导致微服务集群的瘫痪。
>
> 只要集群节点数量超时达到15%以上，保护模式开启下，所有的服务将不会被剔除，等待网络修复

在eureka-server和eureka-client01，eureka-client02都开启的情况下，手动关闭一个客户端程序。通过服务端管理页面会发现，==出现红色字体提示==，并且发现关==闭的客户端实例没有被剔除出去==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112511.png)



> 可以通过修改eureka服务端的`application.properties`配置，关闭保护模式

```properties
#关闭保护模式
eureka.server.enable-self-preservation=false
```



## 客户端

### 注册

​	客户端在启动时，根据配置的地址：`service-url.defaultZone`发起http请求，访问eureka注册中心，并携带当前客户端的详细信息

### 心跳

​	启动完成，注册也就完成。

- 后续为了让`eureka-server`不做剔除处理，每隔30秒，发送心跳续约请求，携带新的更新数据到`eureka-server`服务端
- 服务端接收心跳，修改更新的注册信息（时间戳）

### 发现-同步

​	客户端一旦开启`fetch-registry`功能，每隔30秒，到注册中心抓取注册信息。

​	本质上是通过网络访问获取双层map数据，如果客户端在抓取数据后，发现本地已经存在相同数据，就不会做过多操作；一旦发现更新内容，将会同步到本地。==更新：更新新服务，老服务剔除==



# eureka注册中心高可用

> eureka治理组件结构中，存在一个eureka的情况下存在的问题

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112512.png)

> 注册中心服务器一旦出现问题，整个结构都不可使用，导致瘫痪
>
> **解决思路**：一旦是高可用的问题：添加节点，添加角色数量
>
> ==通过搭建eureka的多个服务端，实现高可用结构==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112513.png)

## 多个eureka服务端实现高可用

> ==本质思路：多个服务端之间相互作为客户端使用==
>
> 多个注册中心，互相作为服务器，互相作为客户端。同步注册和发现。这样当前微服务集群中的所有的服务在2个注册中心，都会保持相同，任何一个宕机故障，都会保留下另外一个提供者的所有注册信息

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112514.png)

## 实现高可用结构

> 配置多个注册中心工程

1. 准备两个eureka注册中心工程

   - 第一个端口：8761
   - 第二个端口：8762

2. 两个注册中心使用相同的服务名称：`spring.application.name=eureka-server`

3. 两个注册中心同时开启，抓取；启用ip优先；关闭保护模式

   ```properties
   #关闭保护模式
   eureka.server.enable-self-preservation=false
   #开启ip优先
   eureka.instance.prefer-ip-address=true
   
   #开启注册服务
   eureka.client.fetch-registry=true
   #开启抓取服务
   eureka.client.register-with-eureka=true
   ```

4. 两个注册中心的注册地址指向对方

   - `8761`注册中心：

     ```properties
     eureka.client.service-url.defaultZone=http://127.0.0.1:8762/eureka
     ```

   - `8762`注册中心：

     ```properties
     eureka.client.service-url.defaultZone=http://127.0.0.1:8761/eureka
     ```

> 启动两个注册中心，如上图所示，这种结构下启动的第一个注册中心==肯定会报两个错误==，因为第二注册中心还没有启动，第一个找不到第二个注册中心，所以会报错误。但这不影响正常使用
>
> `Connection refused: connect`
>
> `Cannot execute request on any known server`



# order-user系统使用eureka组件

1. 创建eureka注册中心
2. eureka客户端：客户端就是所有服务提供者，服务的进程都是eureka客户端
   - order系统：配置成eureka客户端，注册在注册中心：test-order
   - user系统：配置成eureka客户端，注册在注册中心：test-user



# eureka治理组件存在的问题

微服务框架中，如果仅存在eureka治理组件，仅仅多了一个server进程，多了一批注册信息的内容数据结构（双层map），不能够完成调用服务的功能。

要实现每个小系统之间的互相调用，就需要使用另一个组件：`ribbon`

[ribbon]: 5、组件-ribbon.md




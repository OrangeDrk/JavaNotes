[TOC]

# 介绍

ribbon是SpringCloud中进行==负载均衡==调用服务提供者集群的一个客户端组件，需要依赖eureka-client的发现机制，==先从注册中心抓取注册信息，根据注册信息实现负载均衡调用（拦截逻辑）==



# ribbon使用

> 在父工程下（继承了spring-boot-starter-web），通过quickstart骨架创建一个项目。简单编写controller、service逻辑，测试使用
>
> - url：localhost:9004/ribbon/hello?name=wang
> - 返回一句话，这句话是调用了eureka客户端
>   - 端口：9001/9002
>   - 服务名称：server-hi

## 配置ribbon环境

1. 修改pom.xml文件

   - 继承springboot：（父级工程已经继承）
   - 导入SpringCloud的dependency资源
   - 添加依赖
     - eureka-client依赖
     - ribbon依赖

   ```xml
   <dependencies>
       <!--eureka-client-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka</artifactId>
       </dependency>
   
       <!--ribbon依赖-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-ribbon</artifactId>
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

   

2. 配置application.properties文件

   - 端口：9004
   - 服务名称：server-ribbon
   - 注册，抓取服务
     - 注册服务：必须要开启，为了调用其他客户端
     - 抓取服务：开启，为了让别的客户端调用当前客户端
   - 注册中心地址：8761

   ```properties
   #端口9004
   server.port=9004
   
   #提供该功能的服务名称，注册在注册中心 serviceId
   spring.application.name=server-ribbon
   
   #开启注册，开启抓取的功能:默认开启
   #开启IP地址使用优先
   eureka.instance.prefer-ip-address=true
   
   #指向注册中心的地址，实现注册发现
   eureka.client.service-url.defaultZone=http://127.0.0.1:8761/eureka
   
   ```

   

3. 启动类

   - `@SpringBootApplication`:本身是一个springboot项目

   - `@EnableEurekaClient`：本身也是eureka客户端

   - 为了去调用别的客户端，需要通过注入的方式注入`RestTemplate`

     - `@LoadBalanced`：开启负载均衡

     ```java
     //构造一个RestTemplate对象，注入到业务层使用
     //启动类本省是配置类的前提，准备一个bean对象，注入使用
     @Bean
     @LoadBalanced
     //负载均衡。
     // 本质：对该对象添加过滤的逻辑，所有这个对象发送的http请求都会ribbon过滤拦截
     public RestTemplate initTemplate(){
         return new RestTemplate();
     }
     ```

   - 可以==修改负载均衡机制==

     ```java
     //自定义的负载均衡计算执行对象
     @Bean
     public IRule initMyRule(){
         return new RandomRule();//随机调用
     }
     ```

     



## 测试调用

> 实现在ribbon工程中，使用RestTemplate对象调用服务名称为`server-hi`的集群==eureka客户端==的功能

1. RibbonController

   ```java
   @RestController
   public class RibbonController {
       @Autowired
       private RibbonService rs;
   
       //localhost:9004/ribbon/hello?name=wang
       //返回一个英文，调用service-hi
       @RequestMapping("/ribbon/hello")
       public String sayHello(String name){
           //使用业务层返回调用service-hi的结果
           return "RIBBON"+rs.sayHello(name);
       }
   }
   ```

   

2. RibbonService

   > url:`"http://server-hi/client/hello?name=" + name`
   >
   > 应为RestTemplate对象在注入时通过注解：`@LoadBalanced`开启了负载均衡，他会==通过负载均衡的机制==，调用服务名称为：`server-hi`集群中的某一个客户端

   ```java
   @Service
   public class RibbonService {
       
       @Autowired
       private RestTemplate template;
   
       public String sayHello(String name) {
           //调用service-hi的功能
           //通过治理的逻辑
           String result = template.getForObject("http://server-hi/client/hello?name=" + name, String.class);
           return result;
       }
   }
   ```

   

## 测试结果

> 因为负载均衡机制自定义为了：随机
>
> 所以每次刷新页面都会随机调用`server-hi`集群中的某一客户端

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ribbon负载均衡测试结果1.png)

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ribbon负载均衡测试结果2.png)

# ribbon项目结构和逻辑

## 结构

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ribbon负载均衡原理.png)

## 负载均衡逻辑和原理

> - ribbon配合了eureka-client组件，实现了服务的抓取
> - 抓取到了注册的所有信息，包含server（9001,9002）
>   - 然后在ribbon中编写RestTemplate的访问代码实现调用`"server-hi"`服务，完成负载均衡
>   - 底层经过ribbon过滤逻辑

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ribbon项目结构.png)

1. ribbon中已经抓取到了服务信息

2. RestTemplate调用api的时候，就进入了拦截逻辑

   - 先对请求地址：`http://server-hi/client/hello?name=王老师` 解析计算

   - 获取了服务名称：`server-hi`

   - 从双层map中通过`server-hi`这个key值找到了`9001`、`9002`客户端的详细信息

   - 经过负载均衡计算（IRule的接口实现类做的计算，实现类不同，负载均衡逻辑不同）

   - 拿到详细信息中某个节点的`ip`,`port`，将请求地址中的`server-hi`服务器名称替换成`ip:port`真实节点信息

   - 最后从ribbon工程发起了最终的访问

     `http://127.0.0.1:9001/client/hello?name=王老师`

## 扩展问题

1. RestTemplate对象，不能使用域名访问服务

   `http://loaclhost:9001/client/hello?name=wang`

   在实际场景中，每个服务都在单独的服务器上，如果用localhost访问，将不能访问远程服务

2. ribbon的负载均衡相比nginx的优势

   eureka治理组件是动态的数据维护，在扩展实例的服务提供者时，已有的结构不需要任何变化

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/ribbon的作用-order-user为例.png)

3. 默认负载均衡是轮询：`RoundRobinRule`，其他的负载均衡逻辑

   ribbon使用RestTemplate调用api后，进行拦截逻辑，实现负载均衡计算，这种负载均衡计算使用的是`IRule`接口的实现类，==实现类不同，负载均衡计算逻辑也不同==

   | 实现类                   | 负载均衡逻辑                                                 |
   | ------------------------ | ------------------------------------------------------------ |
   | RoundRobinRule           | 默认负载均衡：轮询                                           |
   | RandomRule               | 随机访问                                                     |
   | WeightedResponseTimeRule | 一种权重的计算。<br />根据不同环境下，后端服务实例响应速度动态分配权重比例。后端响应速度越快，权重比例越高，负载均衡访问的次数越多；反之相同逻辑 |

   > 实现配置负载均衡逻辑的自定义：
   >
   > - 在当前系统启动类中，通过new不同的IRule实现类实现不同的负载均衡逻辑
   > - 放到方法中，通过`@Bean`，将该对象返回，作为Spring管理的Bean对象
   >
   > ```java
   > //自定义的负载均衡计算执行对象
   > @Bean
   > public IRule initMyRule(){
   >     return new RandomRule();//随机访问
   > }
   > ```



# ribbon在项目中的作用

> 以order-user系统结构，讨论eureka和ribbon实现方式

1. `order-user`系统拆分为2个工程：`order`、`user`
   - `order`：eureka-client、ribbon、RestTemplate
     - 支付功能，被前端调用、实现支付（eureka-client）
     - RestTemplate，发起请求访问user，实现用户积分的新增，相当于调用别人的服务。实现发现，并且利用ribbon做负载均衡
   - `user`：eureka-client
     - 积分新增：纯粹的被调用、实现注册（eureka-client）
     - 积分查询：被前端调用，实现注册（eureka-client）

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ribbon利用eureka动态数据维护实现负载均衡.png)



# 整合order-user系统

> order：eureka、ribbon
>
> user：eureka

## user

> 工程已存在，需要在工程中添加组件：eureka、配置文件、启动类

1. 修改pom.xml配置

   - 导入SpringCloud的依赖
   - 添加eureka客户端依赖

   ```xml
   </dependencies>
       <!--eureka客户端依赖-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka</artifactId>
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

2. application.properties配置

   - 端口不变
   - ==添加服务名称：test-user==
   - ==开启注册功能，不开启抓取服务（抓取用不到）==
   - ==ip优先==
   - ==指向`eureka:8761`注册中心==
   - 持久层配置不变

   ```properties
   #修改端口
   server.port=8092
   server.context-path=/
   
   #服务名称
   spring.application.name=test-user
   #开启注册
   eureka.client.register-with-eureka=true
   #关闭发现
   eureka.client.fetch-registry=false
   #ip优先
   eureka.instance.prefer-ip-address=true
   #注册中心地址
   eureka.client.service-url.defaultZone=http://127.0.0.1:8761/eureka
   
   #配置DataSource
   spring.datasource.username=root
   spring.datasource.password=123456
   spring.datasource.url=jdbc:mysql://localhost:3306/easymall?useSSL=false
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   
   #配置连接池 默认：tomcat jdbc
   #spring.datasource.type
   
   #SqlSession+MyBatis
   #扫描映射文件：mapper.xml路径
   mybatis.mapper-locations=classpath:/mappers/*.xml
   #别名包
   mybatis.type-aliases-package=cn.tedu.domain
   #开启驼峰命名
   mybatis.configuration.map-underscore-to-camel-case=false
   #关闭持久层框架缓存
   mybatis.configuration.cache-enabled=false
   
   ```

3. 启动类

   - 因为该系统要注册到注册中心，所以要添加客户端注解：`@EnableEurekaClient`

   ```java
   @SpringBootApplication
   @MapperScan("cn.tedu.mapper")
   @EnableEurekaClient
   public class StartUser {
       public static void main( String[] args ) {
           SpringApplication.run(StartUser.class,args);
       }
   }
   ```

   

## order

> 工程已存在，需要在工程中添加组件：eureka、配置文件、启动类

1. 修改pom.xml配置

   - 导入SpringCloud依赖
   - 添加eureka、ribbon依赖

   ```xml
   </dependencies>
       <!--eureka客户端依赖-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-eureka</artifactId>
       </dependency>
   
       <!--ribbon依赖-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-ribbon</artifactId>
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

   

2. application.properties配置

   - 端口不变
   - ==添加服务名称：test-user==
   - ==开启注册功能，不开启抓取服务（抓取用不到）==
   - ==ip优先==
   - ==指向`eureka:8761`注册中心==
   - 持久层配置不变

   ```properties
   #修改端口
   server.port=8091
   server.context-path=/
   
   #服务名称
   spring.application.name=test-order
   #开启注册，抓取---默认开启
   #ip优先
   eureka.instance.prefer-ip-address=true
   #注册中心
   eureka.client.service-url.defaultZone=http://127.0.0.1:8761/eureka
   
   #配置DataSource
   spring.datasource.username=root
   spring.datasource.password=123456
   spring.datasource.url=jdbc:mysql://localhost:3306/easymall?useSSL=false
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   
   #配置连接池 默认：tomcat jdbc
   #spring.datasource.type
   
   #SqlSession+MyBatis
   #扫描映射文件：mapper.xml路径
   mybatis.mapper-locations=classpath:/mappers/*.xml
   #别名包
   mybatis.type-aliases-package=cn.tedu.domain
   #开启驼峰命名
   mybatis.configuration.map-underscore-to-camel-case=false
   #关闭持久层框架缓存
   mybatis.configuration.cache-enabled=false
   
   ```

   

3. 启动类

   - 添加eureka客户端注解
   - 调用user服务的RestTemplate要满足ribbon拦截逻辑

   ```java
   @SpringBootApplication
   @MapperScan("cn.tedu.mapper")
   @EnableEurekaClient
   public class StartOrder {
       public static void main(String[] args) {
           SpringApplication.run(StartOrder.class, args);
       }
   
       @Bean
       @LoadBalanced//开启负载均衡
       public RestTemplate initTemplate(){
           return new RestTemplate();
       }
   }
   ```

4. 对应的service代码逻辑修改

   - 之前new的`RestTemplate`改为`@Autowired`注入
   - 修改发送请求的url地址：`"http://test-user/update.action?money="+money`

   ```java
   @Service
   public class OrderServiceImpl implements OrderService {
   
       @Autowired
       private OrderMapper orderMapper = null;
   
       @Autowired
       private RestTemplate template;
   
       @Override
       public void pay(String orderId) {
           //1.支付订单将订单状态从未支付改为已支付
           System.out.println("支付订单.."+orderId);
           //2.获取订单金额信息，计算该订单对应的积分
           Order order = orderMapper.selcOrderById(orderId);
           int money = order.getOrder_money();
           System.out.println("Order_money:"+money);
           //3.将积分更新到用户信息中
   
           //用户的积分新增不再由订单系统管理
           //发送请求，通知用户系统进行积分更新
           String url = "http://test-user/user/update.action?money="+money;
           String body = template.getForObject(url, String.class);
           System.out.println(body);
       }
   }
   ```

   



# 问题：页面如何调用服务

> - 能否实现通过js访问：http://www.pq.com/orderuser/order/pay.action访问`test-order`服务？
> - 实现nginx配置，使order、user系统负载均衡访问
> - 不能通过之前的nginx配置去访问目前的SpringCloud组件的框架集群

## 解决思路

> 自定义一个对外提供访问的工程

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ribbon问题：页面如何调用服务.png)

> 上图所述的工程就是：SpringCloud组件：==zuul组件==
>
> **笔记连接：**[zuul](6、组件-zuul.md)


[TOC]

# 生产者

1. 创建生产者SpringBoot工程
2. 引入依赖坐标
3. 编写yml配置，基本信息配置
4. 定义交换机，队列以及绑定关系的配置类
5. 注入RabbitTemplate，调用方法，完成消息发送

## 创建工程

1. 新建一个maven项目

## 引入坐标依赖

在Pom文件中引入相关依赖

```xml
<!--父工程-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
</parent>

<!--springboot整合MQ依赖-->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <!--springboot测试依赖-->
     <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
</dependencies>
```

## 写配置文件

> **application.yml**
>
> 配置Rabbit MQ的基本信息

```yaml
spring:
  rabbitmq:
    host: 172.16.98.133 #ip地址，默认localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
```

## springboot启动类

```java
public class ProducerApplication{
    public static void main(String[] args){
        SpringApplication.run(ProducerApplication.class);
    }
}
```

## rabbitMQ配置类

```java

@Configuration
public class RabbitMQConfig{
    public static final String EXCHANGE_NAME = "boot_topic_exchange";
    public static final String QUEUE_NAME = "boot_queue";
    
    // 1.交换机
    @Bean("bootExchange")
    public Exchange bootExchange(){
        return ExchangeBuilder.topicExchange(EXCHANGE_NAME).durable(true).build();
    }
    
    // 2. Queue队列
    @Bean("bootQueue")
    public Queue bootQueue(){
        return QueueBuilder.durable(QUEUE_NAME).build();
    }
    
    // 3.队列和交换机绑定关系
    /*
    	1.哪个队列
    	2.哪个交换机
    	3.routing key
    */
    public Binding bindQueueExchange(@Qualifier("bootQueue")Queue queue,@Qualifier("bootExchange")Exchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with("boot.#").noargs();
    }
    
}
```

## 测试

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class ProducerTest{
    // 1. 注入RabbitTemplate
    @AutoWired
    private RabbitTemplate rabbitTemplate;
    
    @Test
    public void testSend(){
        /**
        	参数：
        		1.交换机名字
        		2.routing key
        		3.消息
        */
        rabbitTemplate.converAndSend(RabbitMQConfig.EXCHANGE_NAME,"boot.haha","boot mq hello");
    }
}
```



# 消费者

1. 创建消费者SpringBoot工程
2. 引入start，依赖坐标
3. 编写yml配置，基本信息配置
4. 定义监听类， 使用`@RabbitListener`注解完成队列监听

## 创建工程

创建一个maven工程，引入springboot依赖

## pom文件引入依赖

```xml
<!--父工程-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
</parent>

<!--springboot整合MQ依赖-->
<dependencirs>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    <!--springboot测试依赖-->
     <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
</dependencirs>
```

## 编写yml配置

```yaml
spring:
  rabbitmq:
    host: 172.16.98.133 #ip地址，默认localhost
    port: 5672
    username: guest
    password: guest
    virtual-host: /
```

## springboot启动类

```java
public class ConsumerApplication{
    public static void main(String[] args){
        SpringApplication.run(ConsumerApplication.class);
    }
}
```

## 定义监听类

```java
@Component
public class RabbitMQListener{
    
    @RabbitListener("boot_queue")
    public void ListenerQueue(Message message){
        System.out.println(message);
        System.out.println(new String(message.getBody()));
    }
}
```


[TOC]

# 1、准备

## 1.1、依赖

```xml
<!--rabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<!--mvc-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 1.2、配置文件

```yaml
# rabbitmq 配置
  rabbitmq:
    host: 182.92.95.61
    username: admin
    password: admin
    publisher-returns: true  # 消息发送到交换机，确认机制，是否返回回调
    publisher-confirm-type: correlated  # 消息发送到交换机，确认机制，是否确认回调
    listener:
      simple:
        acknowledge-mode: manual # 采用手动应答
        concurrency: 1 #指定最小的消费者数量
        max-concurrency: 1 # 指定最大的消费者数量
        retry:
          enabled: true # 是否支持重试
    virtual-host: /test
```



# 2、注入RabbitTemplate

> 编写RabbitConfig，配置RabbitTemplate

```java
package org.example.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.annotation.Resource;

/**
 * 实现 InitializingBean，重写afterPropertiesSet，可以对注入的Bean进行初始化后的属性操作
 */
@Configuration
public class RabbitConfig implements InitializingBean {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    Logger logger = LoggerFactory.getLogger(RabbitTemplate.class);

    /**
     * 定义一个队列
     * Queue 可以有4个参数
     * 1.队列名
     * 2.durable       持久化消息队列 ,rabbitmq重启的时候不需要创建新的队列 默认true
     * 3.auto-delete   表示消息队列没有在使用时将被自动删除 默认是false
     * 4.exclusive     表示该消息队列是否只在当前connection生效,默认是false
     */
    @Bean
    public Queue helloQueue() {
        return new Queue("logQueue");
    }

    /**
     * 把消息对队列中的消息转换成json格式
     *
     * @return
     */
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }


    /**
     *  当不使用springboot自动导入，自己注册rabbitmqtemplate时，定义回调
     *  定制化amqp模版
     * <p>
     * ConfirmCallback接口用于实现消息发送到RabbitMQ交换器后接收ack回调   即消息发送到exchange  ack
     * ReturnCallback接口用于实现消息发送到RabbitMQ 交换器，但无相应队列与交换器绑定时的回调  即消息发送不到任何一个队列中  ack
     */
    /*public RabbitTemplate rabbitTemplate() {

        // 消息发送失败返回到队列中, yml需要配置 publisher-returns: true
        rabbitTemplate.setMandatory(true);

        // 消息返回, yml需要配置 publisher-returns: true
        rabbitTemplate.setReturnCallback((Message message, int replyCode, String replyText, String exchange, String routingKey) -> {
                    String correlationId = message.getMessageProperties().getCorrelationId();
                    logger.debug("消息：{} 发送失败, 应答码：{} 原因：{} 交换机: {}  路由键: {}", correlationId, replyCode, replyText, exchange, routingKey);
                }
        );

        // 消息确认, yml需要配置 publisher-confirms: true
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                logger.debug("消息发送到exchange成功,id: {}", correlationData.getId());
            } else {
                logger.debug("消息发送到exchange失败,原因：" + cause);
            }
        });

        return rabbitTemplate;
    }*/

    @Override
    public void afterPropertiesSet() throws Exception {
        // 消息返回, yml需要配置 publisher-returns: true
        rabbitTemplate.setReturnCallback((Message message, int replyCode, String replyText, String exchange, String routingKey) -> {
            String correlationId = message.getMessageProperties().getCorrelationId();
            logger.debug("消息：{} 发送失败, 应答码：{} 原因：{} 交换机: {}  路由键: {}", correlationId, replyCode, replyText, exchange, routingKey);
        }
                                        );

        // 消息确认, yml需要配置 publisher-confirms: true
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                logger.debug("消息发送到exchange成功,id: {}", correlationData.getId());
            } else {
                logger.debug("消息发送到exchange失败,原因：" + cause);
            }
        });
    }
}

```



# 3、生产者

```java
package org.example.mqtest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

/**
 * 生产者
 */
@Controller
@RequestMapping("/mqTest")
@ResponseBody
public class Producer {
    Logger logger = LoggerFactory.getLogger(Producer.class);

    @Autowired
    RabbitTemplate rabbitTemplate;

    @RequestMapping("/Producer")
    public String send(){
        Map<String, String> msg = new HashMap<>();
        msg.put("msg", "测试连接");
        msg.put("code", "200");
        logger.info("发送消息：" + msg.toString());
        rabbitTemplate.convertAndSend("mq.test", "logQueue", msg);
        return "生产者生产成功";
    }
}

```



# 4、消费者

```java
package org.example.mqtest;

import com.rabbitmq.client.Channel;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.Map;

/**
 * 消费者
 */
@Component
@RabbitListener(queues = "logQueue") // 监听队列的名字
public class Consumer {
    @Autowired
    RabbitTemplate rabbitTemplate;

    Logger logger = LoggerFactory.getLogger(Consumer.class);

    /**
     * 这里的 第一个参数可以接收生产者的消息，因为RabbitConfig中配置了消息转JSON，并且发送的是一个Map
     * 所以接收的时候也要用Map接收，不然会报错找不到接收方法的问题
     *
     * 建议：所有消息都已JSON字符串发送，接收时在进行转换，方便处理
     *
     * @param testMessage
     * @param message
     * @param channel
     * @throws IOException
     */
    @RabbitHandler
    public void process(Map testMessage, Message message, Channel channel) throws IOException{
        try {
            //确认一条消息
            channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
            System.out.println("logQueue 消费者收到消息---  : " + testMessage.toString());
        }catch (Exception e){
            //消费者告诉队列信息消费失败
            /**
             * 拒绝确认消息:
             * channel.basicNack(long deliveryTag, boolean multiple, boolean requeue) ;
             * deliveryTag:该消息的index
             * multiple：是否批量true:将一次性拒绝所有小于deliveryTag的消息
             * requeue：被拒绝的是否重新入队列
             */
            channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, true);
        }
    }
}

```



# 5、效果


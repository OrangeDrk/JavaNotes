[TOC]



# 概括

## 什么是微服务

​	应用增长迅速，功能繁多，用户终端种类庞杂。业务功能编写的特别细致。所以微服务概念出现了

​	==微==：相对而言较小，大型业务场景拆分后的独立运行系统，对比之前的功能变小了

​	==服务==：本质上就是拆分出来的独立运行的应用，每一个应用都可以称为服务。由大量的应用组成的集群，每个应用都要被别人调用：==体现被调用的概念==

## 微服务的特点

1. 独立：为了解决庞大应用中存在的并发问题，耦合问题，所产生相对独立的系统
2. 松耦合
   - 例如：order-user系统，虽然进行纵向拆分，但是并不是微服务结构，可以吧order/user称之为微服务。
   - 存在强耦合：nginx配置文件

## order-user系统

​	经过拆分后，解决了功能强耦合，并发集中，但是一旦运行，使用这种结构不合理，不规范，没有一个管理的概念。需要引入==微服务框架==解决这个问题



# SpringCloud微服务框架

## Spring家族

Spring、SpringMVC、SpringBoot、SpringCloud、SpringSession、Spring-data、Spring-Security

## SpringCloud

​	不是一项独立的技术：不是Spring团队开发的

​	只是将一些成熟的技术经过整合，调试，组合成了一套可以用来维护监听管理搭建微服务集群的组件

# SpringCloud组件

1. [eureka](4、组件-eureka.md)
2. [ribbon](5、组件-ribbon.md)
3. [zuul](6、组件-zuul.md)
4. config
5. feign
6. hystrix
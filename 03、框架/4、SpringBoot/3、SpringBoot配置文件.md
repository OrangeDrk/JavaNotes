[TOC]

# SpringBoot的配置文件

指的是读取属性使用的properties格式的全局文件。

在SpringBoot创建一些赋予属性定义的值的bean对象，没有默认值

有的一些属性是具备默认值的，例如：

- Tomcat端口：8080
- DataSource操作持久层数据源：user、password、url、driver

可以在全局配置文件【application.properties/application.yml】中指定属性。还可以覆盖默认值



# application.properties

> ==学习过程常用的格式==
>
> 是SpringBoot唯一的一个全局配置文件，可以读取属性，可以配置需要的自定义数据。

例如：

1. 修改web容器启动端口
2. 修改web容器的访问根目录

## 使用

> 默认端口是：`8080`
>
> 默认访问项目根目录是：`/`

1. 在Maven工程的`src/main/resources`文件夹下创建配置文件

   ![](img\application.properties.png)

   > 注意：此时resources文件夹会变形状，如果没有改变，需要在项目的module资源配置中调整
   >
   > ![](img\调整resources文件夹.png)

2. 编写application.properties文件

   ```properties
   #修改端口
   server.port=8090
   #修改默认项目根路径 localhost:8080/hello
   #localhost:8080/big1910/hello
   server.context-path=/big1910
   ```

   

# application.yml

> properties学习过程中常用的属性配置格式 key-value，经常会出现重复的配置，例如：

```properties
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql:///easydb
spring.datasource.username=root
spring.datasource.password=root
```

其中，`spring.datasource`重复了4次

## yml

> yml格式是==企业中常见的一种配置格式==，面向数据结构的，可以轻松的体现层级关系的配置逻辑
>
> 注：底层翻译的时候依然是编程`key-value`读取

1. properties格式

   ```properties
   server.port=8090
   server.context-path=/big1910
   ```

2. yml格式

   > `[空格]`不是真正输入的字符串，提示这里使用空格

   ```yaml
   server:
   [空格]port:[空格]8090
   [空格]context-path:[空格]/big1910
   ```

   ```yaml
   server:
     port: 8090
     context-path: /big1999
   ```



## 练习

> 将properties手动转化yml

1. properties格式

   ```properties
   spring.datasource.driver-class-name=com.mysql.jdbc.Driver
   spring.datasource.url=jdbc:mysql:///easydb
   spring.datasource.username=root
   spring.datasource.password=root
   ```

2. yml

   ```yaml
   spring:
       datasource:
           driver-class-name: com.mysql.jdbc.Driver
           url: jdbc:mysql:///easydb
           username: root
           password: root
   ```

> 实现properties与yml格式互相转化可通过工具进行
>
> 在线工具：https://www.toyaml.com/index.html
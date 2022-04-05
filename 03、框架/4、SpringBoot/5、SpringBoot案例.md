[TOC]

# 思路

## 原工程需要变动的地方

1. Spring+SpringMVC+MyBatis的xml配置文件不用再配置
2. 代码controller、service、mapper、domain不需要变动，搭建好SpringBoot环境后直接粘贴使用
3. 静态文件：静态文件放到项目中访问，可以使用Nginx来管理这些静态文件内容

## 实现转化操作步骤

1. 准备好一个SpringBoot工程
2. 具备的环境（依赖资源）
   - SSM中：Spring+SpringMVC+MyBatis
   - SpringBoot中：Spring+SpringMVC ---- `spring-boot-starter-web`
   - 持久层：MyBatis依赖，datasource配置，mybatis配置
3. 代码编写
   - 先写好启动类
   - 将原有代码按照原来的结构粘贴到当前工程中
   - 粘贴mapper映射文件
4. 静态文件
   - Nginx配置使用静态文件访问
   - 最终www.pq.com访问到一个SpringBoot启动工程



# SpringBoot整合order-user项目

## 配置SpringBoot环境

1. 使用quickstart创建工程
2. 修改pom.xml
   - 继承：自动实现继承自定义的父级maven：`springboot-parent`
   - 依赖：
     - spring-boot-starter-web
     - jdbc
     - mysql
     - mybatis
3. application.properties文件
   - 项目端口：8080
   - 项目根路径：/orderusers
   - datasource属性
     - username
     - password
     - url：指向本地数据库
     - driverclass
   - mybatis
     - 扫描映射文件：文件夹名称要注意---`classpath:/mappers/*.xml`
     - 别名包
     - 驼峰命名
     - 关闭缓存
4. 编写启动类
   - 核心注解：`@SpringBootApplication`
   - 持久层包扫描注解：`@MapperScan("cn.tedu.mapper")`



## 将源码粘贴进来

1. 代码粘贴：controller、service、mapper、domain
2. 粘贴映射文件到`resources`文件夹下
3. 准备数据库

## 手动测试所有接口功能

1. 启动系统
2. 订单支付功能测试
3. 积分查询功能测试

## 动静分离

最终一定是通过访问页面静态资源，点击页面按钮等等操作，实现JQuery的Ajax发送请求，调用到了系统的动态资源。使用Nginx完成动静分离，最终访问www.pq.com实现功能

1. 将页面静态资源保存在Nginx根目录中进行管理

   ![](img\springboot下的Nginx动静分离.png)

2. 使用www.pq.com访问到静态资源的主页:`index.html`

   - 修改本机host文件

     ```
     127.0.0.1	www.pq.com
     ```

   - nginx.conf文件中配置一个server

     ```shell
     server{
     	listen 80;
     	server_name www.pq.com;
     	location / {
     		root webapp;
     		index index.html;
     	}
     }
     ```

3. 动态资源的访问

   > 通过页面中的点击发起Ajax异步请求。因为项目虚拟路径为：`/orderusers/......`，所以`nginx.conf`的配置匹配以`/orderusers`开头的请求，匹配到的进行动态资源访问
   >
   > 积分查询：http://www.pq.com/orderusers/user/point.action
   >
   > 支付订单：http://www.pq.com/orderusers/order/pay.action

   ```shell
   location /orderusers {
   	#请求转向localhost:8080服务器
   	proxy_pass http://localhost:8080;
   }
   ```

   

# 总结

​	Nginx保存对应的静态数据，配置nginx.conf实现新的server编辑，多个location通过优先级、匹配逻辑将静态资源交给`location /` ；将动态资源交给`location /orderusers`

```shell
server {
    #静态资源访问
    location / {
        root   webapp;
        index  index.html index.htm;
    }

    #动态资源访问
    location /orderusers {
    	#请求转向localhost:8080服务器
        proxy_pass http://localhost:8080;
    }
}
```


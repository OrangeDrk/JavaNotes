[TOC]

# Servlet简介

1. 什么是servlet
   - servlet是由sun公司提供的一套接口规范，专门应用于Javaweb网站的开发技术
   - servlet是web应用服务器（Tomcat）实现了这套接口规范
   - servlet = server + applet（服务端应用程序）
2. servlet主要作用
   - 处理客户端（浏览器）发送的HTTP请求
   - 相应给客户端（浏览器）请求结果
3. servlet的API
   - Tomcat/lib库--->servlet.api.jar





# Servlet

1. src下创建包：cn.tedu.servlet

2. 创建类，实现Servlet接口，重写方法

   - `init(ServletConfig servletConfig)`：servlet初始化方法
   - `service()`：servlet处理请求和相应的方法
   - `destroy()`：servlet销毁时的方法

3. 在WEB-INF下的web.xml中配置servlet映射

   > `servlet`标签
   >
   > - `name`标签：自定义的；
   > - `class`标签：要映射的servlet类的路径
   >
   > `servlet-mapping`标签
   >
   > - `name`标签：要和servlet的name标签一致
   > - `url-pattern`标签：访问的虚拟路径

   ```xml
   <!--servlet的映射关系：浏览器请求  与 servlet代码的 映射关系-->
   <servlet>
       <servlet-name>hello</servlet-name><!--程序员自定义的名字-->
       <servlet-class>cn.tedu.servlet.Servlet01_servlet接口</servlet-class>
   </servlet>
   <servlet-mapping>
       <servlet-name>hello</servlet-name><!--必须与servlet标签的name对应-->
       <url-pattern>/hello</url-pattern><!--访问的虚拟路径-->
   </servlet-mapping>
   ```

## Servlet的生命周期

1. 当Tomcat服务器启动时：
   - 加载配置文件：web.xml（该文件正确才会启动成功）
2. HTTP请求发送到web.xml解析时，如果找到class文件，Tomcat就会自动创建一个class的对象
3. 活动：在请求处理期间是一直存在的
4. servlet的销毁：在Tomcat关闭之前，执行destroy方法，最终对象会被jvm回收



------



# GenericServlet

> 抽象类

## 与Servlet关系

> 实现了Servlet接口，重写了servlet的方法，只用关注service()方法

1. servlet是一个接口，内部只有方法签名，没有方法体
2. 如果要使用servlet就必须实现servlet接口，有以下方法：
   1. `void destroy()` ------常用
   2. `void init(ServlerConfig config);`----常用
   3. `void service(ServletRequest req, ServletResponse res)`---- 常用



## 请求方式的处理

> 请求方式7种:
>
> - 常用：get，post， 
> - 不常用：put，delete，tarce，head，options

service能接收上面7种所有的请求

> 这种方式书写很繁琐，需要简化处理
>
> - ==所以该类不使用，使用HttpServlet，继承了GenericServlet类==

- 接收到请求后，service方法种的处理逻辑要分开执行
  - get方式的请求执行get方法
  - post方式的请求执行post方法



------



# HttpServlet（✔使用）

> 介绍：
>
> 1. 继承关系：HttpServlet继承了GenericServlet，GenericServlet实现了Servlet
> 2. ==HttpServlet为抽象类==
> 3. HTTP Servlet是专门为了HTTP请求而服务的
> 4. 封装了各种请求的处理方法，例如：`doGet()`，`doPost()`





## service()

> get，post请求都能处理

```java
@Override
protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

}
```



## doGet()

> 处理get请求

```java
@Override
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    //需要的指定业务逻辑，可以被重复的调用
}
```



## doPost()

> 处理post请求

```java
@Override
protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

}
```







# HTTP协议	

## 请求

> - 一个请求行：
>   - 请求方式、请求的地址和HTTP协议版本
>
> - 多个请求头
>   - 消息报头、一般用来说明客户端要使用的一些附加信息
>
> - 空行（必须有）
>
> - 请求的数据

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110952.jpg)





## 响应

> - 一个状态行（相应行）：
>   - HTTP版本、状态码、状态消息
>
> - 多个响应头
>   - 消息报头，客户端使用的附加信息
>
> - 空行（必须有）
>
> - 响应数据
>   - 正文，服务器返回给浏览器的信息



### HTTP协议之响应

- 响应格式的结构：

  - 响应行（状态行）：HTTP版本、状态码、状态消息
  - 响应头：消息报头，客户端使用的附加信息
  - 空行：响应头和响应实体之间的，必须有的
  - 响应实体：正文，服务器返回给浏览器的信息

- HTTP常见响应状态码含义：

  > HTTP状态码由3个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用。HTTP状态码共分为5种类型：

  | 分类 | 分类描述                                       |
  | ---- | ---------------------------------------------- |
  | 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
  | 2**  | 成功，操作被成功接收并处理                     |
  | 3**  | 重定向，需要进一步的操作已完成请求             |
  | 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
  | 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

  > 常见状态码：
  >
  > | **状态码** | **状态消息**         | **解释**                                                     |
  > | ---------- | -------------------- | ------------------------------------------------------------ |
  > | 200        | OK                   | 客户端请求成功                                               |
  > |            |                      |                                                              |
  > | 400        | Bad request          | 客户端请求有语法错误，不能被服务器所理解                     |
  > | 401        | Unauthorized         | 请求未经授权，这个状态码必须和WWW-Authenticate报头域一起使用 |
  > | 403        | Forbidden            | 服务器收到请求，但是拒绝提供服务                             |
  > | 404        | Not Found            | 请求资源不存在，eg：输入了错误的URL                          |
  > | 500        | Internal Sever Error | 服务器发生不可预期的错误                                     |
  > | 503        | Sever  Unavailable   | 服务器当前不能处理客户端的请求，一段时间后可能恢复正常       |
  >
  > 







------



# Request

> ServletRequest：接口
>
> - 定义将客户端请求信息提供给某个servlet的对象
>
> - servlet容器创建ServletRequest对象，作为参数传递给service方法
> - ServletRequest接收的请求不仅仅是HTTP类型的请求，也可能是其他类型的请求
> - 默认基本都是HTTP请求
>
> ==HttpServletRequest==
>
> - 直接收HTTP协议的请求



## 继承关系

```
ServletRequest接口
	|--- HttpServletRequest接口（对ServletRequest的扩展，提供更多的方法去操作）
		|--- HttpServletRequestWrapper（实现类：tomcat服务器会自动创建一个HttpServletRequest对象）
			 HttpServletRequest req = new HttpServletRequestWrapper();
		
```



> **==HttpServletRequest常用方法参考==**`4、Request`


[TOC]

# Response对象介绍

- 继承关系

  ```
  --ServletResponse
  	---HttpServletResponse	
  ```

- 作用：

  - 响应数据到浏览器的一个对象
    - 比如：一句话/一个页面/一个多媒体资源

- 使用：

  - 设置响应头

    - `setHeader(String name,String value);`——在响应头中添加响应信息，同键会覆盖
    - `addHeader(String name,String value);`——在响应头中添加响应信息，不会覆盖
    - `setContentType("text/html;charset=utf-8");`——设置html网页，编码格式为utf-8

  - 设置响应状态

    - `sendError(int num,String msg);`——自定义状态码

      ```java
      resp.sendError(404,"Sorry");//在访问该Servlet时会显示404页面
      ```

  - 设置响应实体（非常重要）

    - `resp.getWrite().write(String str);`——响应具体的数据给浏览器



------



# 设置响应头

## `setHeader(String name,String value);`设置响应头，会覆盖

```java
resp.setHeader("mouse","two fly birds");
```

## `addHeader(String name,String value);`设置响应头，不会覆盖

```java
resp.addHeader("key","thinkpad");
resp.addHeader("key","leiGod");
```

结果：

![](..\img\response响应头.png)

## `setContentType("text/html;charset=utf-8");`设置编码格式

> 不设置编码格式无法显示中文，中文会乱码

```java
resp.setContentType("text/html;charset=utf-8");
```



## `resp.setHeader("refresh","time;url=xxxxx");`定时刷新

> 等待几秒后跳转到指定页面
>
> `resp.setHeader("refresh","3;url=xxxxx");`
>
> 应用场景：注册成功/登陆成功

```java
resp.setHeader("refresh","3;url=http://www.easymall.com");
```



## 缓存问题

> 不使用缓存：实时更新（验证码）	`resp.setHeader("Cache-control","no-cache");`
>
> 使用缓存：使用缓存数据（自动免密登陆）

1. 不使用缓存：

   > 每次访问都更新最新数据

   ```java
   //2.response对象控制浏览器不使用缓存
   //HTTP/1.0
   resp.setDateHeader("Expires",-1);//第二个参数表示时间毫秒数：-1：不使用缓存
   resp.setHeader("pragma","no-cache");
   
   //HTTP/1.1
   resp.setHeader("Cache-control","no-cache");
   
   //3.把日期输出到浏览器中显示
   resp.getWriter().write(date.toLocaleString());
   ```

   

2. 使用缓存:

   > 在规定时间内不会获取最新内容
   
   ```java
   @Override
   protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       //请求编码格式
       req.setCharacterEncoding("utf-8");
       //响应编码格式
       resp.setCharacterEncoding("utf-8");
       resp.setContentType("text/html;charset=utf-8");
   
       //1.创建一个日期对象
       Date date = new Date();
   
       //使用response对象设置  控制浏览器使用缓存
       //  HTTP/1.0
       resp.setDateHeader("Expires",System.currentTimeMillis()+1000*60*60*24);
       //  HTTP/1.1
       resp.setHeader("Cache-control","max-age=5");
       resp.getWriter().write(date.toLocaleString());
   
   }
   ```
   
   





------



# 设置响应状态

## `sendError(int num,String msg);`设置错误信息

```java
resp.sendError(404,"Sorry");//在访问该Servlet时会显示404页面
```

- 图示：

![](..\img\response响应状态码.png)





## `setStatus(状态码)`

```java
resp.setStatus(302);
```

- 图示：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111044.jpg)

------



# 设置响应实体

## `resp.getWrite().write(String str);`——响应具体的数据给浏览器

> 把HTML页面以该方式一句一句的响应到浏览器上

```java
resp.getWriter().write("<b>今天天气很好</b>");
```

![](E:\java笔记\java自学笔记\Servelt\Servlet\response响应实体.png).







------



# Response乱码问题

> 响应有两种方式：
>
> - 使用字节流响应
> - 使用字符流响应
>
> ==解决乱码问题：==
>
> - `response.setContentType("text/html;charset=utf-8");`

## 字节流响应

> `resp.getOutputStream().write(str.getBytes());`

### 问题一：

> 如果在获取字节流时没有设置编码格式，默认使用平台的编码（Windows:GBK）
>
> - 平台：操作系统 (windows：GBK)
>
> 浏览器：默认字符集：GBK

```java
String str = "我爱你中国";
//2.1 使用字节流响应
//如果在获取字节流时没有设置编码格式，默认使用平台的编码（GBK）
//      平台：操作系统   windows：GBK
resp.getOutputStream().write(str.getBytes());
```

### 问题二：

> 字节流设置utf-8的格式
>
> - 浏览器默认字符集：GBK
>
> - 响应流设置字符集：utf-8
>
> 编码格式不一致，出现乱码
>
> - ==鎴戠埍浣犱腑鍥�==

```java
String str = "我爱你中国";
resp.getOutputStream().write(str.getBytes("utf-8"));
```

### 解决方案

> 统一编码格式

```java
String str = "我爱你中国";
//设置HTTP协议的响应头，告诉浏览器使用什么类型的格式进行解析
resp.setHeader("Content-Type", "text/html;charset=utf-8");
//服务器端代码是使用utf-8进行字节流获取
resp.getOutputStream().write(str.getBytes("utf-8"));

//该种方式与上面一样
resp.setContentType("text/html;charset=utf-8");
```



## 字符流响应

> `resp.getWriter().write(str);`
>
> ==getWriter方法返回的是PrintWriter字符流==

### 问题一

> 如果进行字符流输出时没有设置字符流的编码格式，默认使用tomcat服务器的字符集：iso8859-1
>
> - 浏览器显示：？？？？？

```java
String str = "我爱你中国";
resp.getWriter().write(str);
```

### 问题二

> 字符流只设置响应实体编码：utf-8
>
> 浏览器默认编码：GBK
>
> 依然会出现乱码：

```java
String str = "我爱你中国";
resp.setCharacterEncoding("utf-8");
resp.getWriter().write(str);
```

### 解决方案

> 服务端和浏览器端统一编码格式

```java
String str = "我爱你中国";
//服务端
resp.setCharacterEncoding("utf-8");
//浏览器端
resp.setContentType("text/html;charset=utf-8");
resp.getWriter().write(str);
```


[TOC]

# 会话技术

> 会话技术的应用范畴：不仅仅是javaweb中存在，其他编程语言也存在，只不过写法不一样而已

## 概念

一次会话中多次请求和多次响应的一个过程，叫做会话

1. 一次会话：打开浏览器，访问某一个网站，直到关闭浏览器/网站服务器关闭时，会话才会断开，会话结束

## 会话的功能

> 为什么会有会话技术的存在

1. 一次会话中，多次请求与响应过程中，产生了大量的数据，==会话技术是让这些数据在不同的请求过程中实现数据的共享==
2. cookie：客户端存储
   - 负责的共享数据：不需要安全加密的数据
3. session：服务器端存储
   - 负责的共享数据：跟用户信息相关的加密的数据



------



# Cookie介绍

> ==Cookie不是域对象==
>
> 解决发送的不同请求的数据共享问题
>
> 例如：
>
> - 登录成功后，记住登录名称，重定向后可以获取用户名
>
> ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111123.jpg)

## 特点

1. ==**浏览器端的数据存储技术**==
2. 浏览器对单个cookie的大小限制为4K；对同一个域名下cookie限制为20个cookie
3. 存储的数据声明在服务器端
4. 临时存储：存储在浏览器的运行内存中，浏览器关闭即失败
5. ==定时存储==：设置了Cookie的有效期，存储在客户端的硬盘中，在有效期内符合路径要求的请求都会附带该信息

## 作用：

1. 存储少量的一般的非隐私的数据
2. 在不登陆的情况下，完成服务器对客户端的身份识别
   - 更换浏览器无法访问cookie购物车信息



## 原理：

> 基于set-cookie与cookie对象实现的

1. `response.addCookie(Cookie对象)`
   - response响应头（HTTP协议）：`set-cookie:msg="hello cy"`
2. 浏览器接收到响应头：set-cookie
   - 意味着：浏览器知道set-cookie的意思是，把cookie存储
   - 存储位置：访问浏览器的浏览器缓存中
3. 发送另一个请求
   - request请求头（HTTP协议）：`Cookie：msg="hello cy"`
   - 意味着：浏览器发送请求时，自动从cookie缓存中拿到cookie数据；封装到request请求头中并发送给服务器端
4. 服务器端获取数据：`request.getCookies();`
   - 通过request请求获取请求中的cookie参数
5. 因此，cookie数据实现了不同请求的数据共享

## 中文乱码问题：

> 支持中文

1. 支持版本问题：

   - tomcat 8之前，不包括8，直接使用中文报错，500错误
     - cookie的value值只支持英文，不支持中文，编码格式的问题
   - tomcat 8以后，包括8，可以直接使用中文

2. tomcat 8之前中文乱码解决方案：

   1. 方案一：**手动编码和解码**

      - 对中文先进行utf-8==编码==：`URLEncoder.encode(cookie.getValue(), "utf-8");`

      ```java
      Cookie cookie = new Cookie("username","张三");
      System.out.println("编码之前："+cookie.getValue());//张三
      //对cookie的value值进行编码
      String newValue = URLEncoder.encode(cookie.getValue(), "utf-8");
      System.out.println("编码之后："+newValue);//%E5%BC%A0%E4%B8%89
      cookie.setValue(newValue);
      ```

      - 获取时对cookie中的value值进行==解码==：`URLDecoder.decode(value, "utf-8");`

      ```java
      String value = c.getValue();
      System.out.println("解码之前：" + value);//%E5%BC%A0%E4%B8%89
      String newValue = URLDecoder.decode(value, "utf-8");
      System.out.println("解码之后："+newValue);//张三
      ```

   2. 方案二：jQuery提供的转码方法

      - 通过jQuery提供的转码方法进行==编码==：`encodeURI(String value)`
   
        > 也可以通过方案一的方式进行编码！！
   
        ```jsp
        <script src="${pageContext.request.contextPath}/js/jquery-1.4.2.js"></script>
        <script>
            $(function () {
                //username=张三
                var username = $("input[name='username']").val();
                //username=%E5%BC%A0%E4%B8%89
                var newUsername = encodeURI(username);
            })
        </script>
        ```
   
        
   
      - 通过jQuery提供的转码方法进行==解码==
   
        ```jsp
        <script src="${pageContext.request.contextPath}/js/jquery-1.4.2.js"></script>
        <script>
            $(function () {
                //cookie中的是：%E5%BC%A0%E4%B8%89
                //转码后：username=张三
                $("input[name='username']").val(decodeURI("${cookie.username.value}"));
            })
        </script>
        ```
   
        
   
   





------



# Cookie使用

## 创建`new Cookie(String name,String value);`

## 临时存储——添加-`resp.addCookie(c);`

> - 首先创建Cookie对象，传入名称和value值
> - 通过response响应给浏览器
>
> 注意：直接添加是==临时存储==
>
> - ==存储在浏览器的内存中，浏览器关闭后，cookie会失效==
> - 浏览器关闭，会话结束，浏览器缓存中的cookie会默认清空

```java
//添加一个cookie对象
Cookie c = new Cookie("mouse","thinkpad");
resp.addCookie(c);

//添加多个cookie对象
 //1.创建cookie对象
Cookie cookie2 = new Cookie("msg2", "hello cy");
Cookie cookie3= new Cookie("msg3", "hello cy");
response.addCookie(cookie2);
response.addCookie(cookie3);
```

- 现象：

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111124.jpg)
  
- 注意：

  - 一个Cookie对象存储一条数据，多条数据，可以多创建几个Cookie对象进行存储

## 定时存储——有效期、有效路径、响应

> ==定时存储==
>
> - ==存储在客户端的硬盘中==，（）中添加秒数，
> - 存储后，只要是访问该项目的根目录下的任意页面（不管存不存在，都会把cookie在请求头发送出去）

==设置Cookie的有效期==

> 默认情况下，浏览器关闭，cookie销毁
>
> 设置后，无论浏览器是否关闭，cookie依然存在

- `c2.setMaxAge(4*24*3600);`
- 参数为int型的数，表示秒(s)数，即该Cookie存在多少秒
  - 正数：把数据存储到硬盘中
  - 负数：-1，不使用缓存存储cookie
  - 0：直接删除一个cookie

==设置Cookie的有效路径==

> 默认情况下，多个web应用之间的cookie不能共享

1. `cookie.setPath("/Cookie/abc");`
   - 只有访问别名为abc页面时，才会把该Cookie请求发送过去
2. `cookie.setPath("/");`
   - 同一个域名下的所有web应用都能访问
   - 比如：localhost下的`/day1401`,`/day1402`都能访问
3. `cookie.setDomain(".baidu.com");`
   - 跨域名（==前提：一级域名必须一致==）共享cookie
   - 一级域名为.baidu.com的都可以访问到cookie
     - fanyi==.baidu.com==
     - tieba==.baidu.com==

==响应Cookie信息==

- `resp.addCookie(c2);`
- 将该Cookie对象响应到浏览器

```java
Cookie c2 = new Cookie("key","leiGod");
c2.setMaxAge(4*24*3600);//设置有效期
c2.setPath("/Cookie/abc");//设置有效路径
//响应Cookie信息
resp.addCookie(c2);
```

## Cookie的获取`req.getCookies();`

- 获取Cookie信息数组

  `Cookie[] c = req.getCookies();`

- 遍历数组获取Cookie信息

  > 遍历前，应该==先判断数组是否为空==

  ```java
  if (c != null) {
      for(Cookie cookie:c){
          System.out.println(cookie.getName()+":"+cookie.getValue());
      }
  ```



------



# Cookie三天免登录

> 有一个CookieServlet文件，获取cookies，如果为null,转发到Login页面，不为空，通过uid查询到用户，查询到后，重定向到登录主页
>
> 在登录Servlet中，输入的用户名，密码和库中的一致，将uid存入cookie，有效路径为CookieServlet
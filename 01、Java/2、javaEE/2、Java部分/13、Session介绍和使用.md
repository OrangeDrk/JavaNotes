[TOC]

# Session介绍


- 问题：一个用户的不同请求处理的数据共享怎么办？
- 解决：使用session技术
- 原理：
  - Session是基于JSESSIONID的cookie工作的
  - 用户第一次访问服务器，服务器会创建一个session对象给此用户，并将该session对象的JSESSIONID使用Cookie技术存储到浏览器中，保证用户的其他请求能够获取到同一个session对象，也保证了不同请求能够获取到共享的数据。

## 特点：

- ==**存储在服务器端**==
- 服务器进行创建
- 依赖Cookie技术
- 有效期:一次会话
- ==默认存储时间为：30分钟==

## 作用域：

- 一次会话（在JSESSIONID和Session对象不失效的情况下为整个项目内）
- 整个过程中，session一直存在，session存放在服务器端，存放的数据更加隐私
- request < session < ServletContext

## 作用：

- 解决了一个用户不同请求处理的数据共享问题
- 只要在JSESSIONID不失效和session对象不失效的情况下，用户的任意请求在处理时都能获取到同一个session对象



------



# Session具体使用

## 创建/获取session对象

```java
HttpSession hs = req.getSession();
```

- 如果请求中拥有session的标识符也就是JSESSIONID，则返回其对应的session对象
- 如果请求中没有session的标识符也就是JSESSIONID，则创建新的session对象，并将其JSESSIONID作为Cookie数据存储到浏览器的内存中
- 如果session对象失效了，也会重新创建一个session对象，并将其JSESSIONID存储在浏览器的内存中

> 注意：JSESSIONID存储在了Cookie的临时存储空间中，浏览器关闭即失效

## 设置session存储时间

```java
hs.setMaxInactiveInterval(5);
```

- 设置JSESSIONID在服务器的有效期：秒（s）
- 有效期过后，Servlet获取不到JSESSIONID，就会创建一个新的，响应到浏览器

> 注意：在指定的时间内session对象没有被使用，则销毁。如果使用了则重新计时
>
> 另一种修改有效时间的方法:
>
> - 在Tomcat下的conf/web.xml中修改;也可以将这句话复制到项目中的web.xml中修改,可以覆盖服务器上的web.xml配置
>
>   ```xml
>   <session-config>
>       <session-timeout>30</session-timeout>
>   </session-config>
>   ```
>
>   

## 设置session强制失效(清空session)

```java
HttpSession hs = req.getSession();
hs.invalidate();
```

- 不管有效期到没到，执行后JSESSIONID都会失效
- 在用户退出时，让JSESSIONID强制失效，释放服务器空间

## 存储/获取数据

> 存储的动作和取出的动作可以发生在不同的请求中，但是存储要先于取出执行

### 存储数据

```java
HttpSession hs = req.getSession();
hs.setAttribute("name",name);
```

- 第一个参数：key值
- 第二个参数：value值，可以是Object

### 获取数据

```java
HttpSession hs = req.getSession();
hs.getAttribute("name")；
```

- `getAttribute(String key);`通过键值获取对应的value值
- 返回的是Object类型，根据情况进行强转

## 删除数据

```java
HttpSession hs = req.getSession();
hs.removeAttribute("name")；
```



## 使用时机

一般用户在登录web项目时，会将用户的个人信息存储到Session中，供该用户的其他请求使用

## session失效处理

将用户请求中的JSESSIONID和后台获取到的Session对象的JSESSIONID进行比对，如果一致则session没有失效,如果不一致则证明session失效了,==失效后重定向到登陆页面,让用户重新登陆==



------



# session细节

1. 当浏览器关闭后，重新打开访问浏览器，访问的session不是同一个

   - 每一次关闭浏览器，JESSIONID会被清除

   - 🎯可以设置生存时间（持久化操作），关闭浏览器后，再次打开会访问同一个session对象

     ```java
     //做持久化的JESSIONID
     Cookie cookie = new Cookie("JSESSIONID",session.getId());
     cookie.setMaxAge(60*60);
     response.addCookie(cookie);
     ```

     

2. 服务器重启后，默认情况下访问的不是一个session

   - session对象在服务器关闭后，会被从内存中清理掉

   - 重启后的session对象是一个新的session对象

   - 🎯Tomcat已经解决了该问题。

     > 将web项目打包成war包，放到Tomcat/webapps目录下，启动tomcat会自动部署项目，访问项目后，正常关闭tomcat，会在`tomcat/work/Catalina/localhost/项目名`下产生一个`SESSIONS.ser`：session的序列化文件

     - 通过对象的序列化：把session对象存储到硬盘中

     - 对象的反序列化实现：服务器重启后，把硬盘中的文件解析到内存中，将session还原

       ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111130.jpg)

   - 🎯如果使用IDEA演示序列化／反序列化

     - 关闭tomcat时会自动将session序列化到work文件夹中
     - 重新启动tomcat时会自动删除work文件夹，并重新建立work文件夹，序列化的session文件被删除，无法进行反序列化

3. session失效时间

   - session默认失效时间为30分钟【tomcat/conf/web.xml】中配置了该信息

     ```xml
     <session-config>
         <session-timeout>30</session-timeout>
     </session-config>
     ```

   - 如果session失效，访问网站后在服务器端找不到该session对象，会让客户端重新登陆。服务器会自动创建一个session对象，并存储起来



------



# Session与cookie的区别

1. session存储在服务器端，cookie存储在浏览器端
2. session没有数据大小限制，cookie有大小限制（单个4K）
3. session数据的存储是安全的，cookie数据的存储不安全



# Session案例：验证码

> 需要的资源：
>
> 1. Util工具类：`VerifyCode.java` 生成验证码工具类
>    - `drawImage(OutputStream outputStream)`:生成验证码图片。显示在页面上，给用户看
>    - `getCode()`:验证码图片中的数字。该数字字符串是用于对比用户输入的验证码
>      - 该数字存储在session对象中。一次会话中生效
>    - `getRandom(int start, int end)`

1. regist.jsp页面：验证码标签

   > 当点击图片时更换验证码图片，发送一个更换验证码的请求发送到Servlet中

   ```html
   <td>
       <input type="text" name="valistr"/>
       <span style="color: red;font-size: 20px"><%=str4 == null ? "" : str4%></span>
       <img id="img" src="/validateCodeServlet" width="" height="" alt=""/>
   </td>
   ```

2. Servlet中

   > 1. 缓存【控制浏览器不使用缓存】
   > 2. 接收【更换验证码的请求】
   > 3. 获取【获取新的验证码图片】
   > 4. 获取【获取验证码正确的Code，存放在session中】
   > 5. 响应【输出流输出图片】
   
   1. 缓存
   
      ```java
      //控制浏览器不能使用缓存
      response.setDateHeader("Expires", -1);
      response.setHeader("Cache-control","no-cache");
      ```
   
   2. 接收【更换验证码的请求】
   
      ```html
      <script>
      
          $(function () {
          //验证码图片的切换:点击图片触发函数
              $("#img").click(function () {
                  console.log(123);
                  //这里改变src路径即可
                  var d1 = new Date();
                  var time = d1.getTime();
                  $("#img").attr("src","/validateCodeServlet?time="+time);
              })
      
      	});
      </script>
      ```
   
   3. 获取【获取新的验证码图片】【获取验证码正确的Code，存放在session中】【输出流输出图片】
   
      ```java
      //1.使用工具类生成一张验证码图片
      VerifyCode verifyCode = new VerifyCode();
      //生成图片，把图片保存到输出流中，并且调用write方法
      verifyCode.drawImage(response.getOutputStream());
      //2.获取验证码Code
      String code = verifyCode.getCode();
      //3.把code放到session中
      HttpSession session = request.getSession();
      session.setAttribute("valistr",code);
      ```
   
      
































[TOC]

# JSP介绍

- 问题：

  在学习了Servlet之后，使用Servlet进行页面的展现，代码书写过于麻烦。极大的影响了开发的效率，那么有没有一种方式可以让我们像写网页一样来进行网页的编程工作呢？？？

- 解决：

  ​	使用JSP技术：（HTML+CSS+JS+jQuery+Java代码）

- 概念：

  JSP全名为Java Server Pages，中文名叫java服务器页面，其根本是一个简化的Servlet设计，它是由Sun Microsystems公司倡导、许多公司参与一起建立的一种动态网页技术标准

- 优点：

  - 实现了Java代码与HTML页面的一个耦合度大大的降低，方便开发测试
  - jsp具有自己的脚本语言，能够让Java代码与HTML代码进行分离

- 缺点：JSP数据量太庞大的时候，因为底层是Java代码，会对服务器产生影响

## 特点

- 本质上还是Servlet
- 跨平台，一次编写处处运行
- 组件跨平台
- 健壮性和安全性

## JSP的访问原理

> Tomcat不认识JSP，只会执行Servlet
>
> <!--JSP页面，最终加载到tomcat服务器中进行运行时，会编译成两个文件（regist_jsp.java  regist_jsp.class）。最终jsp执行的是.class文件【C:\Users\Administrator\.IntelliJIdea2019.2\system\tomcat\Tomcat_7_0_62_javawebCode_8\work\Catalina\localhost\day16\org\apache\jsp】-->
>
> ==浏览器发起请求，请求JSP，请求被Tomcat服务器接收，执行JspServlet 将请求的JSP文件转义为对应的Java文件（也是Servlet），然后执行转义好的Java文件==

- 浏览器地址栏输入：`http://localhost:8080/Jsp/1.jsp`

- 虽然写的是1.jsp，但是Tomcat会在服务器下的conf/web.xml中寻找`*.jsp`配置

  ```xml
  <servlet-mapping>
      <servlet-name>jsp</servlet-name>
      <url-pattern>*.jsp</url-pattern>
      <url-pattern>*.jspx</url-pattern>
  </servlet-mapping>
  ```

- 有匹配的jsp配置后，会在`<servlet></servlet>`中找JSP的Servlet类

  ```xml
  <servlet>
      <servlet-name>jsp</servlet-name>
      <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
      <init-param>
          <param-name>fork</param-name>
          <param-value>false</param-value>
      </init-param>
      <init-param>
          <param-name>xpoweredBy</param-name>
          <param-value>false</param-value>
      </init-param>
      <load-on-startup>3</load-on-startup>
  </servlet>
  ```

- 该类会通过IO流读取Jsp文件，然后在输出Servlet格式的.class文件。

  - JSP文件中的Java代码会原样读取和写出
  - JSP文件中的HTML代码，会以Servlet的write的形式逐行写出

  ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111136.png)



------



# JSP的语法和指令

> ==局部代码块和全局代码块在JSP文件中写会非常的麻烦，一般开发中，把Java的逻辑代码写在Servlet中，然后传给对应的JSP文件==
>
> **三大指令**：
>
> - page
> - taglib
> - include

## Jsp的三种注释

- 前端注释`<!-- -->`(一般不要使用)

  > 会被转译，也会被发送，但是不会被浏览器执行

  ```jsp
  <!--
    前端注释
       会被转译，也会被发送，但是不会被浏览器执行
  -->
  ```

- Java注释

  > 会被转译，但是不会被执行

  ```jsp
  <%
  	//java注释
  	int a = 5;
  %>
  ```

- JSP注释`<%-- --%>`(JSP文件尽量使用JSP注释)

  > 不会被转译。客户端代码不会显示该注释掉的代码

  ```jsp
  <%--
  	Jsp注释
     		不会被转译
  --%>
  ```

  

## JSP的page指令

> `<%@page 属性名="属性值"  属性名="属性值"  .......%>`
>
> 可以写多行，多个page
>
> 作用:
>
> ​		==配置jsp文件的转译相关的参数==

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" import="java.util.*" %>
```

- page中的属性：共12个属性

  | **属性**      | **解释**                                                     |
  | ------------- | ------------------------------------------------------------ |
  | ==language==  | 声明jsp要被转译的语言                                        |
  | ==import==    | 声明转译的Java文件要导入的包，不同的包使用  `,`   隔开       |
  | pageEncoding  | 设置当前JSP所保存的编码格式;<br />同时设置Servlet响应的编码格式 |
  | contextType   | 转译之后就是：<br />`resp.setContextType("text/html;charset=UTF-8");`，<br />高版本只用写`pageEncoding="utf-8"`就行 |
  | ==session==   | 设置转译的servlet中是否开启session支持(JSP中的内置session对象)<br />默认开启<br />true表示开启；false表示关闭 |
  | ==errorPage== | 设置jsp运行错误跳转的页面<br />`errorPage=url`               |
  | isErrorPage   | 错误页面中可以使用Exception对象捕获异常,其中封装的就是上一个页面中抛出的异常对象 |
  | extends       | 设置jsp转译的Java文件要继承的父类<br />父类：包名+类名       |
  | buffer        | 表示响应流的缓冲区<br />` buffer="none | 8kb | sizekb"`      |
  | autoFlush     | 自动刷新缓冲区<br />默认为true<br />不需要人为干预           |
  | isThreadSafe  | servlet是否实现SingleThreadModel<br />true:单线程<br />false:多线程(默认) |
  | isELIgnored   | 是否忽略EL表达式<br />默认false,支持EL表达式                 |



## JSP的include指令

> 指的是 当前页面包含其他的某一个页面
>
> 不太常用，了解意义即可

```jsp
<%@include file="jsp01.jsp" %>
```



## JSP的taglib指令

> 指的是 引入第三方的lib库
>
> 主要使用在 JSTL标准标签库

```jsp
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```





## JSP的局部代码块

> 在JSP文件中的HTMl代码中穿插Java逻辑代码，进行逻辑的判断

- 特点：

  - 局部代码块中声明的Java代码会被原样转译到jsp对应的servlet文件的_JspService方法中
  - 代码块中声明的变量都是局部变量，都在`_JspService()`方法中

- 使用：`<% java代码%>`

  ```java
  <html>
      <head>
          <title>jsp基本语法学习</title>
          <meta charset="UTF-8">
      </head>
  
      <body>
          <h3>jsp基本语法学习</h3>
          <hr>
          <%
              int a = 5;
              if (a>3) {%>
          <b>jsp学习很简单</b>
          <%}%>
      </body>
  </html>
  ```

- 缺点：

  - 使用局部代码块在jsp中进行逻辑判断，书写麻烦，阅读困难

- 开发中使用：

  - Servlet进行请求逻辑处理
  - JSP进行页面展现

## JSP的全局代码块

- 特点：声明的Java代码作为全局代码转译到对应的servlet类中

- 使用：`<%! 全局代码%>`

  ```jsp
  <html>
      <head>
          <title>jsp基本语法学习</title>
          <meta charset="UTF-8">
      </head>
  
      <body>
          <h3>jsp基本语法学习</h3>
          <hr>
          <%
              int a = 5;
              if (a>3) {%>
          <b>jsp学习很简单</b>
          <%test();}%>
          <%!
              public void test(){
                  System.out.println("这是全局代码块");
              }
          %>
      </body>
  
  </html>
  ```

- 注意：全局代码块声明的代码，需要使用局部代码块调用

## JSP的脚本段语句（==常用==）

- 帮助我们快速的获取变量/方法的返回值 作为数据响应给浏览器

- 使用：`<%=变量名或者方法 %>`

  > <%=str%>
  >
  > <%=test()%>

  ```jsp
  <body>
      <h3>jsp基本语法学习</h3>
      <hr>
      <%
      	String str = "jsp学习很简单";
      	int a = 5;
      	if (a>3) {%>
      <b><%=str%></b>
      <%test();}%>
      <%!
      public void test(){
      System.out.println("这是全局代码块");
  		}
      %>
  </body>
  ```

- 注意：不要在变量名或者方法后带分号（；）

- 位置：除JSP语法要求以外的任意位置



------



# JSP的静态引入和动态引入（include）

> 两个引入的显示效果一样。一个可以有同名变量，一个不能有同名变量
>
> 优点：
>
> - 降低jsp代码的冗余，便于维护升级

## 静态引入

`<%@include file="要引入的jsp文件的相对路径"%>`

> 每个页面都用得到的信息
>
> - 新建一个JSP文件（includeStatic.jsp）,并把公共信息写到该JSP文件中
>   - 该JSP文件不会被转译成Java文件，因为它被其他JSP调用，并写入了其他的JSP
> - 在对应的页面中调用`<%@include file="includeStatic"%>`——填写JSP相对路径
>
> 特点：
>
> - ==会将引入的jsp文件和当前jsp文件转译成一个Java(Servlet)文件使用==
> - 在网页中也就显示了合并后的显示效果
>
> 注意：
>
> - 静态引入的jsp文件不会单独转译成Java文件
> - 当前文件和静态引入的jsp文件中不能够使用Java代码块声明同名变量

- includeStatic.jsp文件

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
  <head>
  </head>
  <body>
      <b>我是静态引入——网站声明</b>
  </body>
  </html>
  ```

- 某一个JSP文件

  ```jsp
  <html>
      <head>
          <title>jsp基本语法学习</title>
          <meta charset="UTF-8">
      </head>
  
      <body>
          <hr>
          <%-- jsp静态引入--%>
          <%@include file="includeStatic.jsp"%>
      </body>
  
  </html>
  ```

  


## 动态引入

`<jsp:include page="要引入的jsp文件的相对路径"></jsp:include>`

> 特点:
>
> - ==会将引入的jsp文件单独转译==，在当前文件转译好的Java文件中调用要引入的jsp文件的转译文件
> - 在网页中显示合并后的显示效果
>
> 注意：
>
> - 动态引入允许文件中声明同名变量

- includeActive.jsp代码

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
      <head></head>
      <body>
          <i>jsp动态引入---网站声明</i>
      </body>
  </html>
  ```
  
- 要引入动态jsp的页面

  ```jsp
  <body>
      <h3>jsp基本语法学习</h3>
      <hr>
      <%-- jsp动态引入 --%>
      <jsp:include page="includeActive.jsp"></jsp:include>
  </body>
  ```

  

------



# JSP标签

## JSP的forward转发标签

> 类似于Servler的请求转发，转发后浏览器的地址栏不变，是一次请求

- 使用

  `<jsp:forward page="要转发的jsp文件的相对路径"></jsp:forward>`

  ```jsp
  <%-- jsp的转发forward --%>
  <jsp:forward page="forward.jsp"></jsp:forward>
  ```

- 特点：

  - 一次请求
  - 地址栏信息不改变

- 注意：

  - 在转发标签的两个标签之间出了写`<jsp:param  />`子标签不会报错，其他任意字符都会报错
  - `<jsp:param name="str" value="aaa"/>`用来向转发的jsp流转数据
    - name属性：附带的数据的键名
    - value属性：附带的数据内容
    - 该标签会将数据以？的形式拼接在转发路径的后面，通过request获取

  ```jsp
  <%--    jsp的转发forward--%>
  <jsp:forward page="forward.jsp">
      <jsp:param name="str" value="aaa"/>
  </jsp:forward>
  ```

- 获取到子标签的数据

  `<%=request.getParameter("str")%>`

  > 通过JSP的脚本段语句输出到页面

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  <html>
      <head>
      </head>
      <body>
          <b>
              我是转发页面---forward---<%=request.getParameter("str")%>
          </b>
      </body>
  </html>d
  ```



## JSP的include动态引入标签（看上一标题）

> `<jsp:include page="要引入的jsp文件的相对路径"></jsp:include>`

```jsp
<body>
    <h3>jsp基本语法学习</h3>
    <hr>
    <%-- jsp动态引入 --%>
    <jsp:include page="includeActive.jsp"></jsp:include>
</body>
```



------



# JSP九大内置对象

> 内置对象(隐式对象)：
>
> - JSP文件在转译成其对应的Servlet文件的时候==自动生成并声明的对象==，我们在JSP页面中直接使用即可
>
> 注意：
>
> - 内置对象在JSP页面中使用，使用==局部代码块或脚本段语句==来使用。不能够在全局代码块中使用。
> - （原因：内置对象在转译时已经自动生成在service方法体中，编写的JSP就是Servlet的service方法，所以只能在==局部代码块/脚本段语句==中使用）

## 内容：九个对象

> 域的范围：pageContext < request < session < application

| **对象**        | **解释**                                                     | **注意/作用域/作用**                                         |
| --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| pageContext     | 页面上下文对象，==封存了其他内置对象==。封存了JSP的当前运行信息 | 每个JSP文件单独拥有一个pageContext对象                       |
| ==request==     | 封存当前请求数据的对象。由Tomcat服务器创建。                 | 一次请求                                                     |
| ==response==    | 响应对象，用来响应请求处理结果给浏览器的对象。               | 设置响应头、重定向                                           |
| ==session==     | 此对象用来存储用户的不同请求的共享数据的                     | 一次会话                                                     |
| ==application== | 就是**ServletContext**对象，一个项目只有一个。存储用户共享数据的对象，以及完成其他操作。 | 项目内                                                       |
| ==out==         | 响应对象，Jsp内部使用。带有缓冲区的响应对象，效率高于response对象 |                                                              |
| exception       | 异常对象。存储了当前运行的异常信息                           | 使用此对象需要在page指定中使用属性：`isErrorPage="true"`开启 |
| page            | 代表当前Jsp对象。相当于Java中的this                          |                                                              |
| config          | 也就是**ServletConfig**，主要是用来获取web.xml中的配置数据，完成一些初始化数据的读取 |                                                              |



## pageContext对象

### 获取其他8个内置对象

```jsp
Exception()=<%=pageContext.getException()%>
Page()=<%=pageContext.getPage()%>
Request()=<%=pageContext.getRequest()%>
Response()=<%=pageContext.getResponse()%>
ServletConfig()=<%=pageContext.getServletConfig()%>
ServletContext()=<%=pageContext.getServletContext()%>
Out()=<%=pageContext.getOut()%>
Session()=<%=pageContext.getSession()%>
```

### 获取四大域对象属性

```java
<%=PageContext.PAGE_SCOPE%>//page域对象(pageContext)常量----[1]
<%=PageContext.REQUEST_SCOPE%>//request域对象常量----[2]
<%=PageContext.SESSION_SCOPE%>//session域对象常量----[3]
<%=PageContext.APPLICATION_SCOPE%>//application域对象常量----[4]
```

### 操作域对象的setAttribute()方法

```jsp
<%
    //设置session的属性键值对 key : session-value
    pageContext.setAttribute("key","session-value",PageContext.SESSION_SCOPE);
    //设置application的属性键值对 key : application-value
    pageContext.setAttribute("key", "application-value", PageContext.APPLICATION_SCOPE);
    //设置request的属性键值对 key : request-value
    pageContext.setAttribute("key","request-value",PageContext.REQUEST_SCOPE);
    //设置pageContext的属性键值对 key : pageContext-value
    pageContext.setAttribute("key","pageContext-value");
%>
```

### 获取域对象中的键值对

```java
//1.session-key:
<%=pageContext.getAttribute("key",PageContext.SESSION_SCOPE)%>
    
//2.application-key:
<%=pageContext.getAttribute("key",PageContext.APPLICATION_SCOPE)%>
    
//3.request-key:
<%=pageContext.getAttribute("key",PageContext.REQUEST_SCOPE)%>
    
//4.pagecontext-key:
<%=pageContext.getAttribute("key")%>
```

### 删除/获取四大域对象的Attribute

> 1. 没有`scope`参数的`getAttribute(String key,int scope)`默认获取pageContext域对象的数据
> 2. `findAttribute()`会一次从最小的作用域对四大作用域依次查找
>    - 如果小作用域数据删除，就获取大作用域中的数据，只要找到一个就停止
> 3. `removeAttribute(String key,int scope)`如果不指定scope，会删除四大域对象中所有对应的key-value
> 4. ==scope==：
>    - `PageContext.PAGE_SCOPE`----1
>    - `PageContext.REQUEST_SCOPE`----2
>    - `PageContext.SESSION_SCOPE`----3
>    - `PageContext.APPLICATION_SCOPE`----4

```java
//获取pageContext：
<%=pageContext.getAttribute("key")%>
//删除pageContext中的key：
<%pageContext.removeAttribute("key",PageContext.PAGE_SCOPE);%><br>

//获取Request：
<%=pageContext.findAttribute("key")%>
//删除Request中的key：
<%request.removeAttribute("key");%>

//获取session：
<%=pageContext.findAttribute("key")%>
//删除session中的key：
<%session.removeAttribute("key");%>

//获取application：
<%=pageContext.findAttribute("key")%>
//删除application中的key：
<%application.removeAttribute("key");%>
```



------



# JSP中的资源路径使用

> 四个作用域对象：
>
> - pageContext：当前页面。解决了在当前页面的数据共享问题。获取其他内置对象
> - request：一次请求。一次请求的servlet的数据共享。通过请求转发将数据流转给下一个Servlet
> - session：一次会话。一个用户的不同请求的数据共享。将数据从一次请求流转给其他请求
> - application：项目内。不同用户的数据共享问题。将数据从一个用户流转给其他用户
>
> 作用：数据流转

## Jsp路径

### 在Jsp中资源路径可以使用==相对路径==

- 问题一：资源的位置不可以随意更改
- 问题二：需要使用../进行文件夹的跳出。使用比较麻烦

### **在Jsp中资源路径可以使用==绝对路径==**(必须会，开发中常用)

- /虚拟项目名/资源路径

  ```jsp
  <a href="/jsp/jspPro.jsp">jspPro.jsp</a>
  <a href="/jsp/a/a.jsp">a.jsp</a>
  ```

- 注意：

  - 在Jsp中资源的第一个/表示的是服务器根目录，相当于：localhost:8080

### ==使用JSP中自带的全局路径声明：==

- 代码：

  > - `<%`
  >       `String path = request.getContextPath();`
  >       `String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path + "/";`
  >   `%>`
  > - `<base href="<%=basePath%>">`

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" %>
  
  <%
      String path = request.getContextPath();
      String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path + "/";
  %>
  <html>
      <head>
          <base href="<%=basePath%>">
          <title>a.jsp</title>
      </head>
      <body>
          我是a.jsp </br>
          <a href="jspPro.jsp">jspPro.jsp</a>
      </body>
  </html>
  ```

- 作用：给资源起那面添加项目路径

  - `http://127.0.0.1:8080/虚拟项目名/`



------



# 在信息管理系统中查询所有用户信息

servlet层—service层—Dao层—service层—servlet层，查询到的数据以List集合返回，然后请求转发到对应的jsp显示界面，通过`req.setAttribute("usersList",usersList)`转发到对应页面。不要使用session—重定向，会造成脏读

- 脏读：查询到数据存放在session中，有效时间没过，如果用户改了性别或者其他的，查询到的数据依然是改之前了，就造成了脏读
- 使用请求转发每次查询都会查询数据，不会造成脏读



如果注册中有空字符串，数据库也允许为空，再向数据库插入该空字符串时不应该插入`''`,因该插入`null`

```java
if (user.getBirth().equals("")) {
    ps.setString(5, null);
}else {
    ps.setString(5,user.getBirth());
}
```


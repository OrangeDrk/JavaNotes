[TOC]

# ServletContext介绍

>  ==ServletContext是一个代表web应用的对象==

- 问题：
  - Request解决了一次请求内的数据共享问题，session解决了用户不同请求的数据共享问题，那么不同用户的数据共享问题该怎么办？？？
- 解决：
  - 使用ServletContext对象
- 作用：
  - 解决了不同用户的数据共享问题
- 原理：
  - ServletContext 对象由服务器进行创建，一个项目只有一个对象。不管在项目的任意位置进行获取得到的都是同一个对象，那么不同用户发起的请求获取到的也是同一个对象，该对象由用户共同拥有
- 特点：
  - 服务器进行创建
  - 用户共享
  - 一个项目只有一个
- 生命周期
  - 服务器启动到服务器关闭
- 作用域：
  - 项目内
  - ==ServletContext的作用范围比Session要大==



------



# ServletContext使用

## 获取ServletContext对象

> 三种获取方式获取到的对象相等

- ==方式一：==（常用）

  `ServletContext sc = this.getServletContext();`

- 方式二：

  `ServletContext sc2 = this.getServletConfig().getServletContext();`

- ==方式三：==（常用）

  `ServletContext sc3 = req.getSession().getServletContext();`

## Attribute数据共享

> 注意：
>
> - 不同的用户可以给ServletContext对象进行数据的存储
> - 获取的数据不存在，返回null
>
> 作用域：==整个web应用==

- 存储数据

  `sc.setAttribute("str","ServletContextSevlet学习");`

- 获取数据，返回数据类型为：Object

  `String str = (String) sc.getAttribute("str");`
  
- 获取所有的数据

  `Enumeration<String> names = servletContext.getAttributeNames();`

- 删除数据

  `removeAttribute(String key)`

## 获取项目中web.xml文件中的全局配置数据

> web.xml的全局配置数据：
>
> ```xml
> <!--配置全局数据-->
> <context-param>
>      <param-name>name</param-name>
>      <param-value>zhangsan</param-value>
> </context-param>
> 
> <context-param>
>      <param-name>pwd</param-name>
>      <param-value>zhangsanpwd</param-value>
> </context-param>
> ```
>
> 注意：
>
> - 一组`<context-param>`标签只能存储一组键值对数据，多组可以声明多个`<context-param>`进行存储
>
> 作用：将静态数据和代码进行解耦

- 获取数据

  > 通过==键值==获取

  `String str = sc.getInitParameter("name");`

- 获取所有键的枚举

  > 返回web.xml中全局配置的所有键值

  ```java
  Enumeration e = sc.getInitParameterNames();
  while (e.hasMoreElements()) {
  	String key = (String) e.nextElement();
  	System.out.println(key+":"+sc.getInitParameter(key));
  }
  
  //结果——————————————————————————————————————————————
  name:zhangsan
  pwd:zhangsanpwd
  ```



## 获取项目根目录下的资源的绝对路径

> 项目部署到Tomca后，会在Tomcat目录下由相应的文件夹，需要动态的获取文件夹下的某一个文件的绝对路径
>
> 注意：
>
> - 以下代码是默认从Tomcat路径下开始找（D:\software\tomcat7\bin\test.xml）
>
>   ```java
>   File file = new File("test.xml");
>   System.out.println("该文件绝对路径："+file.getAbsolutePath());
>   ```
>
> IDEA下：
>
> ​	web/WEB-INF/classes/c3p0-config.xml

1. ServletContext对象获取==资源路径==

   > `sc.getRealPath(String path);`——path参数为项目根目录中的路径
   >
   > - ==path参数会从web项目根目录开始找==，如果文件较深，要检查路径是否书写正确
   
   ```java
   //获取项目根目录下的资源的绝对路径
   ServletContext servletContext = this.getServletContext();
   String realPath = servletContext.getRealPath("/WEB-INF/classes/c3p0-config.xml");
   
   //结果
   D:\software\www.easymall.com\ROOT\WEB-INF\classes\c3p0-config.xml
   ```
   
   
   
2. 通过类加载器获取==web资源路径==

   > `classLoader.getResource(String url).getPath();`
   >
   > - 如果当前的应用程序是web应用，会在`WEB-INF/classes`文件中找
   > - 如果当前的应用程序是Java应用，会在`jdk/bin`文件按夹找

   ```java
   //方式三：类加载器加载web资源
   ClassLoader classLoader = ServletContext02.class.getClassLoader();//获取类加载器
   //  通过类加载器获得c3p0-config.xml配置文件
   //如果当前的应用程序是web应用，会在WEB-INF/classes文件中找
   String path = classLoader.getResource("c3p0-config.xml").getPath();
   System.out.println("web应用中的资源路径："+path);
   
   //结果
   web应用中的资源路径：/D:/software/www.easymall.com/ROOT/WEB-INF/classes/c3p0-config.xml
   
   ```

3. 通过域对象获取资源==根路径==

   > 获取web应用的路径: Application Context :url路径
   >
   > 如果资源配置了缺省的路径：/
   >
   > - 输出结果就为：` `（空白行）

   ```java
   //方式五：获取web应用的路径: Application Context :url路径
   String contextPath = servletContext.getContextPath();
   String contextPath1 = request.getContextPath();
   System.out.println(contextPath+"---"+contextPath1);
   ```

   

## 获取项目根目录下的资源的流对象

> 注意：
>
> - 获取项目根目录下某一个普通资源的流对象
> - WEB-INF不是普通资源，里面的.class文件流需要使用类加载器获取

代码：

- `getResourceAsStream(String path);`——path参数为项目根目录中的路径

```java
InputStream is = sc.getResourceAsStream("/doc/1.txt");
```





------



# 网站计数器的实现

在三天免登录、新用户登录的地方获取ServletContext对象，获取计数器变量，然后在重定向到main页面之前，计数器自加1，然后添加到ServletContext中；在主页中获取该计数器进行显示

## login处

```java
//创建网页计数器
ServletContext sc = this.getServletContext();
if (sc.getAttribute("nums")!=null) {
    int nums = (int) sc.getAttribute("nums");
    nums+=1;
    sc.setAttribute("nums",nums);
}else {
    sc.setAttribute("nums",1);
}
resp.sendRedirect("/main");
return;
```

## 三天免登录处

```java
ServletContext sc = this.getServletContext();
int nums = (int) sc.getAttribute("nums");
nums++;
sc.setAttribute("nums",nums);

resp.sendRedirect("/main");
return;
```

## 主页main处

```java
//获取网页浏览次数
ServletContext sc = this.getServletContext();
int nums = (int) sc.getAttribute("nums");
```

## 问题：

- 在不重启服务器的情况下，计数器不会置零，一旦服务器重启，计数器就会从0开始计数

- 解决办法：

  - 服务器启动时会执行：Init()方法，关闭时会执行destroy()方法
  - 在关闭时，把计数器通过流写道文件里
  - 在启动时，把计数器通过流读出来
  - 设置相应Servlet的`<load-on-startup>1</load-on-startup>`，==让该Servlet启动时优先执行==

- 代码：

  ```java
  package com.sxh.servlet;
  
  import javax.servlet.ServletContext;
  import javax.servlet.ServletException;
  import javax.servlet.annotation.WebServlet;
  import javax.servlet.http.HttpServlet;
  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import java.io.*;
  
  @WebServlet("/numsServlet")
  public class NumsServlet extends HttpServlet {
  
      @Override
      public void init() throws ServletException {
          ServletContext sc = this.getServletContext();
          BufferedReader br = null;
          FileReader fr = null;
          try {
              fr = new FileReader(sc.getRealPath("/doc/nums.txt"));
              br = new BufferedReader(fr);
              int nums = Integer.parseInt(br.readLine());
              System.out.println(nums);
              sc.setAttribute("nums",nums);
  
          } catch (FileNotFoundException e) {
              e.printStackTrace();
  
          } catch (IOException e) {
              e.printStackTrace();
          }finally {
              try {
                  fr.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
              try {
                  br.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  
      @Override
      public void destroy() {
          int nums = (int) this.getServletContext().getAttribute("nums");
          BufferedWriter bw = null;
          FileWriter fw = null;
          try {
              fw = new FileWriter(this.getServletContext().getRealPath("/doc/nums.txt"));
              bw = new BufferedWriter(fw);
              bw.write(nums+"");
              bw.flush();
              System.out.println(nums);
          } catch (Exception e) {
              e.printStackTrace();
          }finally {
              try {
                  fw.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
  
              try {
                  bw.close();
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
  }
  
  ```

  
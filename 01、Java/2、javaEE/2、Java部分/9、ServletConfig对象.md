[TOC]

# ServletConfig介绍

- 作用：

  ​	ServletConfig对象是Servlet的专属配置对象，每个Servlet都单独拥有一个ServletConfig对象，用来获取web.xml中的配置信息。

  - 打比方：Servlet是老板，ServletConfig就是该老板的秘书
  - ==获取在web.xml中给每个Servlet单独配置的数据==,主要是做当前的初始化工作
  
- 对象创建的时机：

  - ==当请求访问到当前的Servlet时，会创建一个ServletConfig对象==



------



# ServletConfig的使用

## ServletConfig的配置

> 获取每个Servlet在web.xml中的单独配置
>
> **在`<servlet>`中的`<init-param>`中的内容为该Servlet的单独配置**
>
> 可以有多个`<init-param>`

1. web.xml配置：

   ```xml
   <!--    ServletConfigServlet配置-->
   <servlet>
    <servlet-name>sg</servlet-name>
    <servlet-class>com.sxh.servlet.ServletConfigServlet</servlet-class>
    <init-param>
        <param-name>encode</param-name>
        <param-value>utf-8</param-value>
    </init-param>
       ……………………
       ……………………
   </servlet>
   <servlet-mapping>
    <servlet-name>sg</servlet-name>
    <url-pattern>/sg</url-pattern>
   </servlet-mapping>
   ```

   

2. 通过注解配置

   ```java
   /*
    * initParams  对应着配置文件中所有的<init-param>标签
    * @WebInitParam 对应着其中一个<init-param>标签
    *          name:<param-name>标签
    *          value:<param-value>标签
    */
   @WebServlet(value = "/config02", initParams = {
           @WebInitParam(name = "encoding", value = "utf-8"),
           @WebInitParam(name = "key3",value = "value3")
   })
   ```
   


## 获取web.xml中Servlet的配置信息

> 获取ServletConfig对象
>
> `ServletConfig sc = this.getServletConfig();`

1. 获取指定的key值的value值

   ```java
   //获取web.xml中的配置内容
   String code = sc.getInitParameter("encode");
   
   //结果——————————————————————————
   utf-8
   ```

   

2. 获取所有的配置信息

   ```java
   //2.2   该方法返回当前Servlet中所有的配置信息，key:value键值对信息
   Enumeration<String> initParameterNames = servletConfig.getInitParameterNames();
   while (initParameterNames.hasMoreElements()) {
       String key = initParameterNames.nextElement();
       String value = servletConfig.getInitParameter(key);
       System.out.println(key+":"+value);
   }
   ```

   


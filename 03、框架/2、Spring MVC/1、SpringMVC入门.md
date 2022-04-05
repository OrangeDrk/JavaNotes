[TOC]

# SpringMVC概述

1. SpringMVC是一个WEB层、控制层框架,主要用来负责与客户端交互,业务逻辑的调用.
2. SpringMVC是Spring家族的一大组件.Spring整合SpringMVC可以做到无缝集成.
3. ==特点== ：简单易用性能佳



# SpringMVC的组件

## 前端控制器（DispathcharServlet）

本质上是一个Servlet，相当于一个中转站，所有的访问都会走到这个Servlet中，再根据配置进行中转到相应的Handler中进行处理,获取到数据和视图后，在使用相应视图做出响应

## 处理器映射器（HandlerMapping）

本质上就是一段映射关系，将访问路径和对应的Handler存储为映射关系，在需要时供前端控制器查阅。

## 处理器适配器（HandlerAdapter）

本质上是一个适配器，可以根据要求找到对应的Handler来运行。前端控制器通过处理器映射器找到对应的Handler信息之后，将请求响应和对应的Handler信息交由处理器适配器处理，处理器适配器找到真正handler执行后，将结果即model和view返回给前端控制器

- model：返回的结果数据
- view：在那个页面展示

## 视图解析器（ViewResolver）

本质上也是一种映射关系，可以将视图名称映射到真正的视图地址。前端控制器调用处理器适配完成后得到model和view，将view信息传给视图解析器得到真正的view。

## 视图（View）

本质上就是将handler处理器中返回的model数据嵌入到视图解析器解析后得到的jsp页面中，向客户端做出响应



## 图解

![](img\SpringMVC组件.png)



# SpringMVC入门案例

## 创建web项目

![](img\SpringMVC案例-web项目.png)

## 导入SpringMVC相关开发包

![](img\SpringMVC案例-相关开发包.png)

## 配置前端控制器

> 本质上是一个servlet,在当前web项目中配置该servlet
>
> 在项目的 : `web/WEB-INF/web.xml`配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>
</web-app>
```

## 配置SpringMVC的核心配置文件

> SpringMVC默认会自动在web应用的WEB-INF目录下去寻找:
>
> - `[前端控制器ServletName]-servlet.xml`作为当前SpringMVC的核心配置文件。
>
> 创建这个文件，==这个文件本身其实就是Spring的配置文件==，所以导入Spring相关的约束信息，包括 beans、context、mvc

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd">

</beans>
```

### 此文件一般写在src目录下

> SpringMVC默认在和web.xml相同的位置即WEB-INF目录下寻找核心配置文件：
>
> - 文件名默认`[前端控制器Servlet-name]-servlet.xml`
> - 如果需要，可以通过配置，==手动指定核心配置文件的位置，和文件的名称：==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       
        <!--手动配置前端处理器配置文件在哪里-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:/springmvc.xml</param-value>
        </init-param>
        
    </servlet>
    
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>

</web-app>
```



## 创建处理器

> 开发一个处理器，写一个类实现Controller接口:
>
> - 在其中重写的`handlerRequest()`方法中编写代码，处理请求。并将处理好的数据和目标视图封装到`ModelAndView`中返回

```java
public class FirstController implements Controller {

    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("Hello Spring MVC");
        ModelAndView mav = new ModelAndView();
        mav.addObject("k1", "v1");//存储数据
        mav.addObject("k2", "v2");
        mav.addObject("k3", "v3");
        mav.setViewName("first");//存储视图
        return mav;
    }
}

```

## 配置处理器映射器

> 在SpringMVC核心配置文件中配置（`springmvc.xml`）【次配置文件名称和配置的前端控制器name相关】
>
> 配置处理器映射器中的【路径】和【处理器】的【映射关系】

```xml
<!--配置处理器映射器-->
<bean name="/first.action" class="cn.tedu.controller.FirstController"></bean>
```

## 配置视图解析器

> 在SpringMVC核心配置文件中配置（`springmvc.xml`）【次配置文件名称和配置的前端控制器name相关】

```xml
<!--配置视图解析器-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/jsp/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
```

## 开发视图

> JSP文件

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<html>
<head>
    <title>first</title>
</head>
<body>
    first.jsp..${k1}..${k2}..${k3}
</body>
</html>
```



## 发布并通过浏览器访问

![](img\SpringMVC案例-发布并访问.png)



## 整体流程

![](img\SpringMVC案例-整体流程.png)






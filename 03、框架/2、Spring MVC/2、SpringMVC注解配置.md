[TOC]



# SpringMVC配置流程

> SpringMVC支持使用注解方式配置，比配置文件方式更加的灵活易用，是SpringMVC使用的主流模式。

## 在配置文件中开启SpringMVC的注解模式

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

    <!--开启包扫描-->
    <context:component-scan base-package="cn.tedu"></context:component-scan>

    <!--开启SpringMVC注解-->
    <mvc:annotation-driven></mvc:annotation-driven>

    <!--配置视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>


</beans>
```

## 通过注解开发Handler处理器

```java
@Controller
public class FirstController {

    @RequestMapping("/first.action")
    public ModelAndView first(){
        ModelAndView mav = new ModelAndView();
        mav.addObject("k1", "kv1");
        mav.addObject("k2", "kv2");
        mav.addObject("k3", "kv3");
        mav.setViewName("first");
        return mav;
    }

    @RequestMapping("/first2.action")
    public String first2(Model model){
        model.addAttribute("k1", "vz1");
        model.addAttribute("k2", "vz2");
        model.addAttribute("k3", "vz3");
        return "first2";
    }
}

```



## 发布并访问

![](img\SpringMVC注解-发布并访问.png)

## 整体流程

![](img\SpringMVC注解-整体流程.png)

------



# SpringMVC注解方式工作原理

1. 当服务器启动时,先会加载`web.xml`,之后通过引入核心配置文件加载`springmvc-servlet.xml`.就会解析该xml配置文件.
2. 当解析到包扫描时，扫描指定的包，并将含有`@Controller`注解的类解析为处理器
3. 如果配置过`<mvc:annotation-driven/>`     就会解析Spring-MVC注解
4. 解析`@RequestMapping(value="/hello.action")`，将指定的地址和当前方法的映射关系进行保存
5. 当用户发出请求访问一个地址时，SpringMVC寻找该地址映射关系，如果存在，则找到响应处理器相应方法执行，如果找不到，则报404。


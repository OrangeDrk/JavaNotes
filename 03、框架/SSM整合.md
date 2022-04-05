[TOC]





# SSM整合概述

不用死记硬背，真正的项目开发，都会有对应的SSM整合文档，其中内容包括以下文章所描述的所有内容







------



# 导入相关开发包

> SpringMVC、Spring、MyBatis
>
> 在WEB-INF/lib中

![](https://gitee.com/sxhDrk/images/raw/master/imgs/SSM整合-项目结构.png)



------



# 创建包结构

![](https://gitee.com/sxhDrk/images/raw/master/imgs/SSM整合-开发包.png)





------



# 配置SpringMVC

## `web.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--全站乱码解决过滤器-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--配置前端控制器-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

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



## 核心配置文件：`springmvc.xml`

```xaml
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
    <context:component-scan base-package="cn.tedu.controller"></context:component-scan>

    <!--开启SpringMVC注解支持-->
    <mvc:annotation-driven/>

    <!--视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
```



## 开发`Controller`

```java
@Controller
//@RequestMapping("/user")
public class controller {

    @Autowired
    private UserService userService = null;

    @RequestMapping("/regist.action")
    public void regist(User user){
        userService.regist(user);//调用service层的注册方法
    }
}
```





------



# 配置Spring

## 核心配置文件：`applicationContext.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--包扫描-->
    <context:component-scan base-package="cn.tedu"></context:component-scan>

    <!--注解方式DI-->
    <context:annotation-config></context:annotation-config>

    <!--注解方式AOP-->
    <aop:aspectj-autoproxy/>

   
</beans>
```



## `web.xml`

> ==在原有的基础上，添加监听器==，使Spring容器在web应用初始化时自动加载：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置监听器,服务器已启动加载SpringIOC容器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value><!--该路径是Spring的配置文件路径-->
    </context-param>
  
</web-app>
```



## 开发`Service`

### 接口

```java
public interface UserService {
    public void regist(User user);
}
```



### 实现类

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper = null;

    @Override
    @Transactional
    public void regist(User user) {
        System.out.println("userservice 的注册");
        int i = userMapper.insertOne(user);
        if(i>0){
            System.out.println("注册成功");            
        }
    }
}
```





------



# 配置MyBatis

## 核心配置文件：`sqlMapConfig.xml`

> 单独配置MyBatis要在核心配置文件中配置数据源。
>
> ==SSM整合后，数据源配置到Spring中，由Spring容器自动管理==

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

</configuration>
```



## Spring核心配置：`applicationContext.xml`

> 在原有的基础上添加如下配置
>
> 将MyBatis的`SqlSessionFactory`配置到Spring中，由容器自动管理
>
> 开启MyBatisBean扫描器，负责为`MapperBean`生成实现类（==接口开发==）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">

 
    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///ssmdb"></property>
        <property name="user" value="root"></property>
        <property name="password" value="123456"></property>
    </bean>

    <!--整合MyBatis-->
    <bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        <property name="configLocation" value="classpath:sqlMapConfig.xml"></property>
        <property name="mapperLocations" value="classpath:mappers/*.xml"></property>
    </bean>

    <!--MyBatisBean扫描器，负责为MapperBean生成实现类-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.tedu.mapper"></property>
    </bean>

</beans>
```



## 创建`bean`

```java
public class User {
    private int id;
    private String name;
    private int age;

    public String toString() {}
    public User(int id, String name, int age) {....}
    public User() {}
    
    //成员变量的get,set方法，此处省略。。。。。。。
}

```



## 映射文件：`UserMapper.xml`

> 此处编写SQL语句
>
> `<mapper namespace="cn.tedu.mapper.UserMapper">`此处的名称空间为对应==接口的全类名==

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tedu.mapper.UserMapper">

    <select id="selectBetweenAge" resultType="cn.tedu.domain.User">
        select * from user where age between #{min} and #{max};
    </select>

    <insert id="insertOne">
        insert into user  values (null,#{name},#{age});
    </insert>
</mapper>
```



## 开发`Mapper`

> MyBatis的接口开发：
>
> - 类名：映射文件中的名称空间`<mapper namespace="cn.tedu.mapper.UserMapper">`
> - 方法名：映射文件中的对应的语句标签`id`— `<select id="selectBetweenAge".....`
> - 方法参数：对应语句所需要的参数—`select * from user where age between #{min} and #{max};`
>   - 如果是集合，可以直接传递
>   - 如果是单值，可以通过`@Param`注解标注该参数传递给哪个SQL占位符
> - 方法返回值：映射文件对应语句标签的`resultType`，`<select..... resultType="cn.tedu.domain.User">`
>   - 如果返回多个值，就返回集合,泛型和`resultType`相同

```java
public interface UserMapper {
    public List<User> selectBetweenAge(@Param("min") int min, @Param("max") int max);
    public int insertOne (User user);
}
```



------



# 配置声明式事务处理

## Spring核心配置：`applicationContext.xml`

> 在原有的基础上添加如下配置
>
> ==需要引入`tx`约束==

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    <!--开启事务管理注解-->
    <tx:annotation-driven/>

</beans>
```


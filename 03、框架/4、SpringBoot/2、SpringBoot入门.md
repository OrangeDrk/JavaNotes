[TOC]

> 讲师：
>
> - 姓名：肖旭伟
> - 联系方式：xiaoxuwei4478
> - 备注方式：



# 背景

开发环境的特点：

1. 做复杂的配置，做步骤繁琐的发布安装（spring框架）☹
2. 配置，安装相对非常简单（springboot）🙂

由于==简单-快==特点。出现了spring新的框架：springboot



# SpringBoot

基于spring实现快速配置，方便搭建的一个工具框架

## springboot特点

> 1. 独立运行的spring容器
> 2. 内嵌的servlet容器
> 3. springboot简化依赖
> 4. 自动配置

### 独立运行的spring容器

> 回忆：spring框架运行，必须依赖第三方的容器才可以实现运行启动：外部Tomcat

SpringBoot：可以像开发代码一样，使用main方法开启一个程序的入口。实现Spring容器独立运行

### 内嵌的servlet容器

> 回忆：在开发web应用时，打成war包，放到Tomcat运行

SpringBoot：由于存在独立运行的特点，自动配置特点，可以专门为web容器做单独配置。

配合第一个特点独立运行的Spring容器，依然可以使用main方法启动一个web应用。

默认情况下使用Tomcat，可以换成其他容器：jetty、undertow

### springboot简化依赖

SpringBoot基于Spring，扩展了大量的代码，这些代码都需要非常复杂非常多的jar包支持。

SpringBoot方便依赖的使用，实现了利用maven这种项目管理工具的简化依赖，例如：

- 依赖了`spring-boot-starter-web`
- 相当于把之前使用的所有spring springmvc jackson等相关的依赖jar包全部导入到工程里来。
- 而且简化依赖根据使用场景不同，使用功能不同区分的

### 自动配置

SpringBoot提倡0配置文件（xml消失了）。

封装了大量自动配置代码。可以根据项目中使用的依赖，实现底层自动配置

例如：以前使用SSM框架配置持久层

- 准备xml配置文件，xml配置文件中有很多bean标签
  - DataSource：username，password、driverclass、url
  - sqlSession：别名包，扫描mybatis配置文件，加载的映射文件mapper.xml，注入datasource等等
  - mapperScan：扫描接口mapper的所在包路径

> 1. SpringBoot不需要配置任何xml文件为持久层mybatis整合做准备。
> 2. 只需要提供对应的属性值即可
> 3. 并不是所有的场景都需要持久层支持，当我们依赖了`spring-boot-starter-jdbc`的时候SpringBoot可以判断即将使用持久层，会创建对应的配置内容





# 搭建第一个Springboot工程

> 搭建之前需要了解【Maven的继承】、【Maven聚合（module聚合）】
>
> **详解查看**：
>
> [Maven项目管理]: 1、Maven项目管理.md

## 创建一个父级工程、聚合工程

> 实现SpringBoot所有demo案例的测试编写

1. 使用：`quickstart`骨架创建工程

   ![](img\搭建第一个SpringBoot工程1.png)

2. 修改`pom.xml`文件，添加`<packaging>`类型为`pom`

   ```xml
   <packaging>pom</packaging>
   ```

3. 父级工程继承`spring-boot-starter-parent`资源

   ```xml
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>1.5.9.RELEASE</version>
   </parent>
   ```

   

## 使用聚合功能实现第一个测试案例

1. 右键`springboot_parent`选择【new-module】创建一个新工程，使用`quickstart`骨架

   ![](img\springboot第一个测试module.png)

2. 使用demo案例工程编写配置一个web应用，实现访问

   > url：`localhost:8080/hello?name=wang`
   >
   > 返回响应：`hello springboot I am wang`

3. 搭建一个web应用，springboot已经通过自动配置实现内嵌servlet容器的调用，只需要添加一个简化依赖即可

   > 通过IDEA创建子工程时，子工程会自动添加`<parent>`标签，继承父级工程
   >
   > 在实现该web应用的pom.xml中添加如下配置

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

4. 编写启动类

   > 对于SpringBoot来讲，能够实现独立运行，基于Spring完成了一套方法的封装，最终只需要调用run方法，在main方法中执行，即可实现单独启动spring、springMVC框架进程

   ```java
   /**
    * 使当前class类成为测试案例中唯一的springboot启动类，需要使用main调用run方法
    * 需要使用springboot的核心注解
    */
   @SpringBootApplication
   public class StarterDemo01 {
       public static void main(String[] args) {
           //调用springboot的静态run方法传递当前启动类
           //反射对象
           SpringApplication.run(StarterDemo01.class,args);
       }
   }
   ```

   

5. 实现controller编写

   > controller service通过依赖注入、控制反转
   >
   > 所有的代码所在的包名，必须是创建Maven是groupId路径下（跟包的下级包）
   >
   > ![](img\springboot项目的自定义包位置.png)

   - IndexController

     ```java
     /**
      * RestController在spring更新升级过程中出现
      * 组合注解，可以将单独功能的多个注解组合成新的注解，新的注解
      * 具备所有单个注解的功能
      */
     @RestController//专门为当前环境中习惯使用ajax交互逻辑准备的
     //组合注解，当前controller类的所有方法的返回值，都会封装到响应体的数据中
     public class IndexController {
         @Autowired
         private IndexService indexService;
         
         //接收一个请求 /hello 接收参数 name 返回打招呼
         @RequestMapping("/hello")
         //@ResponseBody//将返回值直接作为响应体内容不去拼接页面前缀后缀，访问页面
         public String sayHi(String name){
             return indexService.sayHi(name);
         }
     }
     
     ```

   - IndexService

     > 在SpringBoot中，不需要创建接口文件，直接通过注解完成

     ```java
     /**
      * 1、之前代码使用接口，实现松耦合，注入实现类完成所有业务层代码编写
      * 2、互联网框架都是直接在实现类上添加service注解完成
      */
     @Service
     public class IndexService {
         public String sayHi(String name){
             return "hello spring boot i am "+name;
         }
     }
     
     ```

     

# 对比SpringBoot和Spring+SpringMVC

| **对比内容**                                                 | **springboot**                | **spring+springmvc**                                         |
| ------------------------------------------------------------ | ----------------------------- | ------------------------------------------------------------ |
| web.xml                                                      | 配置文件消失了                | 第三方tomcat运行时加载web.xml通过web.xml加载框架配置xml      |
| applicationContext.xml                                       | 配置文件消失了                | 1.context:component-scan:包扫描，扫描包路径下所有类上的注解@Controller,@Service,@Repository,@Autowired,@Resource,@Value  2.其他bean标签配置 |
| spring-mvc.xml                                               | 配置文件消失了                | 开启springmvc注解驱动使得@RequestMapping,@ResponseBody..     |
| 一些properties文件  mysql.properties  datasource.properties  redis.properties | 整合了  applicatin.properties | 用来配置需要在配置文件中创建bean对象的属性 ${name}相当于properties有name=*** |
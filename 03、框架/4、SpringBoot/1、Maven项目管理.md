[TOC]

1. maven的依赖：依赖具有传递性
2. ==maven继承==：maven项目可以继承其他maven项目，统一资源版本
3. ==maven聚合==：对项目的统一操作，一旦一个大项目中，多个maven小项目，各自执行自己的命令，可能会出现很多问题，可以通过聚合的管理实现操作的统一
   - 发布全发布
   - 安装全安装
   - 测试全测试

# Maven的继承

## 继承的意义

在Maven的项目管理工具中，提供继承的特性，可以利用继承实现所有项目==资源版本统一==的问题

![](img\Maven的继承.png)

## Maven工程实现继承

### 准备父级工程

> 定义一些继承资源（都是标签）

- 注意：所有可以被继承的父级工程==`<packaging>`类型（打包类型）必须是`pom`类型==：`<packaging>pom</packaging>`

- **搭建配置步骤**

  - 使用quickstart骨架创建Maven工程：parent

  - 实现一些依赖资源，让子工程测试中继承

    ```xml
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.3.7.RELEASE</version>
    </dependency>
    ```

  - 使用install命令将parent安装到本地库

### 搭建子工程

- 继承parent，获取继承的资源（dependency就会继承）
- **搭建配置步骤**
  
  - 使用quickstart骨架创建Maven工程：child
  
  - 实现继承：子工程配置parent标签，使用三个坐标标签指向parent工程
  
    ```xml
    <parent>
        <groupId>cn.tedu</groupId>
        <artifactId>parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    ```
  
    



## 继承资源

1. `<dependency>`：==相当于强制继承==（父级中该标签的资源，在被继承时会强制导入子级中）

2. `<groupId>`

3. `<version>`

4. `<properties>`：父级工程定义的变量

5. `<dependencyManagement>`：常用的一种统一资源的继承方式，==只会继承版本而不会强制让子工程依赖资源==

   ```xml
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework</groupId>
               <artifactId>spring-beans</artifactId>
               <version>4.3.7.RELEASE</version>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```



# module聚合

Maven在管理大量的工程时，每一个工程都可能要执行编译、打包、安装、测试、部署等生命周期的命令，一个一个去执行这些命令效率非常低。

Maven提供了聚合的配置，实现统一执行命令

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427112113.png)

## 实现聚合的配置

挑选一个聚合工程：parent，在聚合工程中添加标签：`<module>`指向管理的所有聚合子工程

```xml
<modules>
    <module>../child1</module>
    <module>../child2</module>
    <module>../child3</module>
</modules>
```

当执行某一Maven命令时，所有聚合的Maven项目都会执行

![](img\Maven的聚合.png)



## IDEA工具实现module聚合

> 聚合需要创建工程，配置相对路径的`<modules>`标签，比较繁琐，实际上所有的开发工具都==具备直接支持聚合工程，聚合子工程的创建==

IDEA：

1. 直接在当前工程右键创建module即可实现聚合的管理
2. 不会将聚合子工程存放到独立的workspace，而是会以当前聚合工程目录为根目录创建聚合子工程

> 可以通过对继承的了解，对聚合的了解，创建IDEA的Maven工程时，同一个大的项目可以通过一个【父工程/聚合工程】实现管理工作（Maven==管理==工具）
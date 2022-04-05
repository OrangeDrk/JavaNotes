[TOC]

# 服务器概述

## 硬件服务器

> oracle服务器（一台价值百万美金）

- 内存32G起步,一般128G
- 学习使用，直接使用笔记本当作一台硬件服务器即可（模拟硬件服务器：本地电脑）

## 软件服务器

> 特指：web应用服务器，他是一个应用程序，该应用程序能够提供浏览器的访问

1. web应用服务器的常用分类：
   - WebLogic：收费，目前应用最广泛的服务器
     - 服务器本身很庞大，几十个G
     - 一般是使用在大型的项目当中：京东、淘宝
   - WebSphere：收费
     - 服务器很庞大，几十个G
   - Tomcat：免费
     - 体积很小，安装程序只有十几M
     - Tomcat是在中小型企业中/学习中 应用最广泛的web服务器



# Tomcat

1. 运行环境：
   - 需要Java-jdk的支持，必须要先安装jdk才可以
   - 环境变量必须要配置JAVA_HOME
2. 与Java版本的对应
   - Tomcat6：jdk5.0
   - Tomcat7：jdk6.0
   - Tomcat8：jdk7.0
   - Tomcat9：最新版本，不建议使用
3. Tomcat7启动：一闪而过
   - JAVA_HOME没有配置
   - 已经启动过，再次启动时将不能启动/有其他程序占用8080端口号
4. 解决无法启动的问题：
   - cmd：netstat -ano，查看占用8080端口号的进程PID，把他结束掉
   - 把Tomcat的端口号改成别的，例如：8090（conf/server.xml--connector port）,修改完后重启Tomcat
5. Tomcat的==80端口==是一个==默认端口==：访问Tomcat时，==直接输入localhost==就可以访问到
6. Tomcat的目录结构
   - \bin	存放启动和关闭Tomcat的可执行文件
   - \conf    存放Tomcat的配置文件
   - \lib    存放所依赖的jar包
   - \logs    存放日志文件
   - \temp    存放临时文件
   - \webapps    存放web应用（开发人员编写的网站代码）
   - \work    存放JSP转换后的Servlet文件（Tomcat运行的真正的目录）
     - 比如：.java文件经过Tomcat运行，.class文件存放在\work文件夹下



## 基本概念

- 本身是一个web应用
- tomcat是一个servlet容器
- Servlet容器就是web容器，web容器则不一定是Servlet容器。 



# Web应用部署3种方式

## 方式一：更改需重启服务器

> 创建一个web应用：创建一个文件夹news1
>
> - news1：这个文件夹就是一个web应用
>   - 1.html
>   - 1.css
>   - 1.js
>   - WEB-INF文件夹
>     - classes：编译之后的Java代码（.class）
>     - lib:存放项目需要的jar包
>     - web.xml：核心的配置文件

- 部署：

  - 在Tomcat/conf/server.xml中添加配置

    > 此种方式，配置文件中不要写中文注释
    >
    > `<Context docBase="D:\software\news1" path="/aaa"/>`
    >
    > - `docBase`:真实路径
    > - `path`:浏览器的访问的虚拟路径

    ```xml
    <Host name="localhost"  appBase="webapps" 
          unpackWARs="true" autoDeploy="true">
    
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />
    
        <Context docBase="D:\software\news1" path="/aaa"/>
    ```

  - 在项目路径下的 `/WEB-INF/web.xml` 中配置

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                          http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">
    </web-app>
    ```

- 访问：

  - 重新启动Tomcat服务器
  - 浏览器地址栏输入：`localhost:8080/aaa/1.html`----/aaa为配置的网站访问虚拟路径



## 方式二：直接部署，不需重启

> 1. 创建一个web应用：创建一个文件夹news2
>    - news2：这个文件夹就是一个web应用
>      - 1.html
>      - 1.css
>      - 1.js
>      - WEB-INF文件夹
>        - classes：编译之后的Java代码（.class）
>        - lib:存放项目需要的jar包
>        - web.xml：核心的配置文件

- 部署

  - web应用目录下的`web.xml`依然要书写

  - 在目录`tomcat7\conf\Catalina\localhost`下，创建一个`news2.xml`

    - tomcat会把`news2.xml`的文件名==前缀==当作  ==当前web应用的虚拟路径====(localhost:8080/news2/1.html)==

  - 编辑`news2.xml`文件，添加项目的真是路径

    ```xml
    <Context docBase="D:\software\news2" />
    ```

- 配置缺省的ROOT.xml文件

  - 把`news2.xml`重命名为`ROOT.xml`
  - 浏览器地址栏直接写：`localhost/1.html`即可访问
  - 优先级：
    - 自己配置的`ROOT.xml`---->Tomcat下`wabapps/ROOT/WEN-INF/ROOT.xml`

- 访问：

  1. 非缺省ROOT.xml：`localhost:8080/news2/1.html`
  2. 缺省ROOT.xml：`localhost:8080/1.html`



## 方式三：自动管理

> tomcat会自动管理webapps中的每一个web应用

- 部署：

  - 把web应用复制到`tomcat/conf/webapps/`下

  - 默认缺省的方式

    - 把webaspps下的项目名直接改为ROOT即可

  - 默认缺省的访问界面，直接输入`localhost`，就会显示1.html内容

    - 把ROOT/WEB-INF/web.xml文件中，添加配置信息

      ```xml
      <welcome-file-list>
          <welcome-file>1.html</welcome-file>
      </welcome-file-list>
      ```

- 访问：

  - 非缺省：通过`localhost:8080/news3/1.html`即可访问
  - 缺省：通过`localhost`访问即可



------



# 配置虚拟主机

1. 在`tomcat7/conf/server.xml`中的`<Host name="localhost"></Host>`下添加配置内容`<Host></Host>`

   > name：想要修改的域名
   >
   > appBase：项目的根路径（当前目录结构 D:\software\baidu\news3\各种文件）

   ```xml
   <Host name="www.baidu.com" appBase="D:\software\baidu"></Host>
   ```

2. 修改本地Host文件

   > Host路径：`C:\Windows\System32\drivers\etc\Host`
   >
   > 浏览器中的域名访问会通过DNS域名解析器进行解析，不修改本地localhost的话无法访问自己的虚拟主机；通过配置本地Host文件绕过DNS解析

   ```
   127.0.0.1       www.baidu.com
   ```

3. 测试

   浏览器地址栏：`www.baidu.com/news3/1.html`即可





# IDEA配置Tomcat

1. Run—Configurations—+—Tomcat Server

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110934.jpg)

2. 在上面界面选择Deployment—右侧+号，添加该项目的架构

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110935.jpg)

3. 启动Tomcat，可通过配置的虚拟路径访问该web项目
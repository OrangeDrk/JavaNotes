[TOC]

# Maven基本命令

> 可以利用maven管理项目的整个生命周期，这是通过maven的不同命令来实现的。
>
> ==以下方式在开发中不使用，IDEA使用时只用鼠标点击即可==

## 创建项目

> 命令：`mvn archetype:generate`

1. 进入要创建项目的目录，执行命令：`mvn archetype:generate`

2. 提示要求选择创建项目的程序骨架

   > 默认提供了十种，我们目前知道两种即可
   >
   > 所谓项目的骨架指的是项目的不同的结构，不同项目往往是具有不同的结构的，例如基本的java项目和javaweb项目的结构就是不同的，在使用mvn创建这些项目时通过指定不同的骨架来创建出不同结构的项目。

   - 提示选择创建项目要使用的框架语句

     ```c
     Choose a number or apply filter (format: [groupId:]artifactId, case sensitive co
     ntains): 7: 
     ```

   - 普通Java项目:`maven-archetype-quickstart`

     ```c
     7: internal -> org.apache.maven.archetypes:maven-archetype-quickstart (An archet
     ype which contains a sample Maven project.)
     ```

   - 普通web项目:`maven-archetype-webapp`

     ```c
     10: internal -> org.apache.maven.archetypes:maven-archetype-webapp (An archetype
      which contains a sample Maven Webapp project.)
     ```

3. 要求输入`groupId`

   > 通常情况下，groupid要以公司域名的倒写形式来声明
   >
   > 类似于==包名==

   ```c
   Define value for property 'groupId':cn.tedu
   ```

4. 要求输入`artifactId`

   > 可以理解为项目名

   ```c
   Define value for property 'artifactId':MVNDemo01
   ```

5. 要求输入`version`

   > 默认即可

   ```c
   Define value for property 'version' 1.0-SNAPSHOT:
   ```

6. 要求输入`package`

   > 默认即可

   ```c
   Define value for property 'package' MVNDemo01: :
   ```

7. 检查信息并确认

   ```c
    Y: :
   ```

8. 创建出的项目结构

   ```shell
   MVNDemo01
   |-src
   	|-main
   		|-java --> 主程序代码
   		|-resources --> 主程序的资源文件(主要是配置文件)
   	|-test
   		|-java -->测试代码
   		|-resources -->测试程序的资源文件(主要是配置文件)
   |-target-->目标文件夹，src/main和src/test下的资源处理过后产生的文件存放在此处
   |-pom.xml -->当前项目的maven核心配置文件
   
   ```



## 编译

> 命令：`mvn compile`

1. 会自动导入`pom`文件中指定的依赖。
2. 会将`src/main`目录下的源码和资源编译后存放到`target/classes`下。

**注意**：`test`文件夹下的所有内容在编译，打包，安装过程都不参加，但是会参加测试过程。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/使用maven命令编译项目.png)



## 测试

> 命令：`mvn test`

1. 通过此命令可以执行test文件夹下的测试用的内容，实现项目测试
2. 首先会将`src/test`目录下的源码和资源编译后存放到`target/test-classes`下，之后执行其中的测试代码，输出测试结果到控制台，同时测试结果保存一份到`target/surefire-reports`中

```c
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running cn.tedu.AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.01 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.917 s
[INFO] Finished at: 2020-01-17T19:16:24+08:00
[INFO] ------------------------------------------------------------------------
```

![](https://gitee.com/sxhDrk/images/raw/master/imgs/使用maven命令测试项目代码.png)



## 清理

> 命令：`mvn clean`

1. 清理mvn命令，此命令可以==清除target文件夹==
2. ==在其他mvn命令执行之前，通常都建议大家先执行一次mvn clean==，这样可以将之前其他操作产生的结果清除，防止对本次执行的命令产生影响。

```c
D:\mvntest\MVNDemo01>mvn clean

[INFO] Scanning for projects...
[INFO]
[INFO] -------------------------< cn.tedu:MVNDemo01 >--------------------------
[INFO] Building MVNDemo01 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ MVNDemo01 ---
[INFO] Deleting D:\mvntest\MVNDemo01\target
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.549 s
[INFO] Finished at: 2020-01-17T19:16:15+08:00
[INFO] ------------------------------------------------------------------------
```



## 打包

> 命令：`mvn package`

1. 打包命令，会将编译完成的资源打包成指定格式(jar包/war包)

2. 具体怎样打包取决于pom.xml文件中的配置

   ```xml
   <packaging>jar</packaging>
   ```

3. `mvn package`命令同时==隐含了编译 测试 打包过程==

4. 建议在执行此命令之前最好先执行一次`mvn clean`防止之前其他命令产生target内容影响此次命令执行的结果。



## 安装

> 命令：`mvn install`

1. maven项目安装，这会将打包好的包及其相关的资源文件存放到本地库中由maven进行管理，成为了maven所管理的一个资源。
2. `maven install`命令会隐含进行编译 测试 打包 安装
3. 建议在执行此命令之前最好先执行一次`mvn clean` 防止之前其他命令产生 的target影响此次命令执行的结果。



## 发布

> 命令：`mvn deploy`

1. maven项目发布，将maven本地库中管理的资源发布到远程库中。
2. 但是无论是中央库还是镜像库都不允许随意上传部署，所以无法在中央库和镜像库中实现这个过程。
3. ==但是如果是自己搭建的私服，是可以通过这个过程完成资源发布的==。



## 真实开发过程

```c
mvn archetype:generate -->创建项目
配置pom文件，声明需要的依赖
mvn compile --->项目编译，此时会自动导入pom文件中声明的依赖jar

while 开发未完成:
	进行开发...
	mvn clean -->项目清理
	mvn compile -->项目编译，将写好的代码编译到target
	mvn test -->项目测试
	...

mvn package -->项目打包
mvn install -->项目安装
mvn deploy -->项目发布

```





# pom文件的编写

> maven管理的项目中，通过项目根目录下的pom.xml文件进行核心配置。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!--当前项目的maven资源坐标-->
    <groupId>cn.tedu</groupId>
    <artifactId>MVNDemo03</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--此项目在mvn package的过程中，要打成什么样的包，通常是jar或war-->
    <packaging>jar</packaging>

    <!--项目名称-->
    <name>MVNDemo03</name>
    <!--项目主页-->
    <url>http://maven.apache.org</url>

    <!--参数配置-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!--资源坐标-->
    <dependencies>
        
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.7.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-core</artifactId>
                    <version>4.3.7.RELEASE</version>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>cn.tedu.App</mainClass> <!-- 此处为主入口-->
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <configuration>
                    <attach>true</attach>
                </configuration>
                <executions>
                    <execution>
                        <phase>compile</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

```



## 项目的基本信息

```xml
<project 
         xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <!--当前项目的maven资源坐标-->
    <groupId>cn.tedu</groupId>
    <artifactId>MVNDemo01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--此项目在mvn package的过程中，要打成什么样的包，通常是jar或war-->
    <packaging>jar</packaging>

    <!--项目名称-->
    <name>MVNDemo01</name>
    <!--项目主页-->
    <url>http://maven.apache.org</url>

    <!--参数配置-->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
</project>

```



## 依赖信息

> 声明当前项目所依赖的资源，maven根据此配置自动导入相关资源。
>
> 可以查询需要导入的资源的官网，通常会提供dependency的写法，或者也可以去maven中央库的网页版(https://mvnrepository.com/)中搜索，同样可以找到相应资源的依赖配置方法。

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>3.8.1</version>
    <scope>test</scope>
</dependency>
```

### 可选配置

1. scope属性

   > 在项目中导入的依赖并不一定在项目全生命周期中都要用，此时可以通过scope属性指定依赖应用的范围。

   ```xml
   <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>3.8.1</version>
       <scope>test</scope>
   </dependency>
   ```

   | 属性值   | 作用                                                         |
   | -------- | ------------------------------------------------------------ |
   | compile  | ==编译范围==<br />scope的默认值就是compile。  在编译,测试,打包,安装,发布全部生命周期都存在的依赖资源 |
   | test     | ==测试范围==  <br />运行测试代码时,才加载的依赖资源,打包,安装,发布都不参加. |
   | runtime  | ==运行时范围==<br />和compile的唯一区别,就是不参加编译;  例如JDBC可以不写代码编译,但是必须在运行,打包安装其他阶段参加. |
   | provided | ==在编译阶段使用,但是运行,打包安装都不参加==<br />官方给了一个案例:servlet-api,编辑servlet,web应用等代码使用的内容,必须使用provided;(web应用,在tomcat中运行),如果将servlet-api依赖资源打包到war包,扔到tomcat执行会出现冲突;编写任何web应用时,使用到servlet-api的资源,必须添加provided的范围; |
   | system   | ==系统范围==<br />  在当前项目的环境中存在需要使用的jar包资源,maven没有提供groupId artifactId version,可以使用system指定从本地路径进行加载  <br />`<dependency>` <br />        `<groupId>cn.tedu</groupId> `<br />       `<artifactId>wotongzhuo</artifactId>`<br />       `<version>1.0</version> ` <br />       `<scope>system</scope> `<br />       `<systemPath>D:\\piaoqian\\xxw\\xxw.jar</systemPath>`<br />  `</dependency> ` |

2. exclusions移除依赖传递

   > 在项目中导入A依赖，单A本身又依赖B和C,B又依赖的D，则Maven会智能的自动导入B、C和D，这样依赖就被传递了开来，这个过程就称之为依赖的传递。
   >
   > 依赖传递是一个非常优良的特性，可以省去maven使用过程中的大量配置细节，但这种智能的自动导入过程偶尔也会造成包引入的冲突，造成程序运行出错。此时可以通过配置`exclusions`来打断依赖的传递来解决问题。
   >
   > 简单来说配置的`exclustions`就是在告诉maven在导入某个包的过程中，指定的依赖传递不要自动导入。

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-context</artifactId>
       <version>4.3.7.RELEASE</version>
       <exclusions>
           <exclusion>
               <!-- 坐标指定移除内容 -->
               <groupId>org.springframework</groupId>
               <artifactId>spring-core</artifactId>
               <version>4.3.7.RELEASE</version>
           </exclusion>
       </exclusions>
   </dependency>
   ```



## 插件信息

> 可以额外的增强maven的功能，实现一些特殊效果

1. main插件

   > 用来指定打包出来的jar中的入口方法

   ```xml
   <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-jar-plugin</artifactId>
       <configuration>
           <archive>
               <manifest>
                   <addClasspath>true</addClasspath>
                   <mainClass>cn.tedu.App</mainClass> <!-- 此处为主入口-->
               </manifest>
           </archive>
       </configuration>
   </plugin>
   ```

   

2. 打源码插件

   > 用来额外打包源代码

   ```xml
   <plugin>
       <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-source-plugin</artifactId>
       <configuration>
           <attach>true</attach>
       </configuration>
       <executions>
           <execution>
               <phase>compile</phase>
               <goals>
                   <goal>jar</goal>
               </goals>
           </execution>
       </executions>
   </plugin>
   ```

   
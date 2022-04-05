[TOC]

# 搭建Maven私服Nexus

## nexus仓库类型

> nexus默认内置了很多仓库，这些仓库可以划分为4种类型，每种类型的仓库用于存放特定的jar包

1. hosted：宿主仓库

   部署自己的jar包到这个类型的仓库，包括Releases和Snapshots两部分，Releases为公司内部发布版本仓库、Snapshots为公司内部测试版本仓库

2. proxy：代理仓库

   用于代理远程的公共仓库，如maven中央仓库，用户连接私服，私服自动去中央仓库下载jar包或插件

3. group：仓库组

   用来合并多个hosted/proxy仓库，通常我们配置自己的maven连接仓库组

4. virtual（虚拟）---==不再使用==

   兼容Maven1版本的jar或者插件

## 将项目发布到Maven私服

> Maven私服是搭建在公司局域网内的Maven仓库，公司内的所有开发团队都可以使用。例如技术研发团队开发了一个基础组件，就可以将这个基础组件打成jar包发布到私服，其他团队成员就可以从私服下载这个jar包到本地仓库并在项目中使用。

**将项目发布到maven私服操作步骤如下：**

1. 配置Maven的settings.xml文件

   - id：可以随便写，只要不冲突(重名)就好
   - username：私服用户名
   - password：私服密码

   ```xml
   <server>
       <id>gwssi-maven-public</id>
       <username>gwssi</username>
       <password>Abc123!@Gwssi</password>
   </server>
   
   <server>
       <id>gwssi-maven-snapshot</id>
       <username>gwssi</username>
       <password>Abc123!@Gwssi</password>
   </server>
   
   <server>
       <id>gwssi-maven-release</id>
       <username>gwssi</username>
       <password>Abc123!@Gwssi</password>
   </server>   
   ```

2. 配置项目的pom.xml文件

   - `<repository>`中的`<id>`要和maven的`settings.xml`文件的`release`下的`id`保持一致

   ```xml
   <distributionManagement>
   	<repository>
       	<id>gwssi-maven-release</id>
           <url>xxxxxxxxxxxxx</url>
       </repository>
       <snapshotRepository>
       	<id>gwssi-maven-snapshot</id>
           <url>xxxxxxxxxxx</url>
       </snapshotRepository>
   </distributionManagement>
   ```

3. 执行 `mvn deploy` 命令

   - 如果Maven项目的版本是：xx-SNAPSHOT的话，会deploy到私服的`snapshot`下
   - 如果Maven项目的版本是：XX-RELEASE的话，会deploy到私服的`release`下

## 从私服下载jar到本地仓库

具体操作步骤如下：

1. 在Maven的settings.xml文件中配置下载模板

   ```xml
   <!--配置私服下载jar包的模板 开始-->
   <profile>
       <!--profile的id，不是固定写法-->
       <id>public-snapshots</id>
       <repositories>
           <repository>
               <!--仓库id，respositories可以配置多个仓库，保证id不重复-->
               <id>public-snapshots</id>
               <!--仓库地址，及nexus仓库组的地址-->
               <url>http://public-snapshots</url>
               <!--是否下载releases构件-->
               <releases>
                   <enabled>false</enabled>
               </releases>
               <!--是否下载snapshots构件-->
               <snapshots>
                   <enabled>true</enabled>
               </snapshots>
           </repository>
       </repositories>
       <pluginRepositories>
           <!--插件仓库，maven的运行依赖插件，也需要从私服下载插件-->
           <pluginRepository>
               <!--插件仓库的id不允许重复，如果重复后边配置会覆盖前边-->
               <id>public-snapshots</id>
               <url>http://public-snapshots</url>
               <releases>
                   <enabled>false</enabled>
               </releases>
               <snapshots>
                   <enabled>true</enabled>
               </snapshots>
           </pluginRepository>
       </pluginRepositories>
   </profile>
   <!--配置私服下载jar包的模板结束-->
   ```

2. 在Maven的settings.xml文件中配置激活下载模板

   ```xml
   <activeProfiles>  
       <!--模板的id-->
       <activeProfile>public-snapshots</activeProfile>
   </activeProfiles>
   ```



## 将第三方jar包安装到本地仓库和私服

1. 将第三方jar包安装到本地仓库

   ```shell
   mvn install:install-file -Dfile=jar包名字 -DgroupId=jar包的groupId -DartifactId=jar包的artifactId -Dpackaging=jar
   ```

2. 将第三方jar包安装到私服

   - 下载jar包

   - 在maven的settings.xml配置文件中配置第三方仓库的server信息

     ```xml
     <server>
     	<id>thirdpary</id>
         <username>admin</username>
         <password>admin123</password>
     </server>
     ```

   - 执行`mvn deploy`命令进行安装

     ```shell
     mvn deploy:deploy-file -Dfile=jar包名字 -DgroupId=jar包的groupId -DartifactId=jar包的artifactId -Dpackaging=jar -Durl=xxxxxx -DrepositoryId=配置文件中第三方库id(上一步的id)
     ```

     
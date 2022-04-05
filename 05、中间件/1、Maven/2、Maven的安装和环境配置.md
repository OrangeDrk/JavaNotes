[TOC]



# Maven的安装和配置

## 下载

`http://maven.apache.org/download.cgi`



## 安装

1. 装好jdk

   `maven3.6`以上的版本至少需要==jdk8==，配置好`JAVA_HOME`环境变量。

2. 安装maven

   解压maven到任意目录（路径中不要出现中文或空格）

3. 配置环境变量

   - 配置`MAVEN_HOME`环境变量指向maven的安装目录
   - 配置`PATH`环境变量指向maven安装目录的`bin`目录

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/配置maven环境变量.png)

4. 配置maven

   > maven的核心配置文件是`conf/settings.xml`
   >
   > 在正式使用maven之前需要配置这个文件
   >
   > 主要需要指定**本地库**和**镜像库**的地址

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   
   <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
   
       <!--本地库配置，默认在 ${user.home}/.m2/repository-->
       <!-- localRepository
          | The path to the local repository maven will use to store artifacts.
          |
          | Default: ${user.home}/.m2/repository
         <localRepository>/path/to/local/repo</localRepository>
         -->
       <localRepository>D:/mvn_repo</localRepository>
   
       <pluginGroups>
       </pluginGroups>
   
       <proxies>
       </proxies>
   
       <servers>
       </servers>
   
       <!--配置镜像库-->
       <mirrors>
           <!--配置阿里镜像库-->   
           <mirror>
               <id>nexus-aliyun</id>
               <mirrorOf>central</mirrorOf>
               <name>Nexus aliyun</name>
               <url>http://maven.aliyun.com/nexus/content/groups/public</url>
           </mirror>
       </mirrors>
       ...其他配置...
   </settings>
   
   ```

   
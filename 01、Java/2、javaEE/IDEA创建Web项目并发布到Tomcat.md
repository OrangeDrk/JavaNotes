[TOC]

# 创建项目

1. 创建Javaweb项目

2. 在`web/WEB-INF`下创建`classes`文件夹和`lib`文件夹

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110535.jpg)

3. 将项目的编译输出路径改为`/web/WEB-INF/classes`文件夹

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110536.jpg)

4. 将项目的`lib`文件夹作为`jar Directory`添加到依赖库

   > 可以修改名称，便于分辨

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110537.jpg)

5. 将项目的web文件夹设置为`Sources`

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110538.jpg)





# 部署项目

> 1. IDEA中的Artifacts是整个Web项目编译完后的一个文件夹。将其输出路径改为本地电脑新建的项目名（www.easymall.com）中的`ROOT`下
> 2. 在本地电脑Hosts中配置域名：127.0.0.1    www.easymall.com
> 3. Tomcat启动后可通过缺省路径名（www.easymall.com）直接访问该项目

## 配置缺省路径

1. 修改系统Hosts

   ```
   127.0.0.1	www.easymall.com
   ```

2. 创建一个文件夹（www.easymall.com）

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110539.jpg)

   创建 子文件夹（TOOT）

   ​	![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110540.jpg)

   

3. IDEA中选择

   - Project Structure—Artifacts—选择要部署的项目
     1. 修改Output directory为之前创建的ROOT子文件夹
     2. 选择下方的Output Layout
        - 点击加号—Directory Content—选择要发布的项目的web文件夹

4. 图解：

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110541.jpg)

   

# IDEA发布项目的==访问页面执行流程==

1. 浏览器发送请求：http://www.easymall.com

2. tomcat服务器的connector解析http请求

3. tomcat引擎会找到一个虚拟主机：localhost/www.easymall.com

4. IDEA发布tomcat时，会自动生成一个ROOT.xml的缺省文件

   > - IDEA中的信息
   >
   >   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110542.jpg)
   >
   > - 生成的目录
   >
   >   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110543.jpg)

   ```xml
   <Context path="" docBase="D:\software\www.easymall.com\ROOT" />
   ```

5. index.jsp是默认的一个欢迎界面，在tomcat7/conf/web.xml中配置

   ```xml
   <welcome-file-list>
       <welcome-file>index.html</welcome-file>
       <welcome-file>index.htm</welcome-file>
       <welcome-file>index.jsp</welcome-file>
   </welcome-file-list>
   ```

6. 找到欢迎界面的配置，然后自动的去打开index.jsp，然后浏览器展示出来



# IDEA发布项目的==Servlet执行流程==

1. 浏览器www.easymall.com/register
2. tomcat的connector解析http请求
3. tomcat引擎找到虚拟主机，再找到ROOT.xml，通过xml文件找到docBase文件夹路径
4. tomcat找到WEB-INF下的web.xml
5. 通过web.xml中的servlet的映射关系
   - tomcat根据请求找到映射关系，对应的class文件
   - 根据class文件，找到WEB-INF/classes下对应的.class文件并创建对象
   - 调用对象的init()初始化方法，初始化信息
   - 调用doGet()，doPost()，service()方法，执行业务逻辑
     - 业务逻辑的书写
     - 数据库的操作
     - 根据业务重定向/请求转发
   - 当服务器关闭时，执行destroy()方法，进行对象的销毁，最终JVM垃圾回收机制会把该对象回收掉
6. 响应结果

# 图解

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427110544.jpg)
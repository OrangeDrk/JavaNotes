[TOC]

# 创建项目

1. 创建Javaweb项目

2. 在`web/WEB-INF`下创建`classes`文件夹和`lib`文件夹

   ![](https://note.youdao.com/yws/api/personal/file/34399F5A3F354F9E8305A0BFFB3B8895?method=download&shareKey=1a8b777c1b98fa2c4ab94bd5ca13a010)

3. 将项目的编译输出路径改为`/web/WEB-INF/classes`文件夹

   ![](https://note.youdao.com/yws/api/personal/file/B644756C50994800AE8A965B00FB5CDE?method=download&shareKey=ed2acb78555821c58b389a470776cc0a)

4. 将项目的`lib`文件夹作为`jar Directory`添加到依赖库

   > 可以修改名称，便于分辨
   > 也可以选择library，new一个新的library
   
   ![](https://note.youdao.com/yws/api/personal/file/E6C6276C72EF44DCBF11C1AD690EDE06?method=download&shareKey=8a818e1358e609a88cecefaf1d3a69da)

5. 将项目的web文件夹设置为`Sources`

   ![](https://note.youdao.com/yws/api/personal/file/F90B5958E27C47CDAF2343F1A8645395?method=download&shareKey=cd6ae506acb132c8bafa8c3c6ad90421)





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

   ![](https://note.youdao.com/yws/api/personal/file/12D1A3F56A71468AAE23EF0B05CE45C1?method=download&shareKey=c38125779316eef7cde6fc53fe9ac9b7)

   创建 子文件夹（TOOT）

    ![](https://note.youdao.com/yws/api/personal/file/CA5D1A69DC234BAB82E999EDEFCD62DB?method=download&shareKey=d7028b9bf2e34cb82af3ce507e7b175c  )

   

3. IDEA中选择

   - Project Structure—Artifacts—选择要部署的项目
     1. 修改Output directory为之前创建的ROOT子文件夹
     2. 选择下方的Output Layout
        - 点击加号—Directory Content—选择要发布的项目的web文件夹

4. 图解：

   ![](https://note.youdao.com/yws/api/personal/file/87876E4AA16546FD8BFAEFA7FA56B927?method=download&shareKey=c43a3aa9284a3f1163ed4a63684424f9)
   
------
   
# IDEA发布项目的==访问页面执行流程==

1. 浏览器发送请求：http://www.easymall.com

2. tomcat服务器的connector解析http请求

3. tomcat引擎会找到一个虚拟主机：localhost/www.easymall.com

4. IDEA发布tomcat时，会自动生成一个ROOT.xml的缺省文件

   > - IDEA中的信息
   >
   >   ![](https://note.youdao.com/yws/api/personal/file/63BD127409194C7591C2304E796FC2DB?method=download&shareKey=33ea5973c6443058f4aa14ffb4084ec7)
   >
   > - 生成的目录
   >
   >   ![](https://note.youdao.com/yws/api/personal/file/A8B62160440F41EB875BCFAD5FEC87BC?method=download&shareKey=f82feb1c37e1982d8e1be0be3782ade1)

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


--------

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

--------

# 图解

![](https://note.youdao.com/yws/api/personal/file/605D02EF6E994AC0ACF5C68AB83DE9C3?method=download&shareKey=e6dceceb2d4b8b1ffd8eba7d91e3a7a7)

   
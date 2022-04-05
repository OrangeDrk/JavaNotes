[TOC]

# Lucene不便之处

## 没有实现分布式

Lucene存储在一个节点，应该引入多台服务器，实现一个索引文件的数据拆分。共同维护索引的数据，保证拆分后的结果形成高可用的集群

![](https://gitee.com/sxhDrk/images/raw/master/imgs/elasticsearch基本结构.png)



## 对其他语言不友好

Lucene是使用Java编写的全文检索，但是使用全文检索的技术不止一个，有C、Python、PHP、C#、R、GO等。这些语言如果使用Lucene技术：

- 要不：将Lucene的逻辑转换成其他语言
- 或者：学习java，使用Java开发Lucene



# ElasticSearch

## 介绍

ElasticSearch是基于Lucene的==搜索服务器==（可以启动的web应用），可以提供大量的客户端使用HTTP协议访问

![](https://gitee.com/sxhDrk/images/raw/master/imgs/Lucene无法实现分布式高可用.png)

## REST风格

> REST是HTTP协议创始人在2000年一篇论文上，提出的一种对HTTP协议使用的风格建议



### url地址表示资源

> 同类资源可以用路径中的不同数据表示

访问一个用户数据：

- 平时：`http://localhost/user/manage/query?username=eeee`，不满足REST风格，没有体现url是资源的概念

- REST风格

  `http://localhost:8080/user/manage/query/eeee`

  `http://localhost:8080/user/manage/query/admin`

  `http://localhost:8080/user/manage/query/haha`

  `http://localhost:8080/user/manage/query/adsfc`



### HTTP请求方式表示操作

> 对同一个资源有多种不同的操作

- 平时

  `/user/manage/saveUsername` 表示新增

  `/user/manage/saveUserEmail`

  `/user/manage/query` 表示查询

  `/user/manage/update` 表示更新

  ==三个url地址，可能操作同一个用户（资源）==

- REST风格

  - put请求：`/user/manage/{userId}` 请求体参数  表示新增
  - get请求：`/user/manage/{userId}` 请求体参数  表示查询
  - delete请求：表示删除
  - post请求：表示更新

## SpringMVC支持REST风格

```java
//支持REST风格

@RequestMapping(value="haha/{id}",method = RequestMethod.DELETE)
public void delete(){ }

@RequestMapping(value="haha/{id}",method = RequestMethod.PUT)
public void save(){ }

@RequestMapping(value="haha/{id}",method = RequestMethod.POST)
public void update(){ }
```

> 1. url定义资源，路径参数必定存在 
> 2. 请求方式定义了操作put新增，post更新，delete删除，get查询 



## 不是所有请求都必须满足REST风格

> 微服务集群中，很多情况下，对用一个资源除了增删查改之外，还有很多细小的功能请求
>
> 比如：查username、userId、userJson等
>
> 一般在这种情况下，还是需要很多自定义的url请求



## ElasticSearch

> 请求url采用REST风格，给客户端使用

**原因**：如果url不采用REST风格，而是随意在后面拼接，各种操作多达20000url时就记不住了

**特点**：

- ES天生就是分布式高可用的集群
- 分布式：索引被切分成5份，每一份都是一个完整的索引文件结构，大量document只要做分片计算，在对应不同的分片文件夹中存储，就能实现分布式
  - N个分片，文件夹名称: 0-N-1
  - hash取余算法，对docId唯一值做取余，找到对应分片进行存储
- 高可用：每一个切分出来的分片，都会计算输出一个从分片



# ElasticSearch基本结构

==存储层==：存储生成的索引、索引中的文档数据、倒排索引表数据

==功能扩展==：基于Lucene实现了索引文件的管理（创建、搜索），还实现了分布式、实现了搭建集群的各种功能

==用户协议层==：HTTP协议的REST风格，给所有的客户端提供统一的访问功能

![](https://gitee.com/sxhDrk/images/raw/master/imgs/elasticsearch介绍.png)



# REST测试工具

## postman

> 自己的账号：17603393007@163.com；密码：sxh19970225

![](https://gitee.com/sxhDrk/images/raw/master/imgs/postman工具.png)

## 火狐插件：RESTCLIENT

![](https://gitee.com/sxhDrk/images/raw/master/imgs/修改集群名称.png)



# ElasticSearch安装配置

## 安装

> 安装到3台云主机上，搭建ES的高可用分布式集群

1. 解压安装包到`/home/software/`下

   ```sh
   [root@10-9-104-184 ~]# tar -xf /home/resources/elasticsearch-5.5.2.tar.gz -C /home/software/
   ```

2. 目录结构

   - bin：启动命令所在文件夹
   - config：核心配置文件`elasticsearch`所在目录
   - plugins：ik分词器插件等

3. 更换用户和用户组

   > ES对安全性有要求，不允许root用户直接启动。
   >
   > - 需要注册一个普通用户：es
   > - 并把用户和用户组赋给该普通用户

   ```sh
   [root@10-9-104-184 software]# chown -R es:es /home/software/elasticsearch-5.5.2/
   ```

   > `-R`：递归处理，表示将后面文件夹中的所有文件的用户和用户组都改为es



## 配置和启动

> 即使使用es普通用户，进入bin目录执行elasticsearch命令启动，也会出现各种问题
>
> 问题之一就是ES默认只允许localhost客户端访问，不允许远程访问，需要修改配置文件



### 修改`elasticsearch.yml`配置文件

1. 修改集群名称

   > ==17行，集群名称==。是ES启动整个集群时使用的名称标识

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/修改节点名称.png)

2. 修改节点名称

   > ==23行，节点名称。==
   >
   > 每一个集群中的ES节点，都应该有一个唯一的名称。
   >
   > （如果注释掉，每次启动都会分配一个随机的名字）

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/修改host地址.png)

3. 关闭bootstrap

   > ==43行，bootstrap相关==
   >
   > CentOS 6.5没有安装bootstrap相关插件，所以ES启动时如果要求开启bootstrap，会报错，但是不影响使用

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/开启http协议端口.png)

4. 修改host地址

   > ==56行，host地址==
   >
   > 默认使用localhost，只允许本地请求访问到ES节点，需要修改成服务器ip地址，允许远程访问

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/火狐插件-RESTCLIENT.png)

5. 打开http协议端口

   > ==60行，http协议端口==
   >
   > 还有一个TCP/IP协议的Java语言访问的端口9300

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/关闭bootstrap.png)

6. 开启http访问插件权限

   > 在文件末尾添加一个http访问插件权限 
   >
   > 后续会开启一个`head-master`插件，视图化展示es中的数据。需要es开启这个权限。

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/开启http访问插件权限.png)

7. 搭建集群时，需要多配置2行内容

   - 一个是：协调器
   - 一个是：最小有效master数量，防止脑裂出现



### 启动

> 进入到bin文件夹，使用普通用户执行启动命令

```sh
[root@10-9-104-184 software]# su es
[es@10-9-104-184 bin]$ elasticsearch
```

> ` elasticsearch -d` ：可以后台运行elasticsearch





# 操作ES

> 可以对启动的ES实现命令操作，本质就是HTTP请求，满足REST风格

## ES中的数据管理结构

> 与数据库结构对比

| mysql数据库  | elasticsearch全文检索 |
| ------------ | --------------------- |
| database库   | index索引             |
| table表格    | ==type类型==          |
| rows行数据   | document文档          |
| column列数据 | field域属性           |



## 添加一个索引文件

> 新增了一个叫做index01的索引文件到ES节点中

```sh
[root@10-9-104-184 software]# curl -XPUT http://10.9.104.184:9200/index01
```

> `curl`：linux提供的一个支持HTTP协议的命令脚本，有很多选项可以携带：HTTP协议的头、参数等
>
> - `-X`：后面跟着这次请求的方式（PUT，DELETE，GET，POST）
> - `-d`：请求体的内容
> - `-h`：请求头的内容



## ES创建的索引结构

在Linux调用curl命令创建一个索引文件后，会返回一个json字符串

```json
{
  "acknowledged": true, 操作确认
  "shards_acknowledged": true 分片成功
}
```

> 索引文件创建后，这个索引文件初始是空的

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES数据分布计算.png)

> 开始创建的索引文件叫：index01，但是ES创建成功后就不叫index01了，该文件名称是计算后的ID值，例如：`neShnvvZQyiT2-3BLXmYJA`
>
> 要知道index01对应的是哪个ID值的文件夹，可以通过curl命令获取index01的信息，在信息中可以查看

```sh
curl -XGET http://10.42.0.33:9200/index01
```

> 结果：
>
> ```json
> {
>     "index02": {
>         "aliases": {},//别名，可以通过别名操作索引
>         "mappings": {},//默认空的，一旦数据增加，mapping将会出现动态结构
>         "settings": {//索引的各种配置信息
>             "index": {
>                 "creation_date": "1582533919910",
>                 "number_of_shards": "5",
>                 "number_of_replicas": "1",
>                 "uuid": "neShnvvZQyiT2-3BLXmYJA",//就是当前索引id值
>                 "version": {
>                     "created": "5050299"
>                 },
>                 "provided_name": "index02"
>             }
>         }
>     }
> }
> ```
>
> 



## ES的数据分布计算

> document数据一旦新增到某个索引中，会利用document中的唯一值`docId`计算hash取余（对分片个数取余），然后存到对应的分片中
>
> 注意：==一个索引文件被切分成多个分片后，不允许修改分片数量==（防止数据未命中）

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES创建的索引结构.png)



# ES的视图化插件：head

> 在启动ES之前配置的`elasticsearch.yml`文件最后一行：`http.cors.enabled: true`
>
> 修改`head`插件的配置文件`/home/presoftware/elasticsearch-head-master/Gruntfile.js`

```sh
[root@10-9-104-184 elasticsearch-head-master]# vim Gruntfile.js  
```

> 这是一个js文件，将==93行==的host改成`head`插件所在的**服务器ip地址**



## 启动

在`head`文件夹根目录执行启动命令

- `grunt server`：控制台运行
- `grunt server &`：后台运行

> 启动成功后，会在控制台打印访问head插件的url地址和端口号：`9100`



## 远程访问

> 通过浏览器输入head提示的ip和端口，访问head插件
>
> 可以通过head插件，连接ES，发送各种命令

在连接标签中，输入ES的地址：`http://ES服务器ip:9200`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/远程访问head插件.png)

1. `es01`：节点名称，elasticsearch.yml配置的node-name值
2. ⭐：表示当前ES集群的master节点（与之前主从复制的master意义不同）
3. 0-4方框：每个索引切分的分片文件夹标号（取余的数组）
4. `Unassigned`：集群最小结构是2个，如果启动一个，认为另一个没有启动。从分配虽然计算出来了，但是没有第二个节点时是不会分配写数据的





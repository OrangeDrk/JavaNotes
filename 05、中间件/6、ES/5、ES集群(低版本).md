[TOC]

# ES集群的结构

## ES集群搭建逻辑

和redis、mycat数据库集群有区别。ES是一个可以启动的web应用，封装了扩展的功能，可以通过发现节点的==协调器==将一个集群所有的启动节点（在同一个网络中）自动添加到一起，实现集群的功能



## 节点角色

1. ==master==：整个集群的大脑，管理集群所有的元数据(meta-data)信息
   - ==元数据==：记录数据信息的数据
   - master管理的元数据：哪些索引、多少索引分片、多少分片副本、分片存放的位置、副本存放的位置等
   - master可以修改元数据，集群中唯一一个可以有权限修改元数据的角色，其他节点角色要想使用元数据，只能从master同步获取
2. ==data==：数据节点，用来读写数据。分配读任务，就读数据；分配写任务，就写数据。存储的就是索引的分片数据
3. ==ingest==：对外访问的连接节点
4. ==协调器==：众多的节点、角色，要想按照运行逻辑完美的整合添加到一个集群中，就需要协调器的加入



## 结构图

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES集群结构图.png)



# ES集群的搭建

## 准备ES节点

集群中有3个ES节点，分别在不同的服务器

- 节点名：es01,es02,es03
- 集群名称相同：elasticsearch
- 全部安装IK分词器



## 修改`elasticsearch.yml`

### 协调器

> 三个节点，每个节点的角色取决于配置文件中node值
>
> - node.master:true
> - node.data:true
> - node.ingest:true
>
> 默认情况下，每一个ES节点都默认有这三个配置，每个节点都可以是三个角色

协调器：任意一个ES节点都可以配置为当前集群的协调器。保证协调器不会出现宕机无法使用，一般这里都需要配置多个协调器。【从3台ES节点选择2个ES节点作为协调器即可】

> 69行：配置协调器ip地址

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES集群配置协调器.png)



### 最小有效master

> 73行，配置最小有效master数，防止==脑裂==的出现
>
> 最小有效master数：==（master总数/2）+1==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES集群配置最小master数.png)



> 脑裂：
>
> - 在一个ES集群中，现役master由于网络波动，导致集群判断它宕机，进而会进行一轮选举，出现新的现役master。
> - 当网络波动恢复后，整个集群中存在了2个master，每个master中有一个meta数据，导致集群数据混乱，使整个集群停止工作

![](https://gitee.com/sxhDrk/images/raw/master/imgs/脑裂.png)

> ES采用配置==过半集群有效master数量==，防止脑裂的出现
>
> ==出现网络波动后，无论如何，集群中保证有一个有效的master==



## 启动

把每个服务器的ES节点启动

- 切换到普通用户：`su es`
- 到`elasticsearch/bin`目录执行`elasticsearch`命令启动

通过插件可以看到一个分布式的高可用的ES集群

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES集群启动.png)

> 启动之后，数据会经过ES计算，实现迁移，将分片分部到多个节点的分布式集群中（可以配置分片和副本数量，适当调整集群的分布式和高可用）
>
> - 分片数量调高：一个集群就扩展更过的节点
> - 副本分片数量调高：支持更高的高可用





# 选举逻辑

> 集群运行过程中，选举逻辑一直存在，只要协调器不全部宕机，集群就可以正常执行选举逻辑

1. 每个节点（具备node.master:true）启动时都会连接ping通协调器（任何一个都可以），获取集群当前的状态。其中有个对象：`activeMaster`保存了现役master的信息。【进入第2步】
2. 判断`activeMaster`是否为空，
   - 空：表明没有选举完成，进入【第3步】
   - 不为空：说明已经选举完成，并选举出一个master，**选举逻辑结束**
3. 当前节点将集群中所有的master节点都保存到一个备选人list中：`candidata{es01,es02,...}`。【进入第4步】
4. 判断备选人list里的master节点数量是否满足最小master数量
   - 满足：从中比较节点Id，较大的为现役master，并将该master信息存放在`activeMaster`中。
   - 不满足：返回第一步，重新执行逻辑



> 节点宕机重新选举：
>
> - ==现役master宕机==（此时`activeMaster`为空）：剩余master重新执行选举逻辑
> - ==备选master宕机==：只要剩余master满足最小master数量，就不影响集群使用



# ES节点内存不够

1. ES进程的jvm默认判断需要2G的内存
2. 修改配置文件`elasticsearch/config/jvm.options`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/修改jvm最低内存.png)







# Java客户端代码

> 理论上来讲，通过http协议的访问操作ES，也就可以使用RestTemplate操作

TransportClient的Java客户端：包装了TCP/IP协议，可以通过访问ES节点的9300端口，实现通过API操作ES



## 依赖资源

> 不能和Lucene6.0.0的依赖在同一个系统中
>
> ES的依赖中包含了Lucene 5.5.2，版本不一样会冲突

```xml
<!--elasticsearch 核心包-->
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>5.5.2</version>
</dependency>

<!--transportclient-->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>transport</artifactId>
    <version>5.5.2</version>
</dependency>
```



## 测试代码

### 索引管理

> 增加索引、删除索引、判断索引是否存在等

```java
@Test
public void indexManage(){

    //通过client拿到索引管理对象
    AdminClient admin = client.admin();

    // ClusterAdminClient cluster = admin.cluster();
    IndicesAdminClient indices = admin.indices();

    //TransportClient中有2套方法，一套直接发送调用
    //一套是预先获取request对象。
    CreateIndexRequestBuilder request1 = indices.prepareCreate("index01");//不存在的索引名称

    IndicesExistsRequestBuilder request2 = indices.prepareExists("index01");
    boolean exists = request2.get().isExists();
    if(exists){
        System.out.println("index01已存在");
        return;
        //indices.prepareDelete("index01").get();
    }

    //发送请求request1到es
    CreateIndexResponse response1 = request1.get();

    //从reponse中解析一些有需要的数据
    System.out.println(response1.isShardsAcked());//json一部分 shards_acknowleged:true
    System.out.println(response1.remoteAddress());
    response1.isAcknowledged();//json acknowledged:true
}
```



### 文档管理

> 向ES集群中新增、查询、删除文档

> 新增

```java
@Test

public void documentManage() throws JsonProcessingException {

    //新增文档，准备文档数据

    //拼接字符串 对象与json的转化工具ObjectMapper
    Product p=new Product();
    p.setProductNum(500);
    p.setProductName("三星手机");
    p.setProductCategory("手机");
    p.setProductDescription("能攻能守，还能爆炸");
    ObjectMapper om=new ObjectMapper();
    String pJson = om.writeValueAsString(p);

    //client 获取request发送获取response
    //curl -XPUT -d {json} http://10.9.104.184:9200/index01/product/1
    IndexRequestBuilder request = client.prepareIndex("index01", "product", "1");

    //source填写成pJson
    request.setSource(pJson);
    request.get();
}
```



> 查询

```java
@Test

public void getDocument(){

    //获取一个document只需要index type id坐标
    GetRequestBuilder request = client.prepareGet("index01", "product", "1");
    GetResponse response = request.get();

    //从response中解析数据
    System.out.println(response.getIndex());
    System.out.println(response.getId());
    System.out.println(response.getVersion());
    System.out.println(response.getSourceAsString());
    client.prepareDelete("index01","product","1");
}
```



### 搜索功能

> 搜索索引文件中的文档信息

```java
@Test
public void search(){
    //其他查询条件也可以创建
    MatchQueryBuilder query = QueryBuilders.matchQuery("productName","三星手机");
    //封装查询条件，不同逻辑查询条件对象不同
    //TermQueryBuilder query = QueryBuilders.termQuery
    // ("productName", "星");
    //底层使用lucene查询，基于浅查询，可以支持分页
    SearchRequestBuilder request = client.prepareSearch("index01");

    //在request中封装查询条件，分页条件等
    request.setQuery(query);
    request.setFrom(0);//起始位置 类似limit start
    request.setSize(5);
    SearchResponse response = request.get();

    //从response对象中解析搜索的结果
    //解析第一层hits相当于topDocs
    SearchHits searchHits = response.getHits();
    System.out.println("总共查到："+searchHits.totalHits);
    System.out.println("最大评分："+searchHits.getMaxScore());

    //循环遍历的结果
    SearchHit[] hits = searchHits.getHits();//第二层hits

    //hits包含想要的查询结果
    for (SearchHit hit:hits){
        System.out.println(hit.getSourceAsString());
    }
}
```

> 双层hits如下图👇👇👇👇👇👇👇👇

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES查询所有.png)



# ELK家族

## 搜索系统结构

1. 数据整理封装，放到ES的索引文件中保存

2. 客户端系统访问ES封装的请求中，携带了query对象的查询参数

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/搜索系统结构.png)

> 通过业务逻辑考虑数据一致性，数据库和索引文件数据要一致。
>
> 数据库的数据源可能发生各种数据变动，要考虑到ES中索引文件的一致性



## ELK家族

> 基于ElasticSearch的全文检索功能，形成一整套技术体系。
>
> - 包含了数据导入、数据更新
> - 包含了全文检索数据管理
> - 包含了数据视图化展示和分析

E：ElasticSearch----有大量的数据（index）

L：logstash----日志数据收集器，将各种数据源的数据导入ES中

K：kibana----试图展示，数据分析工具

> 这样一个结构，使得存储在ES中的数据可以实时的更新，并且除了提供搜索以外，还可以实现更高的数据分析的价值

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ELK家族.png)





# 整合Springboot

> TransportClient整合到springboot，实现搜索功能

1. 准备配置类。注解：@Configuration

2. bean方法@Bean初始化一个TransportClient对象

3. @ConfigurationProperties("es") 读取配置文件属性

   ```properties
   es.nodes=10.42.0.33,10.42.24.13,10.42.104.102
   ```

   
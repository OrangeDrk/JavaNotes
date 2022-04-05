[TOC]

# 索引管理



## 新增索引

> ES根据请求方式PUT，根据请求url地址中资源定义的indexName为索引名称，在ES中创建indexName的索引
>
> `curl -XPUT http://10.42.0.33:9200/{indexName}`

```sh
curl -XPUT http://10.42.0.33:9200/book
```

成功后返回响应

```json
{
    "acknowledged": true, //表示确认成功
    "shards_acknowledged": true //分片创建成功
}
```



## 删除索引

> 提供索引名称，使用delete请求，将索引数据从ES中删除
>
> `curl -XDELETE http://10.42.0.33:9200/{indexName}`

```sh
curl -XDELETE http://10.42.0.33:9200/book
```

成功后返回响应：

```json
{
  "acknowledged": true
}
```



## 查看索引

> 在ES中保存的索引文件，都会对应有各种状态存储对象信息，可以通过GET查询将这些信息返回。
>
> `aliases`别名、`mapping`映射、`settings`配置
>
> `curl -XGET http://10.9.104.184:9200/{indexName}`

```sh
curl -XGET http://10.9.104.184:9200/book
```

成功后返回响应：

```json
{
    "index01": {
        "aliases": {}, //别名，可以通过别名操作索引
        "mappings": {},//默认是空，一旦数据新增，mapping将会出现动态结构
        "settings": { //索引的各种配置信息
            "index": {
                "creation_date": "1582594169179",
                "number_of_shards": "5",
                "number_of_replicas": "1",
                "uuid": "sDnRylu6RXu2INuyDXctbA",
                "version": {
                    "created": "5050299"
                },
                "provided_name": "index01"
            }
        }
    }
}
```



## 关闭/打开索引

1. 关闭

   `curl -XPOST http://10.42.0.33:9200/{indexName}/_close`

2. 打开

   `curl -XPOST http://10.42.0.33:9200/{indexName}/_open`

> 索引文件在ES中创建后，会存在保存状态的持久化文件中，其中一个状态是：打开/关闭。
>
> 一旦将索引文件进行关闭操作，这个索引文件就不能再使用，直到再被打开

## 读写权限限制

> 可以对索引实现不能读的权限

```sh
curl -XPUT -d '{"blocks.read":true}' http://10.42.0.33:9200/{indexName}/_settings
```

`blocks.read:true`：阻止读

`blocks.read:false`：不阻止读



# 文档管理

> http协议访问ES时，新增文档数据操作时，一定要保证url的JSON字符串结构正确



## 新增(覆盖)文档

> 在已存在的索引中，通过url指定索引、类型和文档id，将文档数据作为请求体参数发送到ES中，进行处理

```sh
curl -XPUT -d '{"id":1,"title":"ES简介","content":"ES很好用"}' http://10.42.0.33:9200/{indexName}/{typeName}/{_docId}
```

成功后返回响应:

```json
//document对象的基础属性
{
    "_index": "book", //所属索引
    "_type": "article", //所属类型
    "_id": "1", //document的id值
    
    //版本号
    //随着对document的操作执行，
    //_version会自增处理。在分布式高可用的集群中，
    //用来判断哪些数据可用，哪些延迟
    "_version": 1, 
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "created": true
}
```



## 获取文档

> 使用GET请求，获取指定索引、类型、docId的文档数据

```sh
curl -XGET http://10.9.104.184:9200/{indexName}/{typeName}/{_docId}
```

成功后返回响应

```json
{
    "_index": "book",
    "_type": "article",
    "_id": "1",
    "_version": 3,
    "found": true,
    "_source": {  //source表示数据内容
        "id": "doc1",
        "title": "java编程思想",
        "content": "这是一本开发的圣经"
    }
}
```





## 删除文档

> 删除指定的索引名称、类型、docId的文档数据
>
> ==即使删除了数据，该数据的`_version`也会自增1==

```sh
curl -XDELETE http://10.9.104.184:9200/{indexName}/{typeName}/{_docId}
```

成功后返回响应

```json
{
    "found": true,
    "_index": "book",
    "_type": "article",
    "_id": "1",
    "_version": 4,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    }
}
```



# 搜索功能

ES是基于Lucene的，本身也支持大量搜索功能：词项、布尔、多域、模糊等等。

使用http协议访问ES进行搜索时，只需要将JSON结构传递到不同的query实现类中即可

## match_all

> 查询所有

```sh
curl -XGET http://10.42.0.33:9200/index01/_search -d '{"query":{"match_all":{}}}'
```

返回查询结果：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/ES查询所有.png)

## term_query

> 词项查询，查询的基础方式
>
> ==通过词项查询可以看到一些现象，比如：能看出生成document时使用的什么分词器==

```sh
curl -XGET http://10.9.104.184:9200/index01/_search -d '{"query":{"term":{"title":"java"}}}'
```

> 👆👆👆👆可以搜索到：java编程思想，这本书

```sh
curl -XGET http://10.9.104.184:9200/index01/_search -d '{"query":{"term":{"title":"编程"}}}'
```

> 👆👆👆👆搜不到：Java编程思想，这本书
>
> 得到：这个document在生成时，分词器没有分出编程、思想



结论：

在Lucene中，封装document时，给每一个属性都赋值了很多参数，使用writer时也绑定了分词器。但是ES中这些内容都看不到

- ES中，document的属性对应类型，对应存储，对应分词计算
- 分词器默认是：`standardAnalyzer`
- 自定义分词器：在创建索引文件时传入mapping参数，指定分词器



## match_query

> 在搜索功能中常用的一个查询对象。

```sh
curl -XGET http://10.9.104.184:9200/index01/_search -d '{"query":{"match":{"title":"java编程思想"}}}'
```

> 可以对文本做分词计算，默认也是使用的`standardAnalyzer`，会将匹配到的所有document都返回
>
> 例如：java编程思想———>java，编，程，思，想
>
> - 查询所有的词项：
>   - title：java
>   - title：编
>   - title：程
>   - title：思
>   - title：想





# ES的mapping和IK分词器

## 安装IK分词器

> 在集群中，所有的ES节点配置都要保持一致，IK分词器要在每一台云主机的ES中安装

1. 将IK插件包解压到ES的`plugins`目录下

   ```sh
   [root@10-9-104-184 resources]# unzip /home/resources/elasticsearch-analysis-ik-5.5.2.zip -d /home/software/elasticsearch-5.5.2/plugins/
   ```

2. 测试IK分词器

   - 重启ES

   - 测试使用IK分词器计算文本

     ```sh
     curl -XPOST http://10.42.0.33:9200/_analyze -d '{"analyzer":"ik_max_word","text":"JAVA编程思想"}' 
     ```

     浏览器测试，通过浏览器url计算分词

     > 参数
     >
     > - analyzer：分词器名称
     > - text：测试的文本

     ```
     http://10.9.104.184:9200/index01/_analyze?analyzer=ik_max_word&text=中华人民共和国
     ```

     



## mapping映射

> mapping可以设置一个索引下的document的各种不同域属性的详细类型、不同属性使用的分词器



### 动态mapping

> 根据document中新增时携带的数据，决定对应的各种field域属性的类型，和分词器的使用

新建的索引文件，mapping默认是空的，直到新增一个document才会出现mapping结构，这就是动态mapping

```sh
[root@10-9-104-184 logs]# curl -XGET http://10.9.104.184:9200/index02/_mappings
########结果#############
{"index02":{"mappings":{}}} 
```



> 新增一条document数据，动态mapping会生成一些数据，实现document在该索引的映射

```json
{
    "index02": {
        "mappings": {
            "article": {//表示mapping中的类型
                "properties": {//类型的属性
                    "content": {//域名称
                        "type": "text",//域的类型相当于TextField
                        "analyzer": "standard",
                        "fields": {//对当前域扩展
                            "keyword": {
                                "type": "keyword",//keyword相当于StringField
                                "ignore_above": 256//虽然默认对字符串做整体存储，但是认为一旦长度超过256字节，就超过整体存储的默认要求
                            }
                        }
                    },
                    "id": {
                        //域属性"type": "long"//域类型
                    },
                    "title": {
                        //域属性"type": "text",
                        //相当于TextField"fields": {
                        "keyword": {
                            "type": "keyword",
                            "ignore_above": 256
                        }
                    }
                }
            }
        }
    }
}
}
```



### 静态mapping

> 手动设置静态mapping
>
> 在手动创建索引的时候，就将自定义生成的静态mapping结构一并的添加进去

```sh
curl -XPUT 
	-d '{"mappings":{"article":{"properties":{"id":{"type":"long"},"title":{"type":"text","analyzer":"ik_max_word"},"content":{"type":"text","analyzer":"ik_max_word"}}}}}' 
	http://10.9.104.184:9200/index03/
```

👇👇👇👇👇👇自定义的mapping结构👇👇👇👇👇

```json
{
  "mappings": {
    "article": {
      "properties": {
        "id": {
          "type": "long"
        },
        "title": {
          "type": "text",
          "analyzer": "ik_max_word"
        },
        "content": {
          "type": "text",
          "analyzer": "ik_max_word"
        }
      }
    }
  }
}
```



> 测试数据

1. 新增一条document数据

   ```sh
   curl -XPUT 
   	-d '{"id":1,"title":"es简介","content":"es好用好用真好用"}' 
   	http://10.9.104.184:9200/index03/article/1
   ```

   

2. 使用term_query查询

   ```sh
   curl -XGET 
   	-d '{"query":{"term":{"title":"编程"}}}'
   	http://10.9.104.184:9200/index03/_search 
   ```

   > 如果返回数据，说明该数据的分词词项有编程2个字，IK分词器生效







# ES使用的命令

```sh
############索引管理###############
#1新建索引
curl -XPUT http://10.42.0.33:9200/index01

#2 读写权限
curl -XPUT -d '{"blocks.read":false}' http://10.42.0.33:9200/index01/_settings

#3 查看索引
#单个
curl -XGET http://10.42.0.33:9200/index01/_settings
#多个
curl -XGET http://10.42.0.33:9200/index01,blog/_settings

#4 删除索引
curl -XDELETE http://10.42.0.33:9200/index02

#5打开关闭索引
#关闭
curl -XPOST http://10.42.0.33:9200/index01/_close
#打开
curl -XPOST http://10.42.0.33:9200/index01/_open

#多个
curl -XPOST http://10.42.0.33:9200/index01,blog,index02/_close
curl -XPOST http://10.42.0.33:9200/index01,blog,index02/_open

####################文档管理#################
#1新建文档
curl -XPUT -d '{"id":1,"title":"es简介","content":"es好用好用真好用"}' http://10.42.0.33:9200/book/article/1

#2 获取文档
curl -XGET http://10.42.0.33:9200/index01/article/1

#3 获取多个文档
curl -XGET  -d '{"docs":[{"_index":"index01","_type":"article","_id":"1"},{"_index":"index01","_type":"article","_id":"2"}]}' http://10.42.0.33:9200/_mget

#4删除文档
curl -XDELETE http://10.42.0.33:9200/index01/article/1

###################搜索###################
#1 查询所有文档
#准备一些文档数据

curl -XPUT -d '{"id":1,"title":"es简介","content":"es好用好用真好用"}' http://10.42.0.33:9200/index01/article/1
curl -XPUT -d '{"id":1,"title":"java编程思想","content":"这就是个工具书"}' http://10.42.0.33:9200/index01/article/2
curl -XPUT -d '{"id":1,"title":"大数据简介","content":"你知道什么是大数据吗，就是大数据"}' http://10.42.0.33:9200/index01/article/3

#2 match_all
curl -XGET http://10.42.0.33:9200/index01/_search -d '{"query": {"match_all": {}}}'

#3 term query
curl -XGET http://10.42.0.33:9200/index01/_search -d '{"query":{"term":{"title":"java"}}}'
curl -XGET http://10.42.0.33:9200/index01/_search -d '{"query":{"term":{"title":"java编程思想"}}}'
curl -XGET http://10.42.0.33:9200/jtdb_item/_search -d '{"query":{"term":{"title":"双卡双"}}}'

#4 match query
curl -XGET http://10.42.0.33:9200/index01/_search -d '{"query":{"match":{"title":"java编程思想"}}}'

#logstash启动
logstash -e 'input{stdin{}}output{stdout{codec=>rubydebug}}'


#IK分词器
curl -XPOST http://10.42.0.33:9200/_analyze -d '{"analyzer":"ik_max_word","text":"JAVA编程思想"}'
http://10.42.0.33:9200/index01/_analyze?analyzer=ik&text=%E4%B8%AD%E5%8D%8E%E4%BA%BA%E6%B0%91%E5%85%B1%E5%92%8C%E5%9B%BD
curl 

#IK分词器
curl -XPUT -d '{"id":1,"kw":"我们都爱中华人民共和国"}' http://10.42.0.33:9200/haha1/haha/1

#查看mapping
curl -XGET http://10.42.0.33:9200/jtdb_item/tb_item/_mapping

curl -XPUT -d '{"mappings":{"article":{"properties":{"id":{"type":"keyword"},"title":{"type":"text"},"content":{"type":"text"}}}}}' http://10.42.0.33:9200/book/

```


[TOC]



# Lucene概括

## 定义

属于全文检索技术之一。

Lucene是全文检索技术的==工具包==

利用Lucene可以实现快速方便的开发全文检索功能。在没有Lucene这种工具包之前，就已经存在了全文检索技术。

具体实现：创建索引、搜索索引等。不同的技术实现思路不一样



## 和Hadoop的关系

创始人都是Doug Cutting（国内称他：狗哥）

2000年左右研发Lucene，后来研发了Hadoop。

他利用Lucene开发了nutch搜索引擎，但是卡在海量数据存储的问题上，所以之后引入了Hadoop第一个版本HDFS分布式文件系统



## Lucene特点

基于Java原生语言开发

1. 索引文件创建速度快
2. 占用占内存空间小
3. 支持多种丰富的搜索功能



# 索引文件结构

Lucene创建的索引，使用了==倒排索引==算法，在索引文件中除了数据（网页、数据库数据整理的内容），还保存了很多关系表。索引文件的创建过程，需要对数据、字、段落进行加工

## 分词（analyze）：

将一个文本数据切分成更小单位的字、词、段、句等，形成大量分词结果（词项---term）存储在索引文件中，用来保存词项与数据的关系

> 例如：
>
> - 源数据：中华人民共和国
> - 分词计算：
>   - 字处理：中，华，人，民，共，和，国
>   - 词处理：中华，华人，人民，中华人民，人民共和国
>   - 句处理：中华人民共和国

## 文档（document）：

整理后存储在索引文件中的数据单位。对应源数据一条或者一页，一个整体的内容。

> 例如：
>
> - 海量的网页，每个页面在索引文件中对应一个文档对象
> - 日志中每一行日志，对应索引文件中的一个文档对象，只要根据不同的数据源给文档对象定义各种不同属性



# 倒排索引计算步骤

## 数据源读取

> 数据源来源很多，可以是公网网页资源，可以是数据库的数据等。。

1. web1：
   - 标题：疫情到达拐点
   - 内容：科学家预测月底到达拐点
   - 出版社：新华社
2. web2：
   - 标题：疫情已经被控制住
   - 内容：全国新增病例呈下降趋势
   - 出版社：新华社



## 封装document文档对象

> 将收集的==内容封装到document对象中，进行数据清洗==（不需要的内容就不封装到document对象中）

1. document1封装web1
   - id：1
   - title：疫情到达拐点
   - content：科学家预测月底到达拐点
   - publisher：新华社
2. document2封装web2
   - id：2
   - title：疫情已经被控制住
   - content：全国新增病例呈下降趋势
   - publisher：新华社



## 计算分词

> 对词的加工。

每一个分词计算结果（term）都可以携带很多计算过程出现的参数，比如：

- 分词term所在的document对象的id
- 出现的频率
- 出现的位置

> 分词[term]（documentId，属性名）

- doc1:
  - 疫情（1，title）
  - 达到（1，title）
  - 科学家（1，content)
  - 拐点（1，content）
  - 新华社（1，publisher）
- doc2:
  - 疫情（2，title）
  - 控制（2，title）
  - 全国（2，content）
  - 趋势（2，content）
  - 新华社（2，publisher）



## 合并分词

> 合并分词，形成==倒排索引表==

| 词项\doc | doc1(1) | doc2(2) | doc3(3) | doc4(4) |
| -------- | ------- | ------- | ------- | ------- |
| 疫情     | 1       | 1       | 0       | 0       |
| 达到     | 1       | 0       | 0       | 0       |
| 科学家   | 1       | 0       | 0       | 0       |
| 新华社   | 1       | 1       | 0       | 0       |
| 控制     | 0       | 1       | 0       | 0       |
| 全国     | 0       | 1       | 0       | 0       |
| 趋势     | 0       | 1       | 0       | 0       |

> 只要表格中记录了词项与document之间的关系，就能对应查出搜索的内容

比如：查询关键字：==全国疫情趋势==，分词后———全国，疫情，趋势

1. 查询==包含所有==3个词的document对象

   - 疫情：1100
   - 全国：0100
   - 趋势：0100

   > 查询到的索引值==按位与==运算，得到：0100
   >
   > 0100，左边开始第2位是1，说明表中：doc2匹配到了，doc2中存在这三个词

2. 查询==其中一个==词语的document对象

   - 疫情：1100
   - 全国：0100
   - 趋势：0100

   > 查询到的索引值==按位或==运算，得到：1100
   >
   > 1100，左边开始，第1位，第2位是1，说明表中：doc1，doc2匹配到了，doc1，doc2中存在三个词中的某一个词



## 输出到索引文件

> 将倒排索引表和document数据一并输出到索引文件中，形成了能快速定位查询的数据结构

![](https://gitee.com/sxhDrk/images/raw/master/imgs/索引文件.png)

> 索引文件抽象图解：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/实现了analyzer的IK实现类.png)





# Lucene分词器

在我们要使用的数据源中，形成最终的索引文件的过程。必须要经过数据的分词计算。

数据源中的文本内容是非常丰富的。语言，德文，俄文，法文，意大利文，英文，中文，吐火罗文。底层二进制都是不一样的编码。使用同一个分词计算的原理不能实现对所有文本数据进行分词计算的需求。lucene不可能提供具体详细的分词计算逻辑。所以lucene对于分词提供了一些基础的分词计算，同时提供了在lucene使用创建分词计算的规范----接口`Analyzer`，可以在不满足需求的情况下自定义接口的实现类完成对应分词计算。



## Lucene提供的分词器

> 可以处理字、词、句、段的一些分词器

- `standardAnalyzer`：标准分词器，==按字==切分文本
  - 中华人民：中，华，人，民
  - who are you：who，are，you
- `simpleAnalyzer`：简单分词器，==按标点，空格，回车==切分文本
- `whitespaceAnalyzer`：空格分词器，==按空格==切分文本
- `smartChineseAnalyzer`：智能中文分词器，==按中文==分词
  - 我们都是朋友：我们，都是，朋友





## 引入依赖

```xml
<dependency> <!-- 查询相关jar包 -->
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-queryparser</artifactId>
    <version>6.0.0</version>
</dependency>
<dependency> <!-- lucene自带智能中文分词器jar包 -->
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-analyzers-smartcn</artifactId>
    <version>6.0.0</version>
</dependency>
<dependency> <!-- 测试用到的lucene工具包 -->
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-analyzers-common</artifactId>
    <version>6.0.0</version>
</dependency>
<dependency> <!-- 测试用到的lucene核心包 -->
    <groupId>org.apache.lucene</groupId>
    <artifactId>lucene-core</artifactId>
    <version>6.0.0</version>
</dependency>
```



## 测试代码

```java
/*
    测试方法，实现对一个字符串的使用分词器实现类解析
    的过程，不需要关心分词计算底层，只要使用实现类
    将分词结果打印出来 看到词，字，段的文字
 */
//org.apache.lucene.*
public void analyze(Analyzer analyzer,String msg)throws Exception{
    //将字符串对象，转化成流数据
    StringReader reader= new StringReader(msg);

    //使用分词器对象，获取切分后的流数据
    TokenStream stream = analyzer.tokenStream("test", reader);

    //二进制计算完分词，指针已经指向末尾，需要重置指针位置
    stream.reset();

    //查看stream中的各个切分之后的词项的文本属性，打印，拿到文本属性
    CharTermAttribute attribute = stream.getAttribute(CharTermAttribute.class);

    //循环遍历每一个二进制，打印文本属性
    while(stream.incrementToken()){
        System.out.println(attribute.toString());
    }
}

//编写测试代码，提供多个不同的实现类对象
@Test
public void run()throws Exception{
    //准备一个字符串
    String msg="我们的课程,还要更深入的,研究疫苗形成";

    //分词器实现类
    Analyzer a1=new StandardAnalyzer();
    Analyzer a2=new SimpleAnalyzer();
    Analyzer a3=new WhitespaceAnalyzer();
    Analyzer a4=new SmartChineseAnalyzer();

    System.out.println("************标准**********");
    analyze(a1,msg);

    System.out.println("************简单**********");
    analyze(a2,msg);

    System.out.println("************空格**********");
    analyze(a3,msg);

    System.out.println("************智能**********");
    analyze(a4,msg);
}
```



## 运行结果

```
****************标准****************
我
们
的
课
程
还
要
更
深
入
的
研
究
疫
苗
形
成
****************简单****************
我们的课程
还要更深入的
研究疫苗形成
****************空格****************
我们的课程,还要更深入的,研究疫苗形成
****************智能****************
我们
的
课程
还要
更
深入
的
研究
疫苗
形成
```









# IK中文分词器

即使使用像`smartChineseAnalyzer`这样的中文智能分词器对象，也很难满足需求，因为分词词语都是固定的，不能随着发展而变化。

随着网络发展，新词语出现的速度、频率大大增加，分词器必须支持词语的发展变化

常用的中文分词器中，IK分词器基于==词典==实现的分词计算。可以通过对词典的编辑、扩展、停用和启用，去灵活的设置分词器

- 特点：
  - 可以停用一些无效的词语分词：一个、我们、好多......
  - 可以添加一些新词语



## 引入依赖

> IK分词器的依赖在中央库中不存在，需要引入本地依赖

```xml
<!--本地依赖-->
<dependency>
    <groupId>cn.tedu.ik</groupId>
    <artifactId>ik-analyzer</artifactId>
    <version>1.0</version>
    <!--引入本地依赖-->
    <scope>system</scope>
    <!--配置本地jar包路径-->
    <systemPath>E:/IKAnalyzer2012_u6.jar</systemPath>
</dependency>
```

pom文件中`<dependency>`标签中的`scope`标签，表示当前依赖的使用范围

- `compile`：编译、运行、测试、打包、安装（默认值）
- `test`：只在测试时生效，运行、打包、安装、发布都不会使用这个jar包
- `runtime`：和compile一样，编译时不需要，运行、测试、打包、安装、发布都会存在
- `provided`：运行最后打包安装部署时不使用，编译时检查（servlet-api）
- `system`：使用本地jar包，配合`<systemPath>`引入本地依赖



## 准备实现类

> 将已经实现了analyzer的IK实现类拷贝到当前工程中

![](https://gitee.com/sxhDrk/images/raw/master/imgs/创建的索引文件示意图.png)



## 准备IK分词器词典

1. `IKAnalyzer.cfg.xml`：配置词典的配置文件，IK分词器需要加载到这个文件

   > 将该文件放到`resources`目录下

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">  
   <properties>  
       <comment>IK Analyzer 扩展配置</comment>
       
       <!--用户可以在这里配置自己的扩展字典-->
       <entry key="ext_dict">ext.dic;</entry> 
   
       <!--用户可以在这里配置自己的扩展停止词字典-->
       <entry key="ext_stopwords">stopword.dic;</entry> 
   
   </properties>
   ```

2. `**.dic`：IK分词器可以配置的各种词典（==词典和配置文件同级目录==）

   - `ext.dic`：扩展词典

     > 扩展词典中的内容，会在分词时被识别，单独分出来

     ```
     王者荣耀
     ```

   - `stopword.dic`：停用词典

     > 停用词典中的内容，会在分词时被识别，并且不会被分出来

     ```
     妹妹
     ```



## 代码测试

```java
public class AnalyzerTest {
    public void analyze(Analyzer analyzer,String msg) throws IOException {
        //将字符串转为流数据
        StringReader reader = new StringReader(msg);
        TokenStream stream = analyzer.tokenStream("test", reader);
        stream.reset();
        //查看stream中各个切分之后的词项的文本属性
        CharTermAttribute attribute = stream.getAttribute(CharTermAttribute.class);

        while (stream.incrementToken()) {
            System.out.println(attribute.toString());
        }
    }

    @Test
    public void run() throws IOException {
        String mg="我和妹妹打王者荣耀";

        Analyzer a5 = new IKAnalyzer6x();

        System.out.println("****************IK****************");
        analyze(a5,mg);
    }
}
```



## 运行结果

> 因为配置文件添加了扩展词典，所以把“王者荣耀”这个词分了出来
>
> 还配置了停用词典，所以把“妹妹”这个词屏蔽掉了

```
****************IK****************
加载扩展词典：ext.dic
加载扩展停止词典：stopword.dic
我
和
打
王者荣耀
王者
荣耀
```





# Lucene实现创建索引



## 创建索引文件

> 全文检索技术围绕全文数据库的==索引文件==展开的使用
>
> 第一件事就是使用全文检索技术创建出来有数据的索引文件

### 步骤

1. 读取数据源（读取数据库、其他等）
2. 创建索引文件的输出目录
3. 生成输出流对象：`writer`，指定输出时计算分词的分词器
4. 封装document对象
5. 将document对象输出到索引，形成一个完整的索引文件

```java
/**
 * 模拟Lucene创建索引，封装数据源的过程
 */
public class CreateIndex {
    @Test
    public void createIndex() throws IOException {
        //模拟-->读取数据源
        Product p1 = new Product();
        p1.setProductId("1");
        p1.setProductCategory("手机");
        p1.setProductDescription("3亿像素高清摄影，照亮景色的美");
        p1.setProductName("达内PN3 Pro手机");
        p1.setProductNum(500);

        Product p2 = new Product();
        p2.setProductId("2");
        p2.setProductCategory("电脑");
        p2.setProductDescription("游戏发烧级的选择");
        p2.setProductName("达内游戏杀手");
        p2.setProductNum(200);

        //准备索引文件的输出目录
        Path path = Paths.get("E:/index01");
        FSDirectory dir = FSDirectory.open(path);

        //给writer输出流配置一个配置对象,指定分词器
        IndexWriterConfig config = new IndexWriterConfig(new IKAnalyzer6x());

        //定义创建索引的模式：覆盖/追加
        config.setOpenMode(IndexWriterConfig.OpenMode.CREATE);

        //创建输出流对象
        IndexWriter writer = new IndexWriter(dir, config);

        //封装document,从2个元数据p1,p2封装2个索引需要的document数据
        //数据要一致
        Document doc1 = new Document();
        Document doc2 = new Document();
        doc1.add(new TextField("productId",p1.getProductId(), Field.Store.YES));
        doc1.add(new TextField("productCat", p1.getProductCategory(), Field.Store.YES));
        doc1.add(new TextField("productDesc", p1.getProductDescription(), Field.Store.YES));
        doc1.add(new TextField("productName", p1.getProductName(), Field.Store.YES));
        doc1.add(new IntPoint("num", p1.getProductNum()));

        doc2.add(new TextField("productId",p2.getProductId(), Field.Store.YES));
        doc2.add(new TextField("productCat", p2.getProductCategory(), Field.Store.YES));
        doc2.add(new StringField("productDesc", p2.getProductDescription(), Field.Store.YES));
        doc2.add(new TextField("productName", p2.getProductName(), Field.Store.NO));
        doc2.add(new IntPoint("num", p2.getProductNum()));
        doc2.add(new StringField("num", p2.getProductNum()+"", Field.Store.YES));

        //将document封装到writer
        writer.addDocument(doc1);
        writer.addDocument(doc2);

        //创建索引
        writer.commit();//文件夹出现默认空环境，没有数据

    }

}

```



![](https://gitee.com/sxhDrk/images/raw/master/imgs/创建索引文件.png)

> 输出索引文件会生成这样几个文件：cfe，cfs，si后缀的存储数据的文件
>
> 对应的是：存储==很多的倒排索引表==和==存储的document对象==



## 使用luke软件打开索引文件

1. 选择索引文件夹

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/must.png)

2. 查看索引文件内容

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/luke软件-选择索引文件夹.png)



## 封装document细节

1. `TextField`和`StringField`的区别

   同是处理字符串数据

   - `TextField`：计算分词
   - `StringField`：不计算分词

   > 不是所有的字符串数据都要计算分词，例如：
   >
   > - uuid的主键值
   > - web网页链接地址

2. `Store.YES`和`Store.NO`的区别

   > ==是否在**索引文件**中保存该数据==（搜索出来的数据没有用到的时候就不需要在索引文件中保存起来）
   >
   > - 不保存，不等于不计算分词
   > - 不保存，依然可以计算分词，并且使用该分词也能查询，但是查询出来，也拿不到对应数据
   >
   > （==倒序索引表中存在该数据，document对象中没有该数据==）

   ```java
   p2.setProductName("达内游戏杀手");
   
   doc2.add(new TextField("productName", p2.getProductName(), Field.Store.NO));
   ```

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/luke软件-查看索引文件内容.png)

3. 整数类型数据

   > 数字类型InptPoint、LongPoint、DoublePoint、FloatPoint处理数字既不分词，也不存储在索引文件中，==仅仅用来实现范围数字搜索的特性==（搜索【200，500】之间的所有数据）
   >
   > （虽然没有添加到倒序索引表和document中，但是底层记录了该数字，可以用来实现范围搜索的特性）

   

   **如果说既要使用数字做范围搜索，又要拿到数字。可以使用StringField类型，添加一个同名的field到document中**

   



# Lucene实现搜索索引

> Lucene支持丰富的查询功能，其实是将不同的查询逻辑封装到了不同的query类中。通过对不同实现类的使用，查询索引文件中的数据

## 基本步骤

1. 指向一个索引文件

2. 获取输入流，构造一个搜索对象

3. 构造不同的查询条件

4. 利用Lucene的==浅查询==遍历结果，获取数据

   > ==浅查询==：
   >
   > - 搜索海量数据时，分页查询实现的一种计算逻辑。浅查询只会对数据的标识做计算，然后利用计算结果再去获取数据
   >
   > ==深查询==：
   >
   > - 深查询分页逻辑：分页最后一项数据之前的内容，全部读取出来，然后抛弃分页属于以外的内容（算法简单）

## 词项查询

> `TermQuery`，封装了一个词项（fieldName，termValue）
>
> 在索引文件中找到唯一对应的一个词项，通过词项找到对应的document数据

### 测试代码

```java
//词项查询，对域名，词项匹配，一旦匹配返回查询结果
@Test
public void termQuery() throws Exception {
    //获取索引文件通道，指向索引文件
    Path path = Paths.get("E:/index01");
    FSDirectory dir = FSDirectory.open(path);

    //获取输入流，构造一个搜索对象
    IndexReader reader = DirectoryReader.open(dir);
    IndexSearcher searcher = new IndexSearcher(reader);

    //创建一个搜索对象 TermQuery
    Term term = new Term("productName", "手机");
    Query query = new TermQuery(term);

    //浅查询，遍历结果
    //拿到对标识做了计算的数据 topDocs
    TopDocs topDocs = searcher.search(query, 10);//拿到前10条数据，doc id计算结果
    //doc id计算结果，从中解析初docId的对象数组
    ScoreDoc[] scoreDocs = topDocs.scoreDocs;
    for (ScoreDoc scoreDoc : scoreDocs) {
        //解析需要搜索的分页的id值
        System.out.println("docId:" + scoreDoc.doc);
        //通过id获取document对象
        Document doc = searcher.doc(scoreDoc.doc);
        //使用数据
        System.out.println("productName:" + doc.get("productName"));
        System.out.println("productCat:" + doc.get("productCat"));
        System.out.println("productDesc:" + doc.get("productDesc"));
        System.out.println("productNum:" + doc.get("num"));
    }
}
```



### 结果

```
docId:0
productName:达内PN3 Pro手机
productCat:手机
productDesc:3亿像素高清摄影，照亮景色的美
productNum:500个
```



## 多域查询

> `MultiFieldQuery`，可以将不同的field数据封装在同一个查询条件中，同时查询多个属性对应的document数据

### 测试代码

> `String[] fields={"productName","productCat","productDesc"};`
>
> 当使用一个字符串进行查询时，先进行分词计算：【 "达内手机顶呱呱"-->手机，顶呱呱】
>
> 排列组合
>
> - proudctName:手机，productName:顶呱呱
> - productCat:手机，productCat:顶呱呱
> - productDesc:手机，productDesc:顶呱呱
>
> 一旦任何一个词项匹配到了document，就将其放到返回结果中使用

```java
@Test
public void multiFieldQuery() throws Exception {
    //指向搜索数据索引文件
    Path path = Paths.get("d:/index01");
    FSDirectory dir=FSDirectory.open(path);

    //拿到搜索对象
    IndexReader reader=DirectoryReader.open(dir);
    IndexSearcher searcher=new IndexSearcher(reader);

    //构造查询条件
    //多个域
    String[] fields={"productName","productCat","productDesc"};

    //解析查询文本的解析器 需要使用分词器
    MultiFieldQueryParser parser=new MultiFieldQueryParser(fields,new IKAnalyzer6x());


    //提供一个查询关键字文字
    String text="达内手机顶呱呱";
    Query query = parser.parse(text);//向上造型多域查询

    //遍历循环。
    TopDocs topDocs = searcher.search(query, 10);
    ScoreDoc[] scoreDocs = topDocs.scoreDocs;
    for(ScoreDoc scoreDoc:scoreDocs){
        System.out.println("documentId:"+scoreDoc.doc);

        //获取document对象
        Document doc = searcher.doc(scoreDoc.doc);

        //使用数据，打印数据值
        System.out.println("productName"+doc.get("productName"));
        System.out.println("productCat"+doc.get("productCat"));
        System.out.println("productDesc"+doc.get("productDesc"));
        System.out.println("productNum"+doc.get("num"));
    }
}
```



### 结果

```
加载扩展词典：ext.dic
加载扩展停止词典：stopword.dic
documentId:0
productName:达内PN3 Pro手机
productCat:手机
productDesc:3亿像素高清摄影，照亮景色的美
productNum:500个
documentId:1
productName:null
productCat:电脑
productDesc:游戏发烧级的选择
productNum:200
```



## 布尔查询

> 可以在Lucene中实现非常多的不同查询功能，每一个都对应一个query对象，每个对象查询都可以对应一批结果集。可以利用布尔的逻辑，实现多个query的逻辑关系处理（交集、并集）。多个query就可以形成布尔查询的子条件去使用

### 测试代码

```java
@Test
public void booleanQuery() throws Exception {
    Path path = Paths.get("d:/index01");
    FSDirectory dir=FSDirectory.open(path);
    IndexReader reader=DirectoryReader.open(dir);
    IndexSearcher searcher=new IndexSearcher(reader);

    //构造查询条件
    //多个布尔子条件查询
    Query query1=new TermQuery(new Term("productName","手机"));
    Query query2=new TermQuery(new Term("productCat","手机"));

    //子查询封装成布尔的子条件
    BooleanClause bc1=new BooleanClause(query1,BooleanClause.Occur.MUST);
    BooleanClause bc2=new BooleanClause(query2,BooleanClause.Occur.MUST);

    //利用2个子条件封装布尔
    Query query=new BooleanQuery.Builder().add(bc1).add(bc2).build();

    //遍历循环。
    TopDocs topDocs = searcher.search(query, 10);
    ScoreDoc[] scoreDocs = topDocs.scoreDocs;
    for(ScoreDoc scoreDoc:scoreDocs){
        System.out.println("documentId:"+scoreDoc.doc);

        //获取document对象
        Document doc = searcher.doc(scoreDoc.doc);

        //使用数据，打印数据值
        System.out.println("productName"+doc.get("productName"));
        System.out.println("productCat"+doc.get("productCat"));
        System.out.println("productDesc"+doc.get("productDesc"));
        System.out.println("productNum"+doc.get("num"));
    }
}
```



### 结果

```
documentId:0
productName:达内PN3 Pro手机
productCat:手机
productDesc:3亿像素高清摄影，照亮景色的美
productNum:500个
```



### 布尔查询参数介绍

> `new BooleanClause(query1,BooleanClause.Occur.MUST);`
>
> `new BooleanClause(query2,BooleanClause.Occur.MUST);`
>
> Occur发生逻辑值
>
> - must:查询结果一定属于该子条件的一部分
> - must_not：查询结果一定不属于该子条件
> - should：查询结果可以包含，也可以不包含该解条件，MUST同时使用以must为准
> - filter：must相同的效果，但是使用filter的条件对应的结果，没有评分

1. must

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/Store.NO-YES的区别.png)

2. must---must_not

   ![](https://gitee.com/sxhDrk/images/raw/master/imgs/must-must_not.png)



## 模糊查询

```java
FuzzyQuery query=new FuzzyQuery(new Term("name","tramp"))；
```

可以利用模糊查询，实现对提供的参数进行模糊的对比数据

例如：

- 提供的数据：tramp
- 分词表中的数据：trump

中文的：日-曰；品-晶；又-双

## 通配符查询

| 通配符 | 含义       |
| ------ | ---------- |
| ？     | 表示字符   |
| *      | 表示字符串 |

`WildcartQuery query=new WildcartQuery(new  Term("name","王*"));`

> 当词项在name属性里有：王军、王翠花、王首富等都能被查询条件对比到
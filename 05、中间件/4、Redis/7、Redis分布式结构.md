[TOC]

# 1、分布式结构



![](https://gitee.com/sxhDrk/images/raw/master/imgs/redis分布式结构.png)

> 使用多个节点完成对系统数据的读写工作，==多个节点每个都处理了总数据量的一部分数据==---==数据分片==。

# 2、数据分片计算

![](https://gitee.com/sxhDrk/images/raw/master/imgs/数据分片计算.png)

> 大量的数据生成后，即将发送给redis节点进行处理之前，必须计算完毕，某一条数据应该交给谁，交给哪个节点去进行读写的工作。而且==要考虑单调性==
>
> ==单调性：存哪里，就要去哪里get数据==



# 3、hash取余



## 3.1、hash取余介绍

> 可以实现一个简单的分片计算逻辑--hash取余可以完成
>
> 能够实现大量数据的切分计算，交给不同节点管理维护数据，并且保证了单调性，任何一条key-value如果存储在节点A，必定也能实现读取时找到节点A



## 3.2、hash取余计算公式

> `(key.hashCode()&Integer.MAX_VALUE)%n`

### `key`：

表示redis要处理的key-value数据，key的取值范围就是字符串范围

### `key.hashCode()`：

> hashCode()方法，是Object类型的散列计算方法。

可以将任意一个内存java对象映射到对应的一个Integer整数范围区间。并且==只要对象不变（equals相等），整数值就不会发生变化，可正可负==

### `key.hashCode()&Integer.MAX_VALUE`：

> 利用`&`做位与运算，实现31位的保真运算，保证最后的数是正数
>
> ==保真运算：保证N位二进制和之前相同，其他的都是0==



**三位保真运算图解**

> `&`与运算，只要与`0`做与运算，结果都为`0`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/三位保真运算.png)

> 任意一个负数int类型，二进制特点 第一位一定是1，一共32位，一旦与最大正整数（31个1）做位的与运算，将会把第一位的1变成0.一定是个正整数。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/负数与最大正数做与运算.png)

> 一个整数与上最大正整数之后，一定是正整数。而且key值不变，正整数值不变。



### `n`：

就是数据分片的个数（redis结构中，就是redis的节点个数）



## 3.3、演示hash取余实现原理

> 单独开发一个对象 【MyShardedJedis】，创建这个对象时，给它包装一个底层所有redis节点集合，利用集合实现分片计算。
>
> 把jedis所有api重写一遍，就能实现利用这个MyShardedJedis实现分片计算的逻辑

1. 自定义分片对象

   ```java
   public class MyShardedJedis {
       //属性，list节点信息，保管了所有的集群节点内容
       //有三个节点 list元素3个，5个节点 list元素5个
       List<Jedis> nodes=new ArrayList<>();
       
       //构造方法，每次构造使用这个对象，必须传递所有节点的信息。
       public MyShardedJedis(List<Jedis> nodes){
           this.nodes=nodes;
           //nodes={new Jedis(6379),new Jedis(6380),new Jedis(6381)};
           //下标 取余结果0 1 2
       }
       
       //封装一个hash取余的算法，对任意key值，找到对应节点
       public Jedis keyAndNode(String key){
           //hash取余公式 name 取余 0
           int result=(key.hashCode()&Integer.MAX_VALUE)%nodes.size();
           return nodes.get(result);
       }
       
       //只要对jedis的所有操作key值的方法重新封装一遍，当前分片对象可以使用了
       //get set
       public String get(String key){
           //传进来的key对应了哪个节点
           Jedis jedis = keyAndNode(key);
           return jedis.get(key);
       }
       public void set(String key,String value){
           //传进来的key对应了哪个节点
           Jedis jedis = keyAndNode(key);
           jedis.set(key,value);
       }
       public boolean exists(String key){
           //传进来的key对应了哪个节点
           Jedis jedis = keyAndNode(key);
           return jedis.exists(key);
       }
       //重写其他的方法....
   }
   
   ```

   

2. 测试使用

   ```java
   //分片计算逻辑测试
   @Test
   public void test(){
       //创建一个分片对象，提供3个节点的信息
       List<Jedis> nodes=new ArrayList<Jedis>();
       nodes.add(new Jedis("10.9.104.184",6379));
       nodes.add(new Jedis("10.9.104.184",6380));
       nodes.add(new Jedis("10.9.104.184",6381));
       MyShardedJedis msj=new MyShardedJedis(nodes);
       
       //使用for循环模拟大量不同key-value数据的生成
       for(int i=0;i<100;i++){
           //每循环一次，模拟系统需要操作redis分布式集群处理的数据一次
           String key= UUIDUtil.getUUID()+i;
           String value="name"+i;
           msj.set(key,value);
           System.out.println(msj.get(key));
       }
   }
   ```

   



# 4、一致性hash

## 4.1、Java中Redis实现的对象

> **==Java中，Jedis已经实现了分片对象，可以直接使用实现一致性hash==**
>
> Redis实现的类：`ShardedJedis`，该类中存放节点信息的集合泛型为：`JedisShardInfo`

```java
@Test
public void test03(){
    List<JedisShardInfo> nodes = new ArrayList<>();
    //收集节点信息
    nodes.add(new JedisShardInfo("192.168.16.84", 6379));
    nodes.add(new JedisShardInfo("192.168.16.84", 6380));
    nodes.add(new JedisShardInfo("192.168.16.84", 6381));

    ShardedJedis sj = new ShardedJedis(nodes);
    for (int i = 0; i < 100; i++) {
        String key = UUIDUtil.getUUID();
        String value = i+"";
        sj.set(key, value);
        System.out.println(sj.get(key));
    }
}
```





## 4.2、hash取余存在的问题

> 当前环境中，使用redis的集群，做hash取余分布式数据切分计算，存在隐患。
>
> hash取余会造成集群扩容（加节点），缩容（减节点）时，数据未命中的概率较高。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/hash取余存在的隐患.png)

> 数据未命中的概率，会随着节点增多，提高。数据未命中概率越高，出现扩容后雪崩的概率就更高。
>
> 所以，在redis这种集群计算数据分片的环境中，不能光考虑单调性，也要考虑这种扩容缩容对单调性的影响。==可以使用一致性hash算法==



## 4.3、一致性hash的逻辑

### 介绍

1997年由麻省理工提出并创造的一种散列计算。核心结构就是散列计算，可以将任意数据经过散列计算后得到==【0——2^32-1(43亿)】==这样一个正整数的区间。我们称这样的一个==区间叫做hash环==，底层是二进制，只关心32位，所以【0-1=最大正整数】【 最大正整数+1=0】

<img src="https://gitee.com/sxhDrk/images/raw/master/imgs/hash环.png" style="zoom:50%;" />



### 节点映射

> redis中，包括其他技术中，启动的节点信息 `10.9.104.184:6379`可以作为字符串数据使用，不同的节点这个字符串不相同。可以利用这样的数据做散列计算得到节点对应hash环中的整数。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/一致性hash-节点映射.png)

### 数据key值的映射

> key值作为字符串也可以在hash换上做散列计算整数映射。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/一致性hash-数据key值的映射.png)

### 数据对应节点的关系

> 使用hash取余也好，使用一致性hash也好，都需要找到key对应的节点，并且保证key不变对应node节点也不变。
>
> 在hash环中，一致性hash算法利用整数，把key的整数做==顺时针寻找最近的节点==整数，得到对应存放节点

![](https://gitee.com/sxhDrk/images/raw/master/imgs/一致性hash-数据对应节点的关系.png)

> 上图中
>
> - 6379节点存储：key1,key2,key5
> - 6380节点存储：key3
> - 6381节点存储：key4
>
> 单调性保证：key值不变，节点不变，整数值不变，顺时针寻找的结果就不变，到哪存的就会到哪去取。



### 扩容时的情况

> 新加入的节点依然做hash散列映射到hash环中，key值映射不变，依然做顺时针寻找最近节点整数对应关系
>
> 所以当添加的节点越多时，反而不会造成大量的数据未命中。恰恰相反，环中存在的节点越多，在扩容缩容时对整体的数据影响越小

![](https://gitee.com/sxhDrk/images/raw/master/imgs/一致性hash扩容时的情况.png)



### 平衡性

> 当节点数量不多时，刚好两个映射整数靠的比较近，会==造成数据分片倾斜==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/一致性hash-分片倾斜.png)

> 一致性hash的平衡性的问题，引入了虚拟节点的概念。每一个真实节点并不是独立的计算到整数环中，使用每个真实节点，计算一批数据，作为虚拟节点

![](https://gitee.com/sxhDrk/images/raw/master/imgs/pool连接池.png)

> 引入的虚拟节点，也做整数映射，这时，key值依然做顺时针寻找最近节点的逻辑，找到虚拟节点就对应到该节点真实节点处理数据。
>
> 虚拟节点计算个数越多，hash环被真实节点切分的弧线越平均。并且可以利用虚拟节点的个数的比例，实现数据的对应比例的权重计算。
>
> 默认情况，Jedis实现一致性hash每个节点虚拟节点个数 ==160*weight==权重值。可以按照服务器性能，容量的比值，去分配不同权重，实现虚拟节点个数的计算比例，完成数据权重划分
>
> ==没有指定权重，按照默认的1:1分配==

```java
@Test
public void test03(){
    //创建一个可以实现分片计算的对象ShardedJedis
    //收集节点信息，交给分片对象
    List<JedisShardInfo> nodes=new ArrayList<>();
    //6379,6380 6381节点信息存到nodes中
    nodes.add(new JedisShardInfo("10.9.104.184",6379,500,500,5));//weigth=5
    nodes.add(new JedisShardInfo("10.9.104.184",6380));//weigth=1
    nodes.add(new JedisShardInfo("10.9.104.184",6381));//wegith=1
    //构造一个分片对象ShardedJedis
    ShardedJedis sj=new ShardedJedis(nodes);
    for(int i=0;i<1000;i++){
        String key=UUIDUtil.getUUID();
        String value=""+i;
        sj.set(key,value);
        System.out.println(sj.get(key));
    }
}
```



# 5、Jedis连接池

## 5.1、普通pool连接池

> 直接创建一个Jedis对象，使用api操作redis服务端。从实现功能角度没有什么问题，但是在代码中使用一个对象，需要使用完了销毁，性能比较低。
>
> 使用池的结构，创建一批可以连接redis节点的Jedis对象，使用就获取一个，用完了就还回一个，还可以根据应用场景，应用性能要求，对池的结构做调整。
>
> - 初始化个数
> - 最大个数
> - 最大空闲书
> - 最小空闲数等等。

![](https://gitee.com/sxhDrk/images/raw/master/imgs/一致性hash-虚拟节点.png)

> 当池中的空闲Jedis对象个数超过==最大空闲数==时，就会适当减少Jedis对象个数，但空闲的对象个数不会少于==最小空闲数==

### 使用

> 1. 创建一个连接池配置对象：`JedisPoolConfig`
>    - 配置最大连接数，【不配置会有默认值】
>    - 配置最大空闲数，【不配置会有默认值】
>    - 配置最小空闲数，【不配置会有默认值】
> 2. 创建一个连接池对象：`JedisPool`
>    - 传入连接池配置对象
>    - 传入Redis的`ip`和`port`
> 3. 使用：从池中获取一个Jedis对象：`getResource()`
>    - 使用完后，不能`close()`，通过`returnResource()`将对象返还到连接池中

```java
@Test
public void test04(){
    //创建连接池对象，先准备一个连接池属性配置对象
    JedisPoolConfig config=new JedisPoolConfig();
    //定义最大连接数 最小空闲，最大空闲，初始化等等
    config.setMaxTotal(200);//最大连接数，当最小空闲不满足时，创建新的连接，上限200
    config.setMaxIdle(8);//最大空闲数，当jedis空闲数量超过最大空闲，会关闭多与的jedis
    config.setMinIdle(3);//最小空闲数,不满足最小空闲说明忙，按照最大空闲的上限，补充jedis
    //使用config对象创建jedisPool
    JedisPool pool=new JedisPool(config,"192.168.16.84",6380);
    //连接池底层就是按照给定属性，创建一批连接6380的jedis
    Jedis resource = pool.getResource();//相当于从连接池拿到一个空闲的jedis对象
    resource.set("name","王老师");
    pool.returnResource(resource);
}
```



## 5.2、分片对象连接池

> 像JedisPool管理的是多个Jedis对象一样，ShardedJedis管理的是多个ShardedJedis连接对象，从中获取的分片对象，使用完毕可以还回资源

![](https://gitee.com/sxhDrk/images/raw/master/imgs/分片对象连接池.png)



### 使用

> 在业务层注入连接池对象，拿到资源，使用完毕后返回资源。整个框架系统只会创建一次连接池

1. 配置一个配置类，编写分片连接池配置

   > `@ConfigurationProperties`：配合配置类实现在配置类中的属性赋值的功能。==前提：属性必须有get/set方法==
   >
   > - `@ConfigurationProperties(prefix = "redis")`
   >   - properties文件中以`redis`开头的配置都会进入该配置类
   >   - `redis.maxTotal=200`----`Integer maxTotal=200`
   >   - Java中的集合元素，在properties文件中，值用`,`分开，springboot会自动以`,`分割存入集合中

   ```java
   @Configuration
   @ConfigurationProperties(prefix = "redis")//当前配置类主要配置redis
   public class ShardedJedisConfig {
       private Integer maxTotal;//properties文件中 redis.maxTotal
       private Integer maxIdle;//properties文件中 redis.maxIdle;
       private Integer minIdle;
       //ConfigurationProperties支持 属性值以，隔开  赋值list类型数据
       private List<String> nodes;//有多少个节点就以10.9.9.9:6379，10.9.9.9：6380,
       //在properties文件中，需要按照对应关系去指定配置key值
       //key=prefix.属性名称
       
       //---------此处编写属性的get/set方法--------
   
       @Bean
       public ShardedJedisPool initPool(){
           //收集节点信息
           List<JedisShardInfo> info=new ArrayList<>();
           //从nodes数据中解析出来每一个节点ip地址和端口号
           //nodes={"10.9.104.184:6379","10.9.104.184:6380","10.9.104.184:6381"}
           for (String node:nodes) {
               //对list的所有元素进行循环，每次循环拿出一个元素值，赋值给node
               //node循环第一次 node="10.9.104.184:6379"
               String host=node.split(":")[0];
               int port=Integer.parseInt(node.split(":")[1]);//"6379"-->6379
               info.add(new JedisShardInfo(host,port));
           }
           //配置属性config
           JedisPoolConfig config=new JedisPoolConfig();
           config.setMaxTotal(maxTotal);
           config.setMinIdle(minIdle);
           config.setMaxIdle(maxIdle);
           return new ShardedJedisPool(config,info);
       }
   }
   ```

2. properties文件

   ```properties
   #==============redis分片连接池配置=============
   redis.maxTotal=200
   redis.maxIdle=8
   redis.minIdle=3
   redis.nodes=192.168.16.84:6379,192.168.16.84:6380,192.168.16.84:6381
   ```

   

### 总结

1. 所有通过代码连接技术实现操作技术发送命令的过程，都需要在springboot中以配置类的形式存在，创建想要的各种对象（jedis 创建ShardedJedisPool）。然后在业务逻辑中注入使用这些对象。

2. 配置类基本结构

   - `@Configuration`标识一个自定义的配置类，在类中实现配置逻辑

   - `@Bean` 配置类中创建框架管理bean对象注解，但是bean对象很多在初始化时，都需要参数赋值。

   - `@ConfigurationProperties`：

     在配置类中的私有属性配合`getter&&setter`方法实现带有前缀匹配的赋值，例如properties中 `easymall.redis.nodes`的key值，在代码中可以定义前缀为`easymall.redis`，属性名称nodes


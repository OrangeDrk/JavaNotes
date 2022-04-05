[TOC]

> **Zookeeper服务端用的哪个版本，API最好也用哪个版本**

# pom.xml文件

> Java代码使用ZooKeeper，需要导入依赖

```xml
<dependencies>
    <dependency>
        <!-- 单元测试 -->
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.10</version>
    </dependency> 
    <dependency>
        <!-- 日志记录 -->
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-simple</artifactId>
        <version>1.6.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <!-- Zookeeper -->
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.7</version>
    </dependency>
</dependencies>
```





# 创建连接

## java连接

> ZooKeeper本身是一个非阻塞式连接，需要加锁（CountDownLatch），不然还没等连接上，线程就结束了
>
> `new ZooKeeper`的三个参数：
>
> - `connectString`：连接地址+端口号
> - `sessionTimeout`：会话超时时间，表示连接超时时间，单位默认ms
> - `watcher`：监控者，监控连接状态

```java
public void connect() throws Exception{

    CountDownLatch cdl = new CountDownLatch(1);
    zk = new ZooKeeper(
        "10.42.0.33:2181",
        5000,
        new Watcher() {
            @Override
            public void process(WatchedEvent event) {
                //判断链接是否建立
                //将监控的事件封装成WatchedEvent对象
                if(event.getState() == Event.KeeperState.SyncConnected)
                    System.out.println("连接成功");
                cdl.countDown();
            }
        });

    cdl.await();
    System.out.println("运行完成");
}
```

## SpringBoot连接

> 因为连接Zookeeper需要一段时间，需要上个锁，不然非阻塞，程序还没连上zk就向下执行了
>
> ==**创建链接时的Watch是监听Session的Watch，只要和Session有关的事件，都会被监听到**==

配置文件

> 因为每个项目可能使用同一个zookeeper，所以==**connectingString可以指定一个path，指定项目的工作目录**==

```yaml
# zookeeper配置
zk:
  connectString: 182.92.95.61:2181  #182.92.95.61:2181/sxhTest
  sessionTimeout: 3000
```



```java
/**
 * @Author shixiaohao
 * @Date 2021/3/29 14:45
 * @Description: ZookeeperConfig
 */
@Configuration
@ConfigurationProperties(prefix = "zk")
public class ZookeeperConfig {
    private String connectString;
    private Integer sessionTimeout;

    private CountDownLatch countDownLatch = new CountDownLatch(1);

    @Bean
    public ZooKeeper zookeeper() throws Exception{
        // 这里的Watch可以创建一个类实现该方法，传入defalutWatch，一般这个watch很少用
        ZooKeeper zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                Event.KeeperState state = watchedEvent.getState();
                Event.EventType type = watchedEvent.getType();
                String path = watchedEvent.getPath();
                switch (state) {
                    case SyncConnected:
                        System.out.println("SyncConnected");
                        countDownLatch.countDown(); // 减少count
                        break;
                    .....
                }

                switch (type) {
                    .....
                }
            }
        });
        ZooKeeper.States state = zk.getState();
        switch (state) {
            case CONNECTING:
                System.out.println("connecting");
                break;
            case CONNECTED:
                System.out.println("connected");
                break;
            .....
        }

        countDownLatch.await(); //等待count变为0
        return zk;
    }
    
     public String getConnectString() {
        return connectString;
    }

    public void setConnectString(String connectString) {
        this.connectString = connectString;
    }

    public Integer getSessionTimeout() {
        return sessionTimeout;
    }

    public void setSessionTimeout(Integer sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

}

```



> 测试

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class ZookeeperTest {

    @Autowired
    ZooKeeper zooKeeper;

    @Test
    public void create() throws KeeperException, InterruptedException {
        String pathName = zooKeeper.create("/sxh/log", 
                                           "sxhTest".getBytes(),
                                           ZooDefs.Ids.OPEN_ACL_UNSAFE,
                                           CreateMode.PERSISTENT);
        System.out.println(pathName);
    }
}
```



# 节点操作



## 创建节点

> `creat(path,data,acl,createMode)`：
>
> - path：节点路径
> - data：节点数据
> - acl：acl策略
> - createMode：节点类型

```java
//创建节点
@Test
public void createNode() throws KeeperException, InterruptedException {
    //返回值表示节点的实际路径
    String path = zk.create(
        "/log", 
        "log servers".getBytes(),
        ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT_SEQUENTIAL);

    System.out.println(path);
}
```



## 删除节点

> `delete(path,version)`：
>
> - path：节点路径
> - version：数据版本（==-1表示不检查版本==）
>
> 再删除节点的时候，会比较指定的数据版本和节点的实际版本是否一致，如果一致则执行删除操作，如果不一致则放弃该操作

```java
//删除节点
@Test
public void delete() throws KeeperException, InterruptedException {
    zk.delete("/log0000000005",0);//检查版本
    zk.delete("/log0000000006",-1);//不检查版本
}
```



## 更新节点数据

> `setData(path,data,version)`：
>
> - path：节点路径
> - data：更新的数据
> - version：数据版本（-1表示不检查版本）

```java
//修改数据
@Test
public void changeData() throws KeeperException, InterruptedException {
    //返回节点信息
    Stat s = zk.setData("/log", "logs".getBytes(), -1);
    System.out.println(s);
}
```



## 获取节点数据

> `getData(path,watcher,stat)`：
>
> - path：节点路径
> - watcher：监控器
> - stat：节点对象
>
> 传入一个节点对象，会把该节点信息封装到对象中
>
> `getData()`方法返回一个`byte[]`，存的内容为：节点内容

```java
//获取节点数据
@Test
public void getData() throws KeeperException, InterruptedException {
    //需要节点信息，第三个参数传递一个stat对象
    Stat s = new Stat();//存储的是节点各种状态信息
    byte[] data = zk.getData("/log", null, s);//返回的节点存储的【内容】
    System.out.println(s);
    System.out.println(new String(data));

    //如果不需要节点信息，第三个参数可以写null
    // byte[] data1 = zk.getData("/log", null, null);
    // System.out.println(new String(data1));

}
```



## 判断节点是否存在

> `exists(path,watcher)`：
>
> - path：节点路径
> - watcher：监控器

```java
//判断节点是否存在
@Test
public void exists() throws KeeperException, InterruptedException {
    //返回值表示节点信息，
    Stat tmp = zk.exists("/tmp", null);
    System.out.println("节点存在？：" + tmp != null);
}
```



## 获取子节点

> `getChildren(path,watcher)`：
>
> - path：节点路径
> - watcher：监控器

```java
//获取一个节点的子节点
@Test
public void getChildren() throws KeeperException, InterruptedException {
    List<String> children = zk.getChildren("/", null);
    for (String child : children) {
        System.out.println(child);
    }
}
```



# Watcher的使用



## 监听节点数据变化

> `Event.EventType.NodeDataChanged`：监听节点数据是否变化

```java
//监控ZooKeeper中/log节点的数据是否发生变化
@Test
public void dataChanged() throws KeeperException, InterruptedException {

    CountDownLatch cdl = new CountDownLatch(1);
    //get是非阻塞的
    zk.getData("/log", new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            if (event.getType()== Event.EventType.NodeDataChanged){
                System.out.println("节点数据被修改");
                System.out.println("修改时间"+new Date(System.currentTimeMillis()));
                cdl.countDown();
            }
        }
    },null);
    cdl.await();
}
```



## 监听子节点个数的变化

> `Event.EventType.NodeChildrenChanged`:监听子节点变化

```java
//监控子节点个数变化
@Test
public void childrenChanged() throws KeeperException, InterruptedException {

    CountDownLatch cdl = new CountDownLatch(1);
    zk.getChildren("/log", new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            if (event.getType() == Event.EventType.NodeChildrenChanged) {
                System.out.println("子节点个数发生变化");
                System.out.println("变化时间：" + new Date(System.currentTimeMillis()));
                cdl.countDown();
            }
        }
    });
    cdl.await();
}
```



## 监听节点的创建/删除

> `Event.EventType.NodeCreated`：监听节点的创建
>
> `Event.EventType.NodeDeleted`：监听节点的删除

```java
//监听节点的创建，删除
@Test
public void nodeChanged() throws KeeperException, InterruptedException {

    CountDownLatch cdl = new CountDownLatch(1);
    zk.exists("/tmp", new Watcher() {
        @Override
        public void process(WatchedEvent event) {
            if (event.getType() == Event.EventType.NodeCreated) {
                System.out.println("tmp节点被创建");
            }else if (event.getType() == Event.EventType.NodeDeleted){
                System.out.println("tmp节点被删除");
            }
            cdl.countDown();
        }
    });
    cdl.await();
}
```



# Watch的使用范围

> 每次watch回调只会生效一次，如果想重复生效，需要重复注册Watch

```java
@Test
    public void getData() throws KeeperException, InterruptedException {
        byte[] data = zooKeeper.getData("/sxh/log", new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                System.out.println("get Data:" + watchedEvent.toString());
                try {
                    zooKeeper.getData("/sxh/log", this, new Stat()); // this : 将该watch重新注册
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, new Stat());
        System.out.println(new String(data));

        Stat stat = zooKeeper.setData("/sxh/log", "abcTest".getBytes(), 1);

        Stat stat1 = zooKeeper.setData("/sxh/log", "abcTest11".getBytes(), stat.getVersion());


        Thread.sleep(2222222); // 临时为了不让程序停止而设置的

    }
```

![image-20210329164705494](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210329170051243.png)



> 也可以重新注册默认的Watch，也就是创建Zookeeper时创建的Watch

```java
@Test
    public void getData() throws KeeperException, InterruptedException {
        byte[] data = zooKeeper.getData("/sxh/log", new Watcher() {
            @Override
            public void process(WatchedEvent watchedEvent) {
                System.out.println("get Data:" + watchedEvent.toString());

                try {
                    zooKeeper.getData("/sxh/log", true,new Stat()); //true : 表示注册默认watch监听
                } catch (KeeperException e) {
                    e.printStackTrace();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }, new Stat());
        System.out.println(new String(data));

        Stat stat = zooKeeper.setData("/sxh/log", "abcTest".getBytes(),1);

        Stat stat1 = zooKeeper.setData("/sxh/log", "abcTest11".getBytes(), stat.getVersion());


        Thread.sleep(2222222);

    }
```

![image-20210329170051243](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210329164705494.png)

> 默认的Watch

![image-20210329170121605](https://gitee.com/sxhDrk/images/raw/master/imgs/image-20210329170121605.png)



# 异步式回调

> 线程不用阻塞的回调。当有事件发生时，回头调用回调方法

```java
@Test
    public void getData() throws KeeperException, InterruptedException {
        String pathName = zooKeeper.create("/sxh/log",
                                           "sxhTest".getBytes(),
                                           ZooDefs.Ids.OPEN_ACL_UNSAFE,
                                           CreateMode.EPHEMERAL);
        System.out.println(pathName);
        
        Stat stat = zooKeeper.setData("/sxh/log", "abcTest".getBytes(),0);

        // 异步式回调
        System.out.println("-----------async start----------");
        zooKeeper.getData("/sxh/log", false, new AsyncCallback.DataCallback() {
            @Override
            public void processResult(int rc, String path, Object ctx, byte[] data, Stat stat) {
                System.out.println("------async call back-------");
                System.out.println(ctx.toString());
                System.out.println(new String(data));
            }
        },"123");


        Thread.sleep(2222222);

    }
```


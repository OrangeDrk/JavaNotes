[TOC]

# 1. 异步watch，callback应用 - 配置中心







# 2. 分布式锁

> 锁的争抢，只能有一个线程能获取到锁

逻辑处理完后，线程需要释放锁，该如何让其他线程知道锁已被释放？

1. 每个线程主动轮询查询是否还存在锁

   弊端：存在网络延迟，大量的线程访问，zk压力太大

2. 通过watch监听

   弊端：也会有大量的线程访问zk，zk压力过大

3. 通过临时顺序节点+watch

   ==每个节点只监听他前一个节点==，前一个节点消失，代表当前线程可以获取锁

> 下面是基于SpringBoot，模拟的zk分布式锁

## 2.1 配置文件

> zk的ip后加`/xxx`，表示接下来的`/`代表`/xxx`
>
> 也就是后面的操作空间都是`/xxx`下的

```yaml
# zookeeper配置
zk:
  connectString: 182.92.95.61:2181/testLock  # 指定工作空间：/testLock
  sessionTimeout: 10000
```



## 2.2 config文件

> 将Zookeeper交给Spring管理，后面可以通过`@Autowired`注入

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

    @Autowired
    DefaultWatch defaultWatch;

    @Bean
    public ZooKeeper zookeeper() throws Exception{
        ZooKeeper zk = new ZooKeeper(connectString, sessionTimeout,defaultWatch);
        defaultWatch.setCdl(countDownLatch);
        countDownLatch.await();
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





## 2.3 默认的zk连接watch

```java

/**
 * @Author shixiaohao
 * @Date 2021/3/29 17:45
 * @Description: DeafultWatch
 */
@Component
@Slf4j
public class DefaultWatch implements Watcher {

    private CountDownLatch cdl;

    @Override
    public void process(WatchedEvent event) {
        log.info("zookeeper default watch --> event:{}", event.toString());
        if (Event.KeeperState.SyncConnected == event.getState()) {
            cdl.countDown();
            log.info("zookeeper已建立链接 , state:{}", event.getState());
        }
    }

    public CountDownLatch getCdl() {
        return cdl;
    }

    public void setCdl(CountDownLatch cdl) {
        this.cdl = cdl;
    }
}

```



## 2.4 自定义分布式锁

```java

/**
 * @Author shixiaohao
 * @Date 2021/4/8 13:54
 * @Description: LockWatch
 */
@Slf4j
public class LockWatch implements Watcher,AsyncCallback.StringCallback, AsyncCallback.ChildrenCallback, AsyncCallback.StatCallback {

    private ZooKeeper zooKeeper;

    CountDownLatch cdl = new CountDownLatch(1);

    private String pathName;

    private String ThreadName;

    /**
     * 加锁
     */
    public void tryLock(){
        String data;
        try {
            byte[] byteData = zooKeeper.getData("/",false,new Stat());
            data = new String(byteData);
            if (!data.equals(ThreadName)) { // 可重入锁，如果/节点的数据正好是当前线程的名字，那么可以直接获得锁，执行业务逻辑
                zooKeeper.create("/lock", ThreadName.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL, this, "abc");
            }
            cdl.await();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    /**
     * 释放锁
     */
    public void unLock(){
        try {
            zooKeeper.delete(pathName, -1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }



    /**
     * 创建回调
     */
    @Override
    public void processResult(int rc, String path, Object ctx, String name) {
        // 创建成功
        if (name != null) {
            log.info(ThreadName + " create node:{}", name);
            pathName = name;
            zooKeeper.getChildren("/", false, this, "aaa");
        }

    }

    /**
     * 前面临时节点删除的监听回调
     * @param event
     */
    @Override
    public void process(WatchedEvent event) {
        if (event.getType().equals(Event.EventType.NodeDeleted)) {
            zooKeeper.getChildren("/", false, this, "aaa1");
        }
    }

    /**
     * 创建成功后，获取当前父节点的children回调
     */
    @Override
    public void processResult(int rc, String path, Object ctx, List<String> children) {
        Collections.sort(children);
        System.out.println(children);
        int i = children.indexOf(pathName.substring(1));
        // 判断是不是第一个
        if (i == 0) {
            // 是第一个
            try {
                // 将ThreadName写入到/下 --->  可重入锁
                zooKeeper.setData("/",ThreadName.getBytes(),-1);
                cdl.countDown();
            } catch (KeeperException e) {
                e.printStackTrace();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }else {
            // 不是第一个
            zooKeeper.exists("/" + children.get(i - 1), this, this, "bbb");
        }
    }

    /**
     * 当前面锁没有失效的回调
     */
    @Override
    public void processResult(int rc, String path, Object ctx, Stat stat) {
        // 暂时不做处理
    }

    public String getThreadName() {
        return ThreadName;
    }

    public void setThreadName(String threadName) {
        ThreadName = threadName;
    }

    public ZooKeeper getZooKeeper() {
        return zooKeeper;
    }

    public void setZooKeeper(ZooKeeper zooKeeper) {
        this.zooKeeper = zooKeeper;
    }
}

```



## 2.5 测试使用

```java
/**
 * @Author shixiaohao
 * @Date 2021/4/8 13:53
 * @Description: ZKLock
 */
@SpringBootTest
@RunWith(SpringRunner.class)
public class TestZKLock {

    @Autowired
    ZooKeeper zooKeeper;


    @Test
    public void test(){
        for (int i = 0; i < 10; i++) {
            LockWatch zkLock = new LockWatch();
            new Thread(()->{
                zkLock.setThreadName(Thread.currentThread().getName());
                zkLock.setZooKeeper(zooKeeper);
                // 加锁
                zkLock.tryLock();

                // 处理业务 --- 通过sleep模拟业务逻辑处理
                try {
                    System.out.println(Thread.currentThread().getName()+" working.....");
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                // 释放锁
                zkLock.unLock();
            }).start();
        }
        while (true) {

        }

    }
}

```



## 2.6 测试结果

```
2021-04-08 15:29:14.914  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-5 create node:/lock0000000140
2021-04-08 15:29:14.914  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-6 create node:/lock0000000141
2021-04-08 15:29:14.914  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-2 create node:/lock0000000142
2021-04-08 15:29:14.914  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-11 create node:/lock0000000143
2021-04-08 15:29:14.917  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-4 create node:/lock0000000144
2021-04-08 15:29:14.917  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-7 create node:/lock0000000145
2021-04-08 15:29:14.917  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-3 create node:/lock0000000146
2021-04-08 15:29:14.917  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-9 create node:/lock0000000147
2021-04-08 15:29:14.917  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-10 create node:/lock0000000148
2021-04-08 15:29:14.917  INFO 19840 --- [ain-EventThread] org.example.testzklock.watch.LockWatch   : Thread-8 create node:/lock0000000149
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
Thread-5 working.....
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000140, lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
[lock0000000141, lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
Thread-6 working.....
[lock0000000142, lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
Thread-2 working.....
[lock0000000143, lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
Thread-11 working.....
[lock0000000144, lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
Thread-4 working.....
[lock0000000145, lock0000000146, lock0000000147, lock0000000148, lock0000000149]
Thread-7 working.....
[lock0000000146, lock0000000147, lock0000000148, lock0000000149]
Thread-3 working.....
[lock0000000147, lock0000000148, lock0000000149]
Thread-9 working.....
[lock0000000148, lock0000000149]
Thread-10 working.....
[lock0000000149]
Thread-8 working.....

Process finished with exit code -1

```




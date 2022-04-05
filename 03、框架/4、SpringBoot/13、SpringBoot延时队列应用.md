# Demo

**DelayQueue简介**

在很多场景我们需要用到延时任务，比如给客户异步转账操作超时后发通知告知用户，还有客户下单后多长时间内没支付则取消订单等等，这些都可以使用延时任务来实现。

jdk中DelayQueue可以实现上述需求，顾名思义DelayQueue就是延时队列。

> DelayQueue提供了在指定时间才能获取队列元素的功能，队列头元素是最接近过期的元素。
>
> 没有过期元素的话，使用poll()方法会返回null值，超时判定是通过getDelay(TimeUnit.NANOSECONDS)方法的返回值小于等于0来判断。
>
> 延时队列不能存放空元素。
>
> 一般使用take()方法阻塞等待，有过期元素时继续。

**队列元素说明**

DelayQueue<E extends Delayed>的队列元素需要实现Delayed接口，该接口类定义如下:

```java
public interface Delayed extends Comparable<Delayed> {
 
    /**
     * Returns the remaining delay associated with this object, in the
     * given time unit.
     *
     * @param unit the time unit
     * @return the remaining delay; zero or negative values indicate
     * that the delay has already elapsed
     */
    long getDelay(TimeUnit unit);
}
```

所以DelayQueue的元素需要实现getDelay方法和Comparable接口的compareTo方法，getDelay方法来判定元素是否过期，compareTo方法来确定先后顺序。

**springboot中实例运用**

DelayTask就是队列中的元素

```java
import java.util.Date;
import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

public class DelayTask implements Delayed {
  final private TaskBase data;
  final private long expire;

  /**
     * 构造延时任务
     * @param data      业务数据
     * @param expire    任务延时时间（ms）
     */
  public DelayTask(TaskBase data, long expire) {
    super();
    this.data = data;
    this.expire = expire + System.currentTimeMillis();
  }

  public TaskBase getData() {
    return data;
  }

  public long getExpire() {
    return expire;
  }

  @Override
  public boolean equals(Object obj) {
    if (obj instanceof DelayTask) {
      return this.data.getIdentifier().equals(((DelayTask)obj).getData().getIdentifier());
    }
    return false;
  }

  @Override
  public String toString() {
    return "{" + "data:" + data.toString() + "," + "expire:" + new Date(expire) + "}";
  }

  @Override
  public long getDelay(TimeUnit unit) {
    return unit.convert(this.expire - System.currentTimeMillis(), unit);
  }

  @Override
  public int compareTo(Delayed o) {
    long delta = getDelay(TimeUnit.NANOSECONDS) - o.getDelay(TimeUnit.NANOSECONDS);
    return (int) delta;
  }
}
```

TaskBase类是用户自定义的业务数据基类，其中有一个identifier字段来标识任务的ID，方便进行索引

```java
import com.alibaba.fastjson.JSON;
 
public class TaskBase {
    private String identifier;
 
    public TaskBase(String identifier) {
        this.identifier = identifier;
    }
 
    public String getIdentifier() {
        return identifier;
    }
 
    public void setIdentifier(String identifier) {
        this.identifier = identifier;
    }
 
    @Override
    public String toString() {
        return JSON.toJSONString(this);
    }
}
```



定义一个延时任务管理类DelayQueueManager，通过@Component注解加入到spring中管理，在需要使用的地方通过@Autowire注入

```java
import com.alibaba.fastjson.JSON;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;
 
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.concurrent.DelayQueue;
import java.util.concurrent.Executors;
 
@Component
public class DelayQueueManager implements CommandLineRunner {
    private final Logger logger = LoggerFactory.getLogger(DelayQueueManager.class);
    private DelayQueue<DelayTask> delayQueue = new DelayQueue<>();
     
    /**
     * 加入到延时队列中
     * @param task
     */
    public void put(DelayTask task) {
        logger.info("加入延时任务：{}", task);
        delayQueue.put(task);
    }
 
    /**
     * 取消延时任务
     * @param task
     * @return
     */
    public boolean remove(DelayTask task) {
        logger.info("取消延时任务：{}", task);
        return delayQueue.remove(task);
    }
 
    /**
     * 取消延时任务
     * @param taskid
     * @return
     */
    public boolean remove(String taskid) {
        return remove(new DelayTask(new TaskBase(taskid), 0));
    }
 
    @Override
    public void run(String... args) throws Exception {
        logger.info("初始化延时队列");
        Executors.newSingleThreadExecutor().execute(new Thread(this::excuteThread));
    }
 
    /**
     * 延时任务执行线程
     */
    private void excuteThread() {
        while (true) {
            try {
                DelayTask task = delayQueue.take();
                processTask(task);
            } catch (InterruptedException e) {
                break;
            }
        }
    }
 
    /**
     * 内部执行延时任务
     * @param task
     */
    private void processTask(DelayTask task) {
        logger.info("执行延时任务：{}", task);
        //根据task中的data自定义数据来处理相关逻辑，例 if (task.getData() instanceof XXX) {}
    }
}
```

DelayQueueManager实现了CommandLineRunner接口，在springboot启动完成后就会自动调用run方法。

 

原文链接：https://www.cnblogs.com/ieinstein/p/12028459.html
[TOC]

# 1. ExecutorService - 执行器服务

> executor	[ɪɡˈzekjətə(r)]	-	执行者

## 1.1 本质

ExecutorService本质上是一个线程池

## 1.2 意义

减少服务器端线程的创建和销毁，做到线程的复用



## 1.3 组成

> 线程池刚创建好的时候，里面是空的，没有任何线程



### 1.3.1 核心线程

每过来一个请求，线程池就会生成一个线程（==核心线程 - core thread==）去处理这个请求。

> 核心线程的数量在创建线程池的时候需要给定

核心线程在用完之后不会销毁，而是等待下一个请求

在核心线程的数量达到指定数量之前，无论是否有空闲的核心线程，每次过来的请求都会在线程池中生成核心线程去处理，直到核心线程的数量达到了指定的数量



### 1.3.2 工作队列

核心线程全部被占用，后来的请求会被放入==工作队列（work queue）==中暂时存储。

> 工作队列本质上是一个阻塞式队列



### 1.3.3 临时线程

如果工作队列被全部占用，后来请求会交给==临时线程（temporary thread）==处理

> 临时线程的数量需要在创建线程池的时候指定

临时线程在用完之后，会**存活一段指定的时间**，如果超过指定时间没有接到新的任务就会被销毁

> 即使临时线程有空闲，也不会处理工作队列中的任务，工作队列中的任务一定是交给核心线程处理，考虑核心线程的复用



### 1.3.4 拒绝执行处理器

临时线程被全部占用，后来的请求会被交给==拒绝执行处理器（RejectedExecutionHandler）==来进行拒绝处理



## 1.4 整体图解

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111226.png)



## 1.5 线程池的使用

> 创建线程池参数：
>
> - `corePoolSize`：核心线程数量
> - `maximumPoolSize`：最大线程数量=核心线程数量+临时线程数量
> - `keepAliveTime`：临时线程存活时间
> -  `unit`：存活时间单位
> - `workQueue`：工作队列
> - `handler`：拒绝执行处理器

```java
public class ExecutorServiceDemo {

    @Test
    public void test() {

        // 创建线程池
        // corePoolSize：核心线程数量
        // maximumPoolSize：最大线程数量，实际上是核心线程+临时线程
        // keepAliveTime：临时线程存活时间
        // unit：时间单位
        // workQueue：工作队列
        // handler：拒绝执行助手，该参数可以定义也可以不定义
        ExecutorService es = new ThreadPoolExecutor(5, 10, 5000, TimeUnit.MILLISECONDS,
                                                    new ArrayBlockingQueue<Runnable>(5), new RejectedExecutionHandler() {
                                            });
        // 执行提交的任务
        // 该方法只能执行Runnable线程
        es.execute(new EsDemo());

        // 关闭线程池
        es.shutdown();
    }

}

class EsDemo implements Runnable {
    @Override
    public void run() {
        System.out.println("hello~~~");
    }
}

```

## 1.6 线程池的关闭

### 1.6.1 方法说明

#### shutdown()

> 停止接收新任务，原来的任务继续执行
>
> 英文原意：关闭，倒闭;停工。 这里的意思是 **关闭线程池**。**与使用数据库连接池一样，每次使用完毕后，都要关闭线程池。**

1. 停止接收新的submit的任务；
2. 已经提交的任务（包括正在跑的和队列中等待的）,会继续执行完成；
3. 等到第2步完成后，才真正停止；

#### shutdownNow()

> 停止接收新任务，原来的任务停止执行

1. 跟 shutdown() 一样，先停止接收新submit的任务；

2. 忽略队列里等待的任务；

3. 尝试将正在执行的任务interrupt中断；

4. 返回未执行的任务列表；

   **说明**：它试图终止线程的方法是通过调用 Thread.interrupt() 方法来实现的，这种方法的作用有限，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt() 方法是无法中断当前的线程的。所以，shutdownNow() 并不代表线程池就一定立即就能退出，它也可能必须要等待所有正在执行的任务都执行完成了才能退出。但是大多数时候是能立即退出的。

#### awaitTermination(long timeOut, TimeUnit unit)

> 当前线程阻塞
>
> timeout 和 TimeUnit 两个参数，用于设定超时的时间及单位

当前线程阻塞，直到：

- 等所有已提交的任务（包括正在跑的和队列中等待的）执行完；
- 或者 等超时时间到了（timeout 和 TimeUnit设定的时间）；
- 或者 线程被中断，抛出InterruptedException

然后会监测 ExecutorService 是否已经关闭，返回true（shutdown请求后所有任务执行完毕）或false（已超时）

### 1.6.2 区别

#### shutdown() 和 shutdownNow() 的区别

​	`shutdown()` 只是关闭了提交通道，用submit()是无效的；而内部该怎么跑还是怎么跑，跑完再停。
​	`shutdownNow()` 能立即停止线程池，正在跑的和正在等待的任务都停下了。

#### shutdown() 和 awaitTermination() 的区别

​	`shutdown()` 后，不能再提交新的任务进去；但是 `awaitTermination()` 后，可以继续提交。

​	`awaitTermination()`是阻塞的，返回结果是线程池是否已停止（true/false）；`shutdown()` 不阻塞。

### 1.6.3 总结

1. 优雅的关闭，用 shutdown()
2. 想立马关闭，并得到未执行任务列表，用shutdownNow()
3. 优雅的关闭，并允许关闭声明后新任务能提交，用 awaitTermination()
4. 关闭功能 【从强到弱】 依次是：shuntdownNow() > shutdown() > awaitTermination()

## 1.7 Callable和Runnable

- 实现Callable接口的线程，执行完后有返回值
- 实现Runnable接口的线程，执行完后没有返回值

> Callable和Runnable的区别

|          | Runnable                                                     | Callable                                     |
| -------- | ------------------------------------------------------------ | -------------------------------------------- |
| 返回值   | 重写run方法，没有返回值                                      | 重写call，通过泛型定义返回值的类型           |
| 启动方式 | 1、可以传递到==Thread中通过start启动==  <br />2、可以通过==线程池==的==execute==或者==submit==启动 | 只能通过==线程池的submit==方法启动           |
| 容错机制 | 不允许抛出异常，所以无法利用全局方式(例如Spring中的异常通知)处理 | 允许抛出异常，所以可以利用全局方式来处理异常 |

> Callable案例

```java
//创建一个线程池    
ExecutorService pool = Executors.newFixedThreadPool(taskSize);    
// 创建多个有返回值的任务    
List<Future> list = new ArrayList<Future>();    
for (int i = 0; i < taskSize; i++) {    
    Callable c = new MyCallable(i + " ");    
    // 执行任务并获取Future对象    
    Future f = pool.submit(c);     
    list.add(f);    
}    
// 关闭线程池   
pool.shutdown();     
// 获取所有并发任务的运行结果    
for (Future f : list) {    
    // 从Future对象上获取任务的返回值，并输出到控制台    
    System.out.println("res：" + f.get().toString());    
}
```



# 2. 内置线程池

> 共四种：
>
> - newCachedThreadPool
> - newFixedThreadPool
> - newScheduledThreadPool
> - newSingleThreadExecutor



## 2.1 newCachedThreadPool

> `ExecutorService es = Executors.newCachedThreadPol()`

- 只有临时线程没有核心线程
- 临时线程的数量是：Integer.MAX_VALUE，所以可以认为临时线程是无限多的
- 每一个临时线程在用完后能存活1分钟
- 工作队列是一个同步队列（`SynchronousQueue`）
- ==是一个大池子小队列的线程池==
- 适用于大量的短任务高并发的场景，不适用于长任务场景



> 初始方法：

```java
//newCachedThreadPool构造方法
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```



## 2.2 newFixedThreadPool

> `ExecutorService es = Executors.newFixedThreadPool(int nThreads);`

- 只有核心线程没有临时线程
- 工作队列是一个阻塞式链式队列，且该队列没有指定容量，所以容量是：Integer.MAX_VALUE，认为可以存储无限多个请求
- ==是一个大队列小池子线程池==
- 适用于长任务场景，不适用于短任务场景



> 初始方法：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```



## 2.3 newScheduledThreadPool

scheduled	['ʃedjuːld]	-	预定的

> `ScheduledExecutorService ses = Executors.newScheduledThreadPool(5);`

定义调度执行器。能够起到定时执行的效果，很多定时机制的底层都是利用这个线程池来实现的 



> 初始方法

```java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}
```



> 使用案例
>
> `scheduleAtFixedRate`：从上次线程启动开始，计算下一次线程启动的时间
>
> - 如果线程的执行时间超过时间间隔，则以线程执行时间为准
>
> `scheduleWithFixedDelay`：从上次线程结束开始，计算下一次线程启动的时间

```java
public class ScheduleExecutorServiceDemo {
    public static void main(String[] args) {
        ScheduledExecutorService ses = Executors.newScheduledThreadPool(5);
        
        //正常延迟5s后执行，只执行一次
        ses.schedule(new ScheduleThread(), 5, TimeUnit.SECONDS);

        // 每隔5s钟执行一次
        // initialDelay->延迟几秒启动
        // period->线程延迟时间
        // 👉👉从上一次启动开始，来计算下一次的启动时间
        // 👉👉如果线程的执行时间超过时间间隔，则以线程执行时间为准
        ses.scheduleAtFixedRate(new ScheduleThread(), 0, 5, TimeUnit.SECONDS);

        // 每隔5s执行一次
        // 从上一次线程结束开始，来计算下一次启动的时间
        ses.scheduleWithFixedDelay(new ScheduleThread(), 0, 5, TimeUnit.SECONDS);
    }
}

class ScheduleThread implements Runnable{

    @Override
    public void run() {
        try {
            System.out.println("hello");
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



## 2.4 newSingleThreadExecutor

> `ExecutorService es = Executors.newSingleThreadExecutor();`

这个线程池只有一个线程

这个线程池可以在线程死后（或发生异常时）重新启动一个线程来替代原来的线程继续执行下去！  



# 3. ForkJoinPool

> 分叉合并池

**分叉**：将一个大的任务拆分成了多个小的任务交给不同的线程来执行

**合并**：将拆分出去的小任务计算结果进行汇总整合

**特点**：分叉合并池能够有效的提高CPU的利用率，甚至能到到CPU利用率的100%，所以实际开发中，分叉合并一般是放在凌晨执行

**本质**：分叉合并本质上就是将大任务拆分成小任务，分配给多个线程，让多个线程落在不同的CPU核上来，提高CPU的利用率

**效率**：

- 如果数据量偏大，分叉合并的效率要远远高于循环
- 如果数据量偏小，循环的效率反而高于分叉合并



**（work-stealing）工作窃取策略**：

分叉合并在向每个核上分配任务的时候，考虑任务的均衡问题：核上任务多的少分，任务少的多分。由于CPU核在执行任务的时候，有快有慢(有的任务简单执行的就快，有的任务复杂执行的就慢)，所以先执行完所有任务的CPU核并不会闲下来，而是会随机的扫描一个其他的核，然后从被扫描的核的任务队列尾端"偷取"一个任务回来执行，这种策略称之为work-stealing(工作窃取)策略 



## 使用

- 功能类实现`RecursiveTask`，重写`compute()`方法
- 创建分叉合并池
- `submit()`提交任务，并接收返回值对象

```java
public class ForkJoinPoolDemo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        long begin = System.currentTimeMillis();
        // 求1-100000000000L
        
        // long sum = 0;
        // for (long i = 1; i <= 100000000000L; i++) {
        //     sum += i;
        // }
        // System.out.println(sum);

        ForkJoinPool pool = new ForkJoinPool();
        Future<Long> f = pool.submit(new Sum(1, 100000000000L));
        pool.shutdown();
        System.out.println(f.get());

        long end = System.currentTimeMillis();
        System.out.println(end - begin);

    }

}

class Sum extends RecursiveTask<Long> {

    private long start;
    private long end;

    public Sum(long start, long end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        // 如果start-end范围内数字比较少，那么可以将这个范围内的数字求和
        if (end - start <= 10000) {
            long sum = 0;
            for (long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } else {
            // 如果start-end数字依然比较多，那么就继续拆分
            long mid = (start + end) / 2;
            Sum left = new Sum(start, mid);
            Sum right = new Sum(mid + 1, end);
            // 拆分成了2个子任务
            left.fork();
            right.fork();
            // 需要将2个子任务的结果来进行汇总才是这个范围内的最终结果
            return left.join() + right.join();
        }

    }
}

```





# 4. 使用线程池过程的一些问题

## 4.1 父子线程之间数据的共享问题

### 4.1.1 每次都new Thread去执行线程

父子线程之间的数据共享通过InheritableThreadLocal类实现，在子线程初始化（new）的时候调用init方法设置的，在子线程中调用InheritableThreadLocal即可和父线程共享数据

```java
/**
 * 用于父子线程间共享 LoginUserOrg 对象
 *
 * @date 2021/07/01
 */
public class LoginUserOrgHolder {


    private LoginUserOrgHolder() {
    }

    private static final ThreadLocal<LoginUserOrg> THREAD_LOCAL = new InheritableThreadLocal<>();

    public static LoginUserOrg get() {
        return THREAD_LOCAL.get();
    }

    public static void set(LoginUserOrg obj) {
        THREAD_LOCAL.set(obj);
    }

    public static void remove() {
        THREAD_LOCAL.remove();
    }

}

```

> 在子线程中使用 `LoginUserOrgHolder` 类获取共享线程数据



### 4.1.2 使用线程池

使用线程池时，会提前在线程池中初始化一些线程，就没办法使用InheritableThreadLocal 类，在子线程init时设置了

所以使用alibaba提供的一个jar包，来解决使用线程池时，数据共享的问题

> 关键类：TransmittableThreadLocal 

```java
import com.alibaba.ttl.TransmittableThreadLocal;

/**
 * 用于父子线程间共享 LoginUserOrg 对象
 *
 * @date 2021/07/01
 */
public final class LoginUserOrgHolder {


    private LoginUserOrgHolder() {
    }

    private static final ThreadLocal<LoginUserOrg> THREAD_LOCAL = new TransmittableThreadLocal<>();

    public static LoginUserOrg get() {
        return THREAD_LOCAL.get();
    }

    public static void set(LoginUserOrg obj) {
        THREAD_LOCAL.set(obj);
    }

    public static void remove() {
        THREAD_LOCAL.remove();
    }

}

```



# 5. 线程池线程数量如何判定

- 业务复杂和简单；比方说业务比较复杂，处理时间比较长的
- 看是CPU密集型还是IO密集型
  - CPU密集型：线程数量最好和CPU核心数一致，避免CPU上下文切换带来的新能损耗
  - IO密集型：线程数量一般是CPU核心数量的2倍



> 线程是否越多越好？ 分析如下：

**对于cpu密集型**

- ​		一个计算为主的程序（专业一点称为CPU密集型程序）。多线程跑的时候，可以充分利用起所有的cpu核心，比如说4个核心的cpu,开4个线程的时候，可以同时跑4个线程的运算任务，此时是最大效率。
- ​		但是如果线程远远超出cpu核心数量 反而会使得任务效率下降，因为频繁的切换线程也是要消耗时间的。
- ​		因此对于cpu密集型的任务来说，线程数等于cpu数是最好的了。

**对于IO密集型**

- ​		如果是一个磁盘或网络为主的程序（IO密集型）。一个线程处在IO等待的时候，另一个线程还可以在CPU里面跑，有时候CPU闲着没事干，所有的线程都在等着IO，这时候他们就是同时的了，而单线程的话此时还是在一个一个等待的。我们都知道IO的速度比起CPU来是慢到令人发指的。所以开多线程，比方说多线程网络传输，多线程往不同的目录写文件，等等。
- ​		所以通常就需要开cpu核数的两倍的线程， 当线程进行I/O操作cpu空暇时启用其他线程继续使用cpu，提高cpu使用率

总结：

CPU 密集型，几核，就是几，可以保持CPu的效率最高！

IO 密集型 一般线程数需要设置2倍CPU数以上，

1：高并发、任务执行时间短的业务，线程池线程数可以设置为CPU核数+1，减少线程上下文的切换

2：并发不高、任务执行时间长的业务这就需要区分开看了：

a）假如是业务时间长集中在IO操作上，也就是IO密集型的任务，因为IO操作并不占用CPU，所以不要让所有的CPU闲下来，可以适当加大线程池中的线程数目，让CPU处理更多的业务

b）假如是业务时间长集中在计算操作上，也就是计算密集型任务，这个就没办法了，和（1）一样吧，线程池中的线程数设置得少一些，减少线程上下文的切换

（其实从一二可以看出无论并发高不高，对于业务中是否是cpu密集还是I/O密集的判断都是需要的当前前提是你需要优化性能的前提下）

3：并发高、业务执行时间长，解决这种类型任务的关键不在于线程池而在于整体架构的设计，看看这些业务里面某些数据是否能做缓存是第一步，我们的项目使用的时redis作为缓存（这类非关系型数据库还是挺好的）。增加服务器是第二步（一般政府项目的首先，因为不用对项目技术做大改动，求一个稳，但前提是资金充足），至于线程池的设置，设置参考 2 。最后，业务执行时间长的问题，也可能需要分析一下，看看能不能使用中间件（任务时间过长的可以考虑拆分逻辑放入队列等操作）对任务进行拆分和解耦。

[TOC]

# ExecutorService - 执行器服务

> executor	[ɪɡˈzekjətə(r)]	-	执行者

## 本质

ExecutorService本质上是一个线程池

## 意义

减少服务器端线程的创建和销毁，做到线程的复用



## 组成

> 线程池刚创建好的时候，里面是空的，没有任何线程



### 核心线程

每过来一个请求，线程池就会生成一个线程（==核心线程 - core thread==）去处理这个请求。

> 核心线程的数量在创建线程池的时候需要给定

核心线程在用完之后不会销毁，而是等待下一个请求

在核心线程的数量达到指定数量之前，无论是否有空闲的核心线程，每次过来的请求都会在线程池中生成核心线程去处理，直到核心线程的数量达到了指定的数量



### 工作队列

核心线程全部被占用，后来的请求会被放入==工作队列（work queue）==中暂时存储。

> 工作队列本质上是一个阻塞式队列



### 临时线程

如果工作队列被全部占用，后来请求会交给==临时线程（temporary thread）==处理

> 临时线程的数量需要在创建线程池的时候指定

临时线程在用完之后，会**存活一段指定的时间**，如果超过指定时间没有接到新的任务就会被销毁

> 即使临时线程有空闲，也不会处理工作队列中的任务，工作队列中的任务一定是交给核心线程处理，考虑核心线程的复用



### 拒绝执行处理器

临时线程被全部占用，后来的请求会被交给==拒绝执行处理器（RejectedExecutionHandler）==来进行拒绝处理



## 整体图解

![](https://note.youdao.com/yws/api/personal/file/13584C4FFC554713B2C2BDB00EDD4C17?method=download&shareKey=bb4eabd0bae8efa6212b7bf00771f9f3)



## 线程池的使用

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



## Callable和Runnable

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



# 内置线程池

> 共四种：
>
> - newCachedThreadPool
> - newFixedThreadPool
> - newScheduledThreadPool
> - newSingleThreadExecutor



## newCachedThreadPool

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



## newFixedThreadPool

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



## newScheduledThreadPool

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



## newSingleThreadExecutor

> `ExecutorService es = Executors.newSingleThreadExecutor();`

这个线程池只有一个线程

这个线程池可以在线程死后（或发生异常时）重新启动一个线程来替代原来的线程继续执行下去！  



# ForkJoinPool

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


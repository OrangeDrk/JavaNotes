[TOC]

# 概述

- Lock是JDK 1.5 提供的一个表示锁的新接口
- 提供了用于加锁的方法：`lock()`，以及解锁的方法：`unlock()`
- 相对传统的synchronized而言，Lock的方式更加灵活和精细

> synchronized要么只能修饰代码块，要么只能限制方法，而且synchronized在修饰代码块的时候需要确定锁对象，如果锁对象过多可能会导致死锁问题



# ReentrantLock - 重入锁

JDK 1.5 提供了一套无锁机制：`java.util.concurrent.locks`。Lock本身是一个接口，所以使用的是它的实现类：`ReentrantLock - 重入锁`

> 重入锁：这个线程用完后，允许这个线程或者其他线程重新抢占这个锁重新使用

```java
public class LockDemo {
    static int i = 0;

    public static void main(String[] args) throws InterruptedException {

        //ReetrantLock - 重入锁
        // Lock lock = new ReentrantLock();//默认非公平锁
        Lock lock = new ReentrantLock(true);//true - 公平锁
        new Thread(new Add(lock)).start();
        Thread.sleep(3000);
        System.out.println(i);
    }

}

class Add implements Runnable {

    private Lock lock;

    public Add(Lock lock) {
        this.lock = lock;
    }

    @Override
    public void run() {
        //加锁
        lock.lock();
        for (int i = 0; i < 1000; i++) {
            LockDemo.i++;
        }
        //释放锁
        lock.unlock();
    }
}

```

> ReentrantLock默认使用非公平锁，true参数表示使用公平锁

## ReadWriteLock - 读写锁

- 读锁：允许多个线程同时读，但是不允许写
- 写锁：只允许一个线程写，但是不允许读



## 公平锁和非公平锁

> 资源在被一个线程锁住期间，其他线程依然会试图抢占资源，只是其他线程抢不到而已

- 公平策略：公平策略中自带一个队列，线程并不是直接抢占资源，而是抢占入队顺序，在这种情况下，每一个线程执行次数基本一致
- 非公平策略：每一个线程都会试图抢占资源，虽然理论上来说每一个线程抢占能力是均等的，但是实际过程中每一个线程抢占的次数并不均等，甚至出现特别大的差别

> 非公平策略的效率是要高于公平策略的，公平策略中涉及到的线程调度会比非公平策略麻烦





# 其他锁

## CountDownLath - 线程递减锁/闭锁

> 会对线程来进行计数，在计数归零之前，会限制线程陷入阻塞；直到计数归零才会放开阻塞继续执行

对一组线程进行计数，当这组线程计数完成之后开启新的任务

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {

        //线程递减锁，线程数：4
        CountDownLatch cdl = new CountDownLatch(4);
        new Thread(new Teacher(cdl)).start();

        new Thread(new Student(cdl)).start();
        new Thread(new Student(cdl)).start();
        new Thread(new Student(cdl)).start();

        //在上面4个线程都执行完成之前，主线程应该等待（陷入阻塞）
        cdl.await();//在计数为0之前，主线程都会等待
        System.out.println("考试开始");
    }
}

class Teacher implements Runnable{
    private CountDownLatch cdl;

    public Teacher(CountDownLatch cdl) {
        this.cdl = cdl;
    }

    @Override
    public void run() {

        //模拟考官到厂时间不同
        try {
            Thread.sleep((long) (Math.random() * 10000));
            System.out.println("考官到达考场");
            cdl.countDown();//线程到达，计数-1
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Student implements Runnable{
    private  CountDownLatch cdl;

    public Student(CountDownLatch cdl) {
        this.cdl = cdl;
    }

    @Override
    public void run() {
        try {
            Thread.sleep((long) (Math.random()*10000));
            System.out.println("考生到达考场");
            cdl.countDown();//线程到达，计数-1
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
//👇👇👇运行结果👇👇👇
考生到达考场
考生到达考场
考官到达考场
考生到达考场
考试开始
```



## CyclicBarrier - 栅栏

> [ˈsaɪklɪk] [ˈbæriə(r)]
>
> 会对线程进行计数，在计数归零之前，会让线程陷入阻塞直到计数归零才会放开阻塞
>
> ==所有线程到达同一个点之后在分别继续往下执行==

```java
/**
 * 案例：跑步比赛
 * 运动员到了起跑线
 * 等待运动员到齐，发信号枪
 * 每个运动员跑出去
 */

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cb = new CyclicBarrier(6);

        new Thread(new Runner(cb),"1号").start();
        new Thread(new Runner(cb),"2号").start();
        new Thread(new Runner(cb),"3号").start();
        new Thread(new Runner(cb),"4号").start();
        new Thread(new Runner(cb),"5号").start();
        new Thread(new Runner(cb),"6号").start();
    }
}

class Runner implements Runnable {
    private CyclicBarrier cb;

    public Runner(CyclicBarrier cb) {
        this.cb = cb;
    }

    @Override
    public void run() {
        String name = Thread.currentThread().getName();
        try {
            //模拟运动员到起跑线的时间不同
            Thread.sleep((long) (Math.random()*10000));
            System.out.println(name+"运动员到达起跑线");
            cb.await();//到达之后，签到，同时计数-1，并等待计数为0
            //每个运动员到达之后因该等待，等所有运动员到齐后才能向下执行
            //运动员都到齐之后，才能跑
            System.out.println(name+"运动员跑了出去");
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
}
//👇👇👇运行结果👇👇👇
1号运动员到达起跑线
5号运动员到达起跑线
2号运动员到达起跑线
4号运动员到达起跑线
3号运动员到达起跑线
6号运动员到达起跑线
1号运动员跑了出去
5号运动员跑了出去
2号运动员跑了出去
3号运动员跑了出去
6号运动员跑了出去
4号运动员跑了出去
```



## Exchanger - 交换机

> 用于交换**两个线程**之间的信息 

```java
/**
 * 案例：
 * 生产 - 消费
 * 生产者将商品交换给消费者
 * 消费者向生产者付钱
 */
public class ExchangerDemo {
    public static void main(String[] args) {
        Exchanger<String> ex = new Exchanger<>();
        
        new Thread(new Producer(ex)).start();
        new Thread(new Consumer(ex)).start();
    }
}

class Producer implements Runnable{
    private Exchanger<String> ex;

    public Producer(Exchanger<String> ex) {
        this.ex = ex;
    }
    @Override
    public void run() {
        try {
            String info = "商品";
            //生产者将商品给消费者，同时应该收到消费者返回的金额
            String msg = ex.exchange(info);
            System.out.println("生产者收到消费者的信息："+msg);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Consumer implements Runnable{
    private Exchanger<String> ex;

    public Consumer(Exchanger<String> ex) {
        this.ex = ex;
    }
    @Override
    public void run() {
        try {
            String info = "财物";
            //消费者给生产者付钱，并收到生产者的商品
            String msg = ex.exchange(info);
            System.out.println("消费者收到生产者的信息："+msg);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
//👇👇👇运行结果👇👇👇
生产者收到消费者的信息：财物
消费者收到生产者的信息：商品
```



## Semaphore - 信号量

> [ˈseməfɔː(r)]
>
> 进行限流的。
>
> 每一个线程需要获取到信号之后才能执行逻辑
>
> - 当信号被全部占用之后，后来的线程就会被阻塞，
> - 直到有信号被释放，阻塞的线程才可以获取信号执行逻辑 

```java
/*
案例：饭馆吃饭
    饭馆的餐桌数量是有限的，每来一桌客人就会占用一张桌子
    当桌子全部被占用之后，后面来的客人就需要等待
    直到有客人离开，桌子被空闲出来，等待的客人才可以使用这张桌子
 */
public class SemaphoreDemo {
    public static void main(String[] args) {
        Semaphore s = new Semaphore(5);

        for (int i = 0; i < 8; i++) {
            new Thread(new Eater(s)).start();
        }
    }
}

class Eater implements Runnable {
    private Semaphore s;

    public Eater(Semaphore s) {
        this.s = s;
    }
    @Override
    public void run() {
        try {
            //每一个客人就会占用一张桌子
            s.acquire();//占取信号
            System.out.println("来了一桌客人，占用一张桌子");
            Thread.sleep((long) (Math.random()*10000));//模拟就餐时间

            System.out.println("客人用餐完毕，餐桌空闲");
            s.release();//释放信号
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
//👇👇👇运行结果👇👇👇
来了一桌客人，占用一张桌子
来了一桌客人，占用一张桌子
来了一桌客人，占用一张桌子
来了一桌客人，占用一张桌子
来了一桌客人，占用一张桌子
客人用餐完毕，餐桌空闲
来了一桌客人，占用一张桌子
客人用餐完毕，餐桌空闲
来了一桌客人，占用一张桌子
客人用餐完毕，餐桌空闲
来了一桌客人，占用一张桌子
客人用餐完毕，餐桌空闲
客人用餐完毕，餐桌空闲
客人用餐完毕，餐桌空闲
客人用餐完毕，餐桌空闲
客人用餐完毕，餐桌空闲
```


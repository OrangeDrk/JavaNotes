[TOC]

# 概述

1. 在操作属性的时候保证属性当原子性，即每次只允许一个线程对该属性的值进行更改
2. 常用当实现类有：
   - AtomicInteger
   - AtomicLong
   - AtomicBoolean
   - AtomicReference



# 代码

```java
public class AtomicIntegerDemo {
    // static int i = 0;
    static AtomicInteger ai = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {

        CountDownLatch cdl = new CountDownLatch(2);

        new Thread(new Add(cdl)).start();
        new Thread(new Add(cdl)).start();
        cdl.await();

        System.out.println(ai);

        // System.out.println(i);

    }
}

class Add implements Runnable{
    private CountDownLatch cdl;

    public Add(CountDownLatch cdl) {
        this.cdl = cdl;
    }

    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            AtomicIntegerDemo.ai.getAndIncrement();
            // AtomicIntegerDemo.i++;
        }
        cdl.countDown();
    }
}

```



# volatile

>  [ˈvɒlətaɪl]	-	易变的

- volatile是Java提供的一个线程之间当轻量级的同步机制

## 特点

1. 保证可见性：

   > 当一个线程改变数据之后，其他线程能立即感知到数据的变化

   ![](https://note.youdao.com/yws/api/personal/file/0277719737EB44739A79750774B462D9?method=download&shareKey=1ae03a3308077d1bc451efb889ccbeb2)

2. 不保证原子性

   > 当一个线程执行的时候，这个线程是不可分割或者不能打断的，保证线程操作的数据是完整的 - 不保证线程的安全

3. 禁止指令重排

   > 实际过程中，指令重排产生的概率不足万分之一 
   >
   > 指令重排就是实际执行过程和预期的执行过程产生了差异
   >
   > - 指令重排即使产生也不能违背数据的依赖性(如果使用这个数据，这个数据必须前面先出现)
   > - 在数据保证可见性的前提下，如果有多个线程指定同一段代码产生了指令重排，那么线程之间可能拿到的结果不一样(指令重排产生的二相性) 

   ![](https://note.youdao.com/yws/api/personal/file/4D6E4577F49241DAA4A97E39E6C7215E?method=download&shareKey=afbaa538201ae1892b78d53799f4d229)

   ![](https://note.youdao.com/yws/api/personal/file/145F62D2A31D42E9BB88B34847D8468A?method=download&shareKey=7092e08f5228bda98b6c8c26f2278a8b)
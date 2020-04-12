[TOC]

# BlockingQueue - 阻塞式队列

> BlockingQueue是阻塞式队列中的==顶级接口==

## 概述

1. 遵循队列FIFO的原则
2. 阻塞式队列往往是定长的，即==阻塞式队列的长度是固定的==
3. 满足阻塞的特点：
   - 如果队列已满，试图放入元素的线程就会被阻塞
   - 如果队列为空，试图获取元素的线程就会被阻塞
4. BlockingQueue中，不允许元素为null
5. ==适用于生产消费模型==



## 重要方法

|                  | 抛出异常 | 返回特殊值 | 阻塞   | 定时阻塞              |
| ---------------- | -------- | ---------- | ------ | --------------------- |
| 队列已满时，添加 | add(o)   | offer(o)   | put(o) | offer(o,  time, unit) |
| 队列为空时，获取 | remove() | poll()     | take() | poll(time,  unit)     |

> 队列已满时，调用add(o)会抛出异常：`java.lang.IllegalStateException: Queue full`
>
> 队列为空时，调用remove()会抛出异常：`java.util.NoSuchElementException`





## 添加方法详解

`offer(o,time,unit)`：在等待指定的延迟时间后还没有添加成功就返回false

```java
public class BlockingQueueDemo01 {
    public static void main(String[] args) throws InterruptedException {
        ArrayBlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

        queue.add("a");
        queue.add("a");
        queue.add("a");
        queue.add("a");
        queue.add("a");


        //定时阻塞
        boolean flag = queue.offer("e", 5, TimeUnit.SECONDS);
        System.out.println(flag);


        System.out.println(queue);
    }
}
//👇👇👇👇运行结果👇👇👇👇，经过5s后，返回false
false
[a, a, a, a, a]
```



## 获取方法详解

`remove()`：

- 队列为空，抛出异常
- 队列不为空，获取(删除)一个元素，并将该元素返回

`poll()`：

- 队列 为空，返回null
- 队列不为空，获取(删除)一个元素，并将该元素返回

`take()`：

- 队列为空，产生阻塞，会一直停在这个个=方法处
- 队列不为空，获取(删除)一个元素，并将该元素返回

`poll(o,time.unit)`：

- 队列为空，阻塞time时间后，返回false
- 队列不为空，获取(删除)一个元素，并将该元素返回

```java
public class BlockingQueueDemo02 {
    public static void main(String[] args) throws InterruptedException {
        LinkedBlockingQueue<String> queue = new LinkedBlockingQueue<>(5);
        queue.add("a");
        queue.add("b");
        queue.add("c");
        queue.add("d");

        //队列为空
        //抛出异常
        System.out.println(queue.remove());
        //删除成功返回true，失败返回false
        System.out.println(queue.remove("e"));

        //队列为空，返回null
        System.out.println(queue.poll());


        //产生阻塞
        System.out.println(queue.take());

        //定时阻塞
        System.out.println(queue.poll(5,TimeUnit.SECONDS));
    }
}
//👇👇👇👇运行结果👇👇👇👇
a
false
b
c
d
```





# 实现类

## ArrayBlockingQueue

> 阻塞式顺序队列

- 在使用的时候需要指定容量/界限
  - 容量在指定之后不能改动
- 底层基于数组来存储数据
- 遵循先进先出FIFO的原则



## LinkedBlockingQueue

> 阻塞式链式队列

- 在使用的时候可以指定容量，也可以不指定容量。
  - 不指定容量的情况下，默认容量为：Integer.MAX_VALUE（2^23^-1），此时认为容量是无限的
  - 如果指定容量，则容量指定之后不可改变
- 底层基于节点（链表）来存储数据



## PriorityBlockingQueue

> 阻塞式优先队列

- 在使用的时候可以指定容量，也可以不指定容量
  - 如果不指定容量，默认容量为11
  - 如果指定容量，则容量指定之后不可改变。指定容量最大不能超过Integer.MAX_VALUE-8
- 要求存储的元素对应的类必须实现Comparable接口，重写comparTo()方法来指定比较规则。在存储的时候会根据指定的比较规则对元素进行排序
- 在使用iterator迭代器遍历时不保证元素的排序顺序
- 底层基于数组实现存储数据

```java
public class PriorityBlockingQueueDemo {

    public static void main(String[] args) throws Exception {

        //如果想重新定义排序规则，又不能修改类源码，可以实现Comparator，重写compar()方法
        /* 
        Comparator<Student> c = new Comparator<Student>() {
            @Override
            public int compare(Student o1, Student o2) {
                return o2.getScore()-o1.getScore();
            }
        };

        PriorityBlockingQueue<Student> queue = new PriorityBlockingQueue<>(5,c);
        */


        // 创建队列
        PriorityBlockingQueue<Student> queue = new PriorityBlockingQueue<>();

        // 添加元素
        queue.put(new Student("Amy", 16));
        queue.put(new Student("Bob", 25));
        queue.put(new Student("Cathy", 20));
        queue.put(new Student("David", 13));

        // 遍历队列
        // 需要注意的是，如果想要拿到排序的结果，不能以迭代的方法获取
        for (int i = 0; i < 4; i++) {
            System.out.println(queue.take());
        }

    }

}

class Student implements Comparable<Student> {

    private String name;
    private int age;

    public Student(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Student [name=" + name + ", age=" + age + "]";
    }

    // 在这个方法中指定比较规则
    // 根据学生的年龄进行升序排序
    @Override
    public int compareTo(Student o) {
        return this.age - o.age;
    }

}
```



## SynchronousQueue

> 同步队列

- 使用时不需要指定容量，默认容量是1，并且只能是1
- 如果队列中已经有元素，试图向队列添加新元素的线程将会阻塞，直到另一个线程将该元素从队列中取走
- 如果队列为空，试图从队列中获取元素的线程将会阻塞，直到另一个线程向该队列添加了一个新的元素
- 一般会将同步队列称之为数据的汇合点



## BlockingDeque

> 阻塞式双向队列

- BlockingDeque继承了BlockingQueue
- 在使用的时候需要制定容量
- 该队列称之为双向队列，即队列的两端即可以添加元素，也可以获取元素
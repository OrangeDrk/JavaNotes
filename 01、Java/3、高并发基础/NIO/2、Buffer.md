[TOC]

# Buffer-缓冲区

## 介绍

1. 作用：对数据进行==存储==
2. 底层实现：底层基于数组来进行存储，针对基本类型来进行存储

## Buffer7种基本类型类

1. ByteBuffer
2. ShortBuffer
3. IntBuffer
4. LongBuffer
5. DoubleBuffer
6. FloatBuffer
7. CharBuffer



# ByteBuffer

## 基本使用

```java
public class ByteBufferDemo {
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(10);

        //获取缓冲区容量位
        System.out.println(buffer.capacity());
        System.out.println(buffer.position());

        //添加元素
        buffer.put("abc".getBytes());
        buffer.put("def".getBytes());

        //设置position位置
        buffer.position(0);

        //获取元素
        byte b = buffer.get();
        System.out.println(b);

       
        while (buffer.hasRemaining()) {
            byte b = buffer.get();
            System.out.println(b);
        }
    }
}
```



## 重要的四个属性

### capacity

> 容量位   [kəˈpæsəti]

用于标记当前缓冲区的容量，在缓冲区定义好之后就不可变

### limit

> 限制位

用于标记操作位所能达到的最大下标。

当操作位和限制位重合，就表示所有的位置已经操作完

==当缓冲区刚创建的时候，limit和capacity是重合的==

### position

> 操作位     [pəˈzɪʃn]

等价于数组中的下标。每次发生put或者get操作的时候就会==自动后移==。当缓冲区刚创建的时候，position指向第0位

### mark

> 标记位

mark的值默认为`-1`，表示不启用



## 重要属性位置

### 初始化时：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111302.png)

### 添加数据时：

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111303.png)

### 读取数据时：

> 读取数据时，需要将limit移动到position位置，再将position这是到初始的0位置
>
> 以上操作称为==翻转缓冲区==，方法为：`flip()`

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111304.png)



## 重要操作

### flip()：翻转缓冲区

```java
public final Buffer flip(){
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111305.png)

### clear()：清空缓冲区

> 就是将所有属性恢复到初始状态，就是清空缓冲区

```java
public final Buffer clear() {
    position = 0;
    limit = capacity;
    mark = -1;
    return this;
}
```



### reset()：重置缓冲区

> 在有标记位时，如果检查后来输入的数据有误，就可以重置缓冲区，在上一个mark标记处，重新输入数据

```java
public final Buffer reset() {
    int m = mark;
    if (m < 0)
        throw new InvalidMarkException();
    position = m;
    return this;
}
```

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111306.png)



### rewind()：重绕缓冲区

```java
public final Buffer rewind() {
    position = 0;
    mark = -1;
    return this;
}
```





# Buffer常用方法

| 方法                                    | 作用                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| allocate(int capacity)                  | 创建缓冲区的时候指定缓冲区容量的大小，实际上是指定缓冲区底层的字节数组的大小 |
| wrap(byte[] array)                      | 利用传入的字节数组来构建缓冲区                               |
| array()                                 | 获取缓冲区底层的字节数组                                     |
| get()                                   | 获取缓冲区中position位置上的字节                             |
| get(byte[]  dst)                        | 将缓冲区中的数据写到传入的字节数组中                         |
| get(int  index)                         | 获取指定下标上的字节                                         |
| put(byte b)                             | 向position位置上放入指定的字节                               |
| put(byte[]  src)                        | 向position位置上放入指定的字节数组                           |
| put(byte[] src, int offset, int length) | 向position位置上放入指定的字节数组的部分元素                 |
| put(ByteBuffer  src)                    | 将字节缓冲区放入                                             |
| put(int  index, byte b)                 | 向指定位置插入指定的字节                                     |
| capacity()                              | 获取容量位                                                   |
| clear()                                 | 清空缓冲区：position  = 0; limit = capacity; mark = -1;      |
| flip()                                  | 反转缓冲区：limit  = position; position = 0; mark = -1;      |
| hasRemaing()                            | 判断position和limit之间是否还有空余                          |
| limit()                                 | 获取限制位                                                   |
| limit(int  newLimit)                    | 设置限制位                                                   |
| mark()                                  | 设置标记位                                                   |
| position()                              | 获取操作位                                                   |
| position(int newPosition)               | 设置操作位                                                   |
| remaining()                             | 获取position和limit之间剩余的元素个数                        |
| reset()                                 | 重置缓冲区：position = mark                                  |
| rewind()                                | 重绕缓冲区：position = 0; mark = -1                          |
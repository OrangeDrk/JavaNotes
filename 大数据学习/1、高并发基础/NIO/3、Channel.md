[TOC]

# Channel - 通道

## 介绍

1. 作用：用于数据的==传输==
2. Channel==面向Buffer来进行传输==，即意味着数据需要先放到Buffer里面，然后再由Channel传输

## 分类

1. 文件：FileChannel
2. UDP：DatagrmChannel
3. TCP：SocketChannel，ServerSocketChannel



## 特点

1. 通道默认是阻塞的，可以手动设置为非阻塞
2. Channel可以实现双向传输：
   - 既可以作为输入流来读取数据
   - 也可以作为输出流来写入数据



# 代码

## 客户端

```java
public class Client {
    public static void main(String[] args) throws IOException {
        //开通通道
        SocketChannel sc = SocketChannel.open();

        //设置非阻塞
        sc.configureBlocking(false);

        //在非阻塞前提下，试图建立连接，无论能否连上，都不会在此等待，而是会继续向下执行
        //发起连接
        sc.connect(new InetSocketAddress("localhost", 8090));

        //判断是否建立连接
        while (!sc.isConnected()) {
            //底层自动计数，如果计数多次依然无法建立连接，说明连接存在问题，就会抛出异常
            sc.finishConnect();
        }

        //如果连接上，就写入数据
        //封装到ByteBuffer中
        ByteBuffer buffer = ByteBuffer.wrap("hello server".getBytes());
        sc.write(buffer);


        //关闭通道
        sc.close();
    }
}

```



## 服务器端

```java
public class Server {

    public static void main(String[] args) throws IOException {
        //开启服务端通道
        ServerSocketChannel ssc = ServerSocketChannel.open();
        //绑定端口
        ssc.bind(new InetSocketAddress(8090));

        //设置为非阻塞
        ssc.configureBlocking(false);

        //接收连接
        SocketChannel sc = ssc.accept();
        while (sc == null) {
            sc = ssc.accept();
        }

        //读数据
        ByteBuffer dst = ByteBuffer.allocate(1024);
        sc.read(dst);

        //解析数据
        System.out.println(new String(dst.array(), 0, dst.position()));

        sc.close();
    }
}
```


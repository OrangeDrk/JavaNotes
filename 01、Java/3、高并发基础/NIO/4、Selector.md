[TOC]

# Selector - 多路复用选择器

## 作用

针对非阻塞的通道进行==事件的选择==

![](https://gitee.com/sxhDrk/images/raw/master/imgs/20210427111313.png)

> 如果客户端连接服务端，作无效的死循环，是无法通过Selector选择器的，是无法连接到服务端的

Selector筛选客户端的连接

- 如果这个连接能够触发服务器端对应的事件，那么就允许产生连接
- 如果这个连接不能触发服务器端对应的事件，那么就会拦截这个连接



# 代码

## 服务器端

```java
public class server {
    public static void main(String[] args) throws IOException {
        //开启服务器通道
        ServerSocketChannel ssc = ServerSocketChannel.open();

        //绑定端口号
        ssc.bind(new InetSocketAddress(8070));

        //因为selector是针对非阻塞通道进行选择，要设置为非阻塞
        ssc.configureBlocking(false);

        //开启选择器
        Selector selec = Selector.open();

        //将服务器注册到选择器上
        ssc.register(selec, SelectionKey.OP_ACCEPT);

        //实际过程中，服务器一旦开启就不会关闭
        while (true) {//模拟服务器一直开启
            //选择出能触发服务器事件的连接
            selec.select();

            //无论筛选出多少个通道，最终只可能触发accept/read/write事件
            //确定这次选择之后的通道能触发的事件类型
            Set<SelectionKey> keys = selec.selectedKeys();

            //遍历set，根据触发的事件，进行不同的处理
            Iterator<SelectionKey> iterator = keys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                System.out.println(key);
                //可接受事件
                if (key.isAcceptable()) {
                    //从事件中将通道获取
                    ServerSocketChannel sscx = (ServerSocketChannel) key.channel();
                    SocketChannel sc = sscx.accept();
                    sc.configureBlocking(false);
                    //允许这个连接过来的通道，即允许读也允许写
                    sc.register(selec, SelectionKey.OP_WRITE+SelectionKey.OP_READ);

                }
                //可读事件
                if (key.isReadable()) {
                    //从事件上获取通道
                    SocketChannel sc = (SocketChannel) key.channel();
                    //准备缓冲区
                    ByteBuffer dst = ByteBuffer.allocate(1024);
                    //读取数据
                    sc.read(dst);
                    //解析数据
                    System.out.println(new String(dst.array(), 0, dst.position()));

                    //可读事件处理完毕，从通道上将可读事件移除
                    //key.interestOps() 获取原来所有的事件
                    sc.register(selec,key.interestOps()-SelectionKey.OP_READ);
                    // sc.register(selec,key.interestOps() ^ SelectionKey.OP_READ);
                }
                //可写事件
                if (key.isWritable()) {
                    // 从事件中获取通道
                    SocketChannel sc = (SocketChannel) key.channel();

                    //写数据
                    sc.write(ByteBuffer.wrap("hello client".getBytes()));

                    //可写事件处理完，将可写事件移除
                    sc.register(selec, key.interestOps() - SelectionKey.OP_WRITE);

                }
                //将通道移除
                iterator.remove();
            }
        }
    }
}

```



## 客户端

```java
public class client {

    public static void main(String[] args) throws IOException {
        //开启客户端通道
        SocketChannel sc = SocketChannel.open();
        //设置阻塞---这里不能设置为非阻塞，不然服务器没有响应完，客户端就关闭了
        //sc.configureBlocking(false);
        //发起连接
        sc.connect(new InetSocketAddress("localhost", 8070));

        //给服务器写数据
        sc.write(ByteBuffer.wrap("hello server".getBytes()));

        //接收服务器打回的数据
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        sc.read(buffer);
        System.out.println(new String(buffer.array(), 0, buffer.position()));

        //关流
        sc.close();
    }
}

```


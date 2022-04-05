[TOC]

# JUC

> concurrent [kənˈkʌrənt] 并发的

JUC是JDK1.5 中提供的一套并发包：

- java.util.concurrent
- java.util.concurrent.locks
- java.util.concurrent.atomic

JUC中包含的内容：

- BlockingQueue
- ConcurrentMap
- ExecutorService
- Lock
- Atomic



# 并发和并行

- 并发：指多个线程同时执行来抢占相同的资源

- 并行：指多个线程同时执行来使用不同的资源

  > 并行是一种特殊的并发

> 方便理解，举个栗子：
>
> - 并发：很多人同时在同一个窗口打饭
> - 并行：很多人同时在不同的窗口打饭
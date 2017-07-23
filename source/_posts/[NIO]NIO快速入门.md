---
title: NIO快速入门

date: 2017-06-23 16:36:00

categories:
- NIO

tags:
- NIO

---

## 为什么需要NIO

1、Java I/O是阻塞的，当线程调用read（）或write（）时，线程是阻塞的，直到数据读取、写入完毕，在此期间线程不能再干其他任何事情了。

2、这种方式对于小规模的程序非常方便，但是对于存在大量并发连接的时候，需要为每一个连接建立一个线程来操作，这种做法存在以下缺陷：

* 并发数与线程数成正比，线程是宝贵的系统资源，当线程数过大会导致系统的性能急剧下降，并发量有限
* 

## NIO和IO的区别

1、面向流与面向缓冲xxxxxxxxxxxx

IO是面向流的，NIO是面向缓冲区的。**Java IO**面向流意味着每次从流中读一个或多个字节，直至读取所有字节，它们没有被缓存在任何地方；**NIO**的将数据存放到一个缓冲区，当缓冲区中包含需要处理的数据

2、阻塞与非阻塞IOxxxxxxxxxxxxxxxxxx

**Java IO**的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入，该线程在此期间不能再干任何事情了。**Java NIO**的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

## NIO原理

除了普通的Socket与ServerSocket实现的阻塞式通信外，Java提供了非阻塞式通信的NIO API。先看一下NIO的实现原理，**NIO中三个重要的类：Selector、Channel、Buffer**

![](http://i.imgur.com/BYMk6vz.png)

从图中可以看出，服务器上所有Channel（包括ServerSocketChannel和SocketChannel）都需要向Selector注册，而该Selector则负责监视这些Socket的IO状态，当其中任意一个或者多个Channel具有可用的IO操作时，该Selector的select()方法将会返回大于0的整数，该整数值就表示该Selector上有多少个Channel具有可用的IO操作，并提供了selectedKeys（）方法来返回这些Channel对应的SelectionKey集合。正是通过Selector，使得服务器端只需要不断地调用Selector实例的select()方法即可知道当前所有Channel是否有需要处理的IO操作。

总结：

1、所有 Channel 需要向 Selector 注册
2、Selector 监听 Socket 的 IO 状态，当其中任意一个 Channel 具有可用的 IO 操作时，Selector 的 select（）方法返回大于0的整数，表示有多少个 Channel 具有可用的 IO 操作，并提供 selectedKeys 返回这些 Channel 对应的 SelectionKey 集合


## Server

## Client


## 参考

Java网络编程——使用NIO实现非阻塞Socket通信
http://blog.csdn.net/yanmei_yao/article/details/8586199

Java NIO 网络编程
https://my.oschina.net/gaoguofan/blog/753213

---
title: Thrift笔记

date: 2017-02-13 16:21:00

categories:
- Thrift

tags:
- Thrift

---

## Thrift 优点

1、跨语言
2、可扩展，简洁的四层接口抽象，每一层都可以独立的扩展增强或替换
3、二进制的编解码方式和NIO的底层传输为它提供了不错的性能

## 四层接口抽象

+-------------------------------------------+
| Server |
| (single-threaded, event-driven etc) |
+-------------------------------------------+
| Processor |
| (compiler generated) |
+-------------------------------------------+
| Protocol |
| (JSON, compact etc) |
+-------------------------------------------+
| Transport |
| (raw TCP, HTTP etc) |
+-------------------------------------------+

**Transport**层提供了一个简单的网络读写抽象层，有阻塞与非阻塞的TCP实现与HTTP的实现。

**Protocol**层定义了IDL中的数据结构与Transport层的传输数据格式之间的编解码机制。传输格式有二进制，压缩二进制，JSON等格式，IDL中的数据结构则包括Message，Struct，List，Map，Int，String，Bytes等。

**Processor**层由编译器编译IDL文件生成。
生成的代码会将传输层的数据**解码为参数对象**(比如商品对象有id与name两个属性，生成的代码会调用Protocol层的readInt与readString方法读出这两个属性值)，**然后调用由用户所实现的函数，并将结果编码送回。**

举个例子：

服务器端定义函数

	Product get(Integer id, String name){
		return new Product(id, name);
	}

1、客户端调用 get(id,name) 时，传递方法名get，参数id、name
2、服务器Transport接收到数据get、id、name
3、Processor调用Protocal的方法从Transport解码，从二进制转化为Integer和String，然后再服务器端调用get方法，得到结果Product

在服务端， **Server**层创建并管理上面的三层，同时提供线程的调度管理。而对于NIO的实现，Server层可谓操碎了心。

在客户端， Client层由编译器直接生成，也由上面的三层代码组成。只要语言支持，客户端有同步与异步两种模式。

## 四层接口的方法

### Transport层

**Transport**

TTransport除了open/close/flush，最重要的方法是int read(byte[] buf, int off, int len)，void write(byte[] buf, int off, int len)，读出、写入固定长度的内容。

* TSocket使用经典的JDK Blocking IO的Transport实现
* TNonblockingSocket，使用JDK NIO的Transport实现
* 相应的，TServerSocket和TNonblockingServerSocket是ServerTransport的BIO、NIO实现，主要实现侦听端口

**WrapperTransport**

包裹一个底层的Transport，并利用自己的Buffer进行额外的操作。

**作用：在数据传输时需要判断什么时候是读取的开始、什么时候是读取的末尾，即需要用分隔符进行标识；使用Frame帧的概念就可以解决这个问题**

### Protocol层

支持类型：

* 基本类型: i16，i32，i64, double, boolean，byte，byte[], String。
* 容器类型: List，Set，Map，TList/TSet/TMap类包含其元素的类型与元素的总个数。
* Struct类型，即面向对象的Class，继承于TBase。TStruct类有Name属性，还含有一系列的Field。TField类有自己的Name，类型，顺序id属性。
* Exception类型也是个Struct，继承于TException这个checked exception。
* Enum类型传输时是个i32。
* Message类型封装往返的RPC消息。TMessage类包含Name，类型(请求，返回，异常，ONEWAY)与seqId属性。

方法：

* Protocol 层对上述数据结构有read与write的方法
* 对基本类型是直接读写，对结构类型则是先调用readXXXBegin()，再调用其子元素的read()方法，再调用readXXXEnd()。
* 在所有函数中，Protocol层会直接调用Transport层读写特定长度的数据

### Processor层

具体流程：

TProcessFunction是生成的服务方法类的基类，它的process函数会完成如下步骤：

1. 调用生成的args对象的read方法从protocol层读出自己
2. 调用子类生成的getResult()方法：拆分args对象得到参数，调用真正的用户实现得到结果，并组装成生成的result对象。
3. 写消息头，
4. 调用生成的result对象的write方法将自己写入protocol层
5 调用transport层的flush()。

### Server层

基类TServer相当于一个容器，拥有生产TProcessor、TTransport、TProtocol的工厂对象。改变这些工厂类，可以修饰包裹Transport与Protocol类，改变TProcessor的单例模式或与Spring集成等等。

**Blocking Server**

* TSimpleServer同时只能处理一个Client连接，只是个玩具。
* TThreadPoolServer才是典型的多线程处理的Blocking Server实现。

**NonBlockingServer**

★★★TThreadedSelectorServer有一条线程专门负责accept，若干条Selector线程处理网络IO，一个Worker线程池处理消息。

很明显TThreadedSelectorServer是被使用得最多的，因为在多核环境下多条Selector线程的表现会更好

具体流程：

1、AcceptThread线程使用TNonblockingServerTransport执行**accept**操作，将accept到的Transport round-robin的交给其中一条SelectorThread
2、AcceptThread是先扔给SelectorThread里的Queue(默认长度只有4，满了就要阻塞等待)
3、

因为SelectorThread自己也要处理IO，所以

SelectorThread每个循环各执行一次如下动作
1. 注册Transport
2. select()处理IO
3. 处理FrameBuffer的状态变化

## Thrift网络服务模型（即Server层，详解）                     

**1、TSimpleServer**

TSimpleServer实现是非常的简单，**循环监听(即while(true))** 新请求的到来并完成对请求的处理，是个单线程阻塞模型。由于是**一次只能接收和处理一个socket连接**，效率比较低，在实际开发过程中很少用到它。

**2、TThreadPoolServer**

**ThreadPoolServer为解决了TSimpleServer不支持并发和多连接的问题, 引入了线程池。**但仍然是多线程阻塞模式即实现的模型是One Thread Per Connection。

线程池采用能线程数可伸缩的模式，线程池中的队列采用同步队列(SynchronousQueue)。

**ThreadPoolServer<font color='red'>拆分</font>了监听线程(accept)和处理客户端连接的工作线程(worker), 监听线程每接到一个客户端, 就<font color='red'>投给</font>线程池去处理。**

TThreadPoolServer模式优点：

线程池模式中，数据读取和业务处理都交由线程池完成，主线程只负责监听新连接，因此在并发量较大时新连接也能够被及时接受。线程池模式比较适合服务器端能预知最多有多少个客户端并发的情况，这时每个请求都能被业务线程池及时处理，性能也非常高。

TThreadPoolServer模式缺点：

**线程池模式的处理能力受限于线程池的工作能力，当并发请求数大于线程池中的线程数时，新请求也只能排队等待。**

**3、TNonblockingServer**

TNonblockingServer采用**单线程非阻塞(NIO)**的模式, 借助Channel/Selector机制, 采用IO事件模型来处理。**所有的socket都被注册到selector中，在一个线程中通过seletor循环监控所有的socket，**每次selector结束时，处理所有的处于就绪状态的socket，对于有数据到来的socket进行数据读取操作，对于有数据发送的socket则进行数据发送，对于监听socket则产生一个新业务socket并将其注册到selector中。

TNonblockingServer模式优点：

相比于TSimpleServer效率提升主要体现在IO多路复用上，TNonblockingServer采用非阻塞IO，同时监控多个socket的状态变化；

TNonblockingServer模式缺点：

**TNonblockingServer模式在业务处理上还是采用单线程顺序来完成，**在业务处理比较复杂、耗时的时候，例如某些接口函数需要读取数据库执行时间较长，此时该模式效率也不高，因为多个调用请求任务依然是顺序一个接一个执行。

**4、THsHaServer**

THsHaServer类是TNonblockingServer类的子类，为解决TNonblockingServer的缺点, THsHa引入了线程池去处理, 其模型**把读写任务放到线程池去处理即多线程非阻塞模式**。HsHa是: Half-sync/Half-async的处理模式, Half-aysnc是在处理IO事件上(accept/read/write io), Half-sync用于handler对rpc的同步处理上。因此可以认为THsHaServer半同步半异步。

THsHaServer的优点：

与TNonblockingServer模式相比，THsHaServer在完成数据读取之后，将业务处理过程交由一个线程池来完成，主线程直接返回进行下一次循环操作，效率大大提升；

THsHaServer的缺点：

主线程需要完成对所有socket的监听以及数据读写的工作，**当并发请求数较大时，且发送数据量较多时，监听socket上新连接请求不能被及时接受。**

**5、TThreadedSelectorServer**

**TThreadedSelectorServer是大家广泛采用的服务模型**，其多线程服务器端使用非堵塞式I/O模型，是对TNonblockingServer的扩充, 其分离了Accept和Read/Write的Selector线程, 同时引入Worker工作线程池。

（1）一个AcceptThread线程对象，专门用于处理**监听socket上的新连接**；

（2）若干个SelectorThread对象专门用于处理业务socket的网络I/O操作，所有网络数据的读写均是有这些线程来完成；**每个SelectorThread维护一个Socket队列，负责监听这些通道上的I/O**

（3）一个负载均衡器SelectorThreadLoadBalancer对象，主要用于AcceptThread线程接收到一个新socket连接请求时，决定将这个新连接请求分配给哪个SelectorThread线程。

（4）一个ExecutorService类型的工作线程池，在SelectorThread线程中，监听到有业务socket中有调用请求过来，则将请求读取之后，交个ExecutorService线程池中的线程完成此次调用的具体执行

![](https://i.imgur.com/C0VlbYX.png)

MainReactor就是Accept线程, 用于监听客户端连接, SubReactor采用IO事件线程(多个), 主要负责对所有客户端的IO读写事件进行处理. 而Worker工作线程主要用于处理每个rpc请求的handler回调处理(这部分是同步的)。因此其也是Half-Sync/Half-Async（半异步-半同步）的 。

TThreadedSelectorServer模式对于大部分应用场景性能都不会差，因为其有一个专门的线程AcceptThread用于处理新连接请求，因此能够及时响应大量并发连接请求；另外它将网络I/O操作分散到多个SelectorThread线程中来完成，因此能够快速对网络I/O进行读写操作，能够很好地应对网络I/O较多的情况。

## QM总结

![](https://i.imgur.com/xgi40Dl.jpg)

## 参考

http://calvin1978.blogcn.com/articles/apache-thrift.html
---
title: Java NIO 系列教程

date: 2017-03-09 16:40:00

categories:
- NIO

tags:
- NIO

---

转自 http://ifeve.com/java-nio-all/

## NIO是什么

Java NIO(**New IO**)是一个可以替代标准Java IO API的IO API（从Java 1.4开始)，**Java NIO提供了与标准IO不同的IO工作方式**。

1、Java NIO: Channels and Buffers（通道和缓冲区）

标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。

2、Java NIO: Non-blocking IO（非阻塞IO）
Java NIO可以让你非阻塞的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。

3、Java NIO: Selectors（选择器）
Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

## Java NIO 概述

Java NIO 由以下几个核心部分组成：

* Channels
* Buffers
* Selectors

### Channel 和 Buffer

基本上，所有的 IO 在NIO 中都从一个Channel 开始。Channel 有点象流。 数据可以从Channel读到Buffer中，也可以从Buffer 写到Channel中。这里有个图示：

![](http://i.imgur.com/OMUe99f.png)

Channel和Buffer有好几种类型。下面是JAVA NIO中的一些主要Channel的实现：

* FileChannel
* DatagramChannel
* SocketChannel
* ServerSocketChannel

正如你所看到的，这些通道涵盖了 UDP 和 TCP **网络IO**，以及**文件IO**。

与这些类一起的有一些有趣的接口，但为简单起见，我尽量在概述中不提到它们。本教程其它章节与它们相关的地方我会进行解释。

以下是Java NIO里关键的Buffer实现：

* ByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

这些Buffer覆盖了你能通过IO发送的基本数据类型：byte, short, int, long, float, double 和 char。

Java NIO 还有个 MappedByteBuffer，用于表示内存映射文件，我也不打算在概述中说明。

### Selector

**Selector允许单线程处理多个 Channel**。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中。

这是在一个单线程中使用一个Selector处理3个Channel的图示：

![](http://i.imgur.com/LXlI8Ux.png)

要使用Selector，得**向Selector注册Channel，然后调用它的select()方法**。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。

## Channel

### Channel与流的区别

<font color='red'>**Java NIO的通道类似流，但又有些不同：**</font>

* 既可以从通道中**读取**数据，又可以**写数据**到通道。但流的读写通常是**单向的**
* 通道可以**异步**地读写
* 通道中的数据总是要先读到一个**Buffer**，或者总是要从一个Buffer中写入

正如上面所说，从通道读取数据到缓冲区，从缓冲区写入数据到通道。如下图所示：

![](http://i.imgur.com/OMUe99f.png)

### Channel的实现

这些是Java NIO中最重要的通道的实现：

* FileChannel
* DatagramChannel
* SocketChannel
* ServerSocketChannel
* FileChannel 从文件中读写数据。

DatagramChannel 能通过UDP读写网络中的数据。

SocketChannel 能通过TCP读写网络中的数据。

ServerSocketChannel可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。

### 基本的 Channel 示例

下面是一个使用FileChannel读取数据到Buffer中的示例：

	//文件通道
	String path = APP.class.getClassLoader().getResource("nio-data.txt").getPath();
	RandomAccessFile aFile = new RandomAccessFile(path, "rw");
	FileChannel inChannel = aFile.getChannel();
	//缓冲区
	ByteBuffer buf = ByteBuffer.allocate(48);
	//通道向缓冲区写数据，每次循环得到ByteBuffer -> byte[] -> ByteArrayOutputStream把每段byte[]拼接起来 ->byte[]
	int bytesRead = -1;
	ByteArrayOutputStream result = new ByteArrayOutputStream();
	while ((bytesRead =  inChannel.read(buf)) != -1) {
	    //注意 buf.flip() 的调用，首先读取数据到Buffer，然后反转Buffer,接着再从Buffer中读取数据（指针移到开始位置）
	    buf.flip();
	    result.write(buf.array(),0, bytesRead);
	    //一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入
	    buf.clear();
	}
	//输出结果
	System.out.println(result.toString());
	//关闭资源
	result.close();
	inChannel.close();
	aFile.close();

## Buffer

Java NIO中的Buffer用于和NIO通道进行交互。如你所知，数据是从通道读入缓冲区，从缓冲区写入到通道中的。

**缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。**

### Buffer的基本用法

使用Buffer读写数据一般遵循以下四个步骤：

* 写入数据到Buffer
* 调用flip()方法
* 从Buffer中读取数据
* 调用clear()方法或者compact()方法

当向buffer写入数据时，buffer会记录下写了多少数据。一旦要读取数据，需要**通过flip()方法将Buffer从写模式切换到读模式**。在读模式下，可以读取之前写入到buffer的所有数据。(不能反过来)

一旦读完了所有的数据，就需要**清空缓冲区，让它可以再次被写入**。有两种方式能清空缓冲区：调用clear()或compact()方法。clear()方法会清空整个缓冲区。compact()方法只会清除已经读过的数据。任何未读的数据都被移到缓冲区的起始处，新写入的数据将放到缓冲区未读数据的后面。

下面是一个使用Buffer的例子：

```
RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
FileChannel inChannel = aFile.getChannel();
//create buffer with capacity of 48 bytes
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf); //read into buffer.
while (bytesRead != -1) {
  buf.flip();  //make buffer ready for read
  while(buf.hasRemaining()){
      System.out.print((char) buf.get()); // read 1 byte at a time
  }
  buf.clear(); //make buffer ready for writing
  bytesRead = inChannel.read(buf);
}
aFile.close();
```

### Buffer的capacity,position和limit

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。

为了理解Buffer的工作原理，需要熟悉它的三个属性：

* capacity
* position
* limit

**position和limit的含义取决于Buffer处在读模式还是写模式。**不管Buffer处在什么模式，capacity的含义总是一样的。

这里有一个关于capacity，position和limit在读写模式中的说明，详细的解释在插图后面。

![](http://i.imgur.com/EVfAOgu.png)

capacity

作为一个内存块，**Buffer有一个固定的大小值**，也叫"capacity"。你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。

position

当你写数据到Buffer中时，position表示当前的位置。初始的position值为0。当一个byte、long等数据写到Buffer后，**position会向前移动到下一个可插入数据的Buffer单元**。position最大可为capacity – 1.

当读取数据时，也是从某个特定位置读。**当将Buffer从写模式切换到读模式，position会被重置为0**。 当从Buffer的position处读取数据时，position向前移动到**下一个可读的位置**。

limit

在写模式下，Buffer的limit表示你**最多能往Buffer里写多少数据**。 写模式下，limit等于Buffer的capacity。

当切换Buffer到读模式时， limit表示你**最多能读到多少数据**。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）

### Buffer的类型

Java NIO 有以下Buffer类型

* ByteBuffer
* MappedByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer

如你所见，这些Buffer类型代表了不同的数据类型。换句话说，就是可以通过char，short，int，long，float 或 double类型来操作缓冲区中的字节。

MappedByteBuffer 有些特别，在涉及它的专门章节中再讲。

### Buffer的分配

要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法

```
//分配48字节capacity的ByteBuffer的
ByteBuffer buf = ByteBuffer.allocate(48);
```

```
//分配一个可存储1024个字符的CharBuffer
CharBuffer buf = CharBuffer.allocate(1024);
```

### 向Buffer中写数据

写数据到Buffer有两种方式：

* 从Channel写到Buffer。
* 通过Buffer的put()方法写到Buffer里。

从Channel写到Buffer的例子

```
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf); //read into buffer.
```

通过put方法写Buffer的例子：

```
buf.put(127);
```

put方法有很多版本，允许你以不同的方式把数据写入到Buffer中。例如，写到一个指定的位置，或者把一个字节数组写入到Buffer。 更多Buffer实现的细节参考JavaDoc。

### flip()方法

flip方法将Buffer**从写模式切换到读模式**。调用flip()方法会**将position设回0**，并**将limit设置成之前position的值**。

换句话说，position现在用于标记读的位置，limit表示之前写进了多少个byte、char等 —— 现在能读取多少个byte、char等。

### 从Buffer中读取数据

从Buffer中读取数据有两种方式：

* 从Buffer读取数据到Channel。
* 使用get()方法从Buffer中读取数据。

从Buffer读取数据到Channel的例子：

```
//read from buffer into channel.
int bytesWritten = inChannel.write(buf);
```

使用get()方法从Buffer中读取数据的例子

```
byte aByte = buf.get();
```

get方法有很多版本，允许你以不同的方式从Buffer中读取数据。例如，从指定position读取，或者从Buffer中读取数据到字节数组。更多Buffer实现的细节参考JavaDoc。

### rewind()方法

Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。

### clear()与compact()方法

一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。

如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。**Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。**

如果Buffer中有一些未读的数据，调用clear()方法，数据将"被遗忘"，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。

**如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。**

compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是**不会覆盖未读的数据**。

### mark()与reset()方法

通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。(注意，使用flip方法会重置mark为-1)例如：

```
buffer.mark();
//call buffer.get() a couple of times, e.g. during parsing.
buffer.reset();  //set position back to mark.
```

### equals()与compareTo()方法

可以使用equals()和compareTo()方法两个Buffer。

equals()

当满足下列条件时，表示两个Buffer相等：

* 有相同的类型（byte、char、int等）。
* Buffer中剩余的byte、char等的个数相等。
* Buffer中所有剩余的byte、char等都相同。

如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。实际上，它只**比较Buffer中的剩余元素。**

compareTo()方法

compareTo()方法**比较两个Buffer的剩余元素(byte、char等)**， 如果满足下列条件，则认为一个Buffer"小于"另一个Buffer：

* 第一个不相等的元素小于另一个Buffer中对应的元素 。
* 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。
（译注：剩余元素是从 position到limit之间的元素）

## Scatter/Gather

Java NIO开始支持scatter/gather，scatter/gather用于描述从Channel（译者注：Channel在中文经常翻译为通道）中读取或者写入到Buffer中的操作。

* 分散（scatter）从Channel中读取是指在读操作时**将读取的数据写入多个buffer中**。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。
* 聚集（gather）写入Channel是指在写操作时**将多个buffer的数据写入同一个Channel**，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

### Scattering Reads

Scattering Reads是指数据从一个channel读取到多个buffer中。如下图描述：

![](http://i.imgur.com/5gzGOOT.png)

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = { header, body };
channel.read(bufferArray);
```

注意buffer首先被插入到数组，然后再将数组作为channel.read() 的输入参数。read()方法按照buffer在数组中的顺序将从channel中读取的数据写入到buffer，**当一个buffer被写满后，channel紧接着向另一个buffer中写。**

Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息(译者注：消息大小不固定)。换句话说，如果存在消息头和消息体，消息头必须完成填充（例如 128byte），Scattering Reads才能正常工作。

### Gathering Writes

Gathering Writes是指数据从多个buffer写入到同一个channel。如下图描述：

![](http://i.imgur.com/oP8lagL.png)

```
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body   = ByteBuffer.allocate(1024);
//write data into buffers
ByteBuffer[] bufferArray = { header, body };
channel.write(bufferArray);
```

buffers数组是write()方法的入参，write()方法会按照buffer在数组中的顺序，将数据写入到channel，注意只有position和limit之间的数据才会被写入。因此，如果一个buffer的容量为128byte，但是仅仅包含58byte的数据，那么这58byte的数据将被写入到channel中。因此与Scattering Reads相反，Gathering Writes能较好的处理动态消息。

## 通道之间的数据传输

在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel（译者注：channel中文常译作通道）传输到另外一个channel。

### transferFrom()

FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中（译者注：这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。下面是一个简单的例子：

    String fromFilePath = TransferToDemo.class.getClassLoader().getResource("fromFile.txt").getPath();
    String toFilePath = TransferToDemo.class.getClassLoader().getResource("toFile.txt").getPath();

    RandomAccessFile fromFile = new RandomAccessFile(fromFilePath, "rw");
    FileChannel fromChannel = fromFile.getChannel();
    RandomAccessFile toFile = new RandomAccessFile(toFilePath, "rw");
    FileChannel toChannel = toFile.getChannel();

    long position = 0;
    long count = fromChannel.size();
    toChannel.transferFrom(fromChannel, position, count);

方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。
此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

### transferTo()

transferTo()方法将数据从FileChannel传输到其他的channel中。下面是一个简单的例子：

	String fromFilePath = TransferToDemo.class.getClassLoader().getResource("fromFile.txt").getPath();
	String toFilePath = TransferToDemo.class.getClassLoader().getResource("toFile.txt").getPath();
	//打开通道
	RandomAccessFile fromFile = new RandomAccessFile(fromFilePath, "rw");
	FileChannel fromChannel = fromFile.getChannel();
	RandomAccessFile toFile = new RandomAccessFile(toFilePath, "rw");
	FileChannel toChannel = toFile.getChannel();
	//传输数据
	long position = 0;
	long count = fromChannel.size();
	fromChannel.transferTo(position, count, toChannel);

是不是发现这个例子和前面那个例子特别相似？除了调用方法的FileChannel对象不一样外，其他的都一样。
上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。

## Selector

Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，**一个单独的线程可以管理多个channel，从而管理多个网络连接**。

### 为什么使用Selector?

仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，**可以只用一个线程处理所有的通道**。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。

但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。不管怎么说，关于那种设计的讨论应该放在另一篇不同的文章中。在这里，只要知道使用Selector能够处理多个通道就足够了。

下面是单线程使用一个Selector处理3个channel的示例图：

![](http://i.imgur.com/LXlI8Ux.png)

### Selector的创建
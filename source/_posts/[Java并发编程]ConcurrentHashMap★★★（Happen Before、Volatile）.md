---
title: ConcurrentHashMap★★★（Happen Before、Volatile）

date: 2017-06-27 10:00:00

categories:
- Java并发编程

tags:
- Java

---

## 结构

![](http://i.imgur.com/8TDvzvH.jpg)

Segment 类继承于 ReentrantLock 类，从而使得 Segment 对象能充当锁的角色。每个 Segment 对象用来守护其（成员对象 table 中）包含的若干个桶。

## 并发读写考虑

1、**非结构性修改操作**只是更改某个 HashEntry 的 value 域的值。由于对 **Volatile** 变量的写入操作将与随后对这个变量的读操作进行同步。当一个写线程修改了某个 HashEntry 的 value 域后，另一个读线程读这个值域，Java 内存模型能够保证读线程读取的一定是更新后的值。所以，写线程对链表的非结构性修改能够被后续不加锁的读线程"看到"。

2、put-读

情况一：key已经存在，put操作为非结构性修改，查看1

情况二：新插入节点

put 操作如果需要插入一个新节点到链表中时 , 会在链表头部插入这个新节点。此时，链表中的原有节点的链接并没有被修改。

采用头插法插入节点不影响读，**每次遍历都是从头结点table[x]开始**，即不影响读的遍历

![](http://i.imgur.com/4BLoPBR.png)

3、remove-读

![](http://i.imgur.com/jzXlXZm.jpg)

![](http://i.imgur.com/wan6AGT.jpg)

和 get 操作一样，首先根据散列码找到具体的链表；然后遍历这个链表找到要删除的节点；**最后把待删除节点之后的所有节点原样保留在新链表中，把待删除节点之前的每个节点克隆到新链表中，直到复制完之后将table引用过去**

下面通过图例来说明 remove 操作。假设写线程执行 remove 操作，要删除链表的 C 节点，另一个读线程同时正在遍历这个链表

在执行 remove 操作时，**原始链表并没有被修改**，也就是说：读线程不会受同时执行 remove 操作的并发写线程的干扰。

4、clear-读

	void clear() {
	    if (count != 0) {
	        lock();
	        try {
	            HashEntry<K,V>[] tab = table;
	            for (int i = 0; i < tab.length ; i++)
	                tab[i] = null;
	            ++modCount;
	            count = 0; // write-volatile
	        } finally {
	            unlock();
	        }
	    }
	}

clear 操作只是把 ConcurrentHashMap 中所有的桶"置空"，**每个桶之前引用的链表依然存在**，只是桶不再引用到这些链表（所有链表的结构并没有被修改）。正在遍历某个链表的读线程依然可以正常执行对该链表的遍历。

## ConcurrentHashMap的get操作(volatile)★★★★★

**get操作的高效之处在于整个get过程不需要加锁，除非读到的值是空的才会加锁重读？**，我们知道HashTable容器的get方法是需要加锁的，那么ConcurrentHashMap的get操作是如何做到不加锁的呢？

原因是它的**get方法里将要使用的共享变量都定义成volatile**，如用于统计当前Segement大小的 **count** 字段和用于存储值的 **HashEntry 的 value**。
<br/>

<font color='red'>定义成volatile的变量，能够在线程之间保持可见性，能够被多线程同时读，并且保证不会读到过期的值，但是只能被**单线程写**（有一种情况可以被**多线程写，就是写入的值不依赖于原值**），在get操作里只需要读不需要写共享变量count和value，所以可以不用加锁。</font>

**之所以不会读到过期的值，是根据 java 内存模型的 happen before 原则，对 volatile 字段的写入操作先于读操作**，即使两个线程同时修改和获取volatile变量，get操作也能拿到最新的值，这是用volatile替换锁的经典应用场景。

**源码分析：**

	V get(Object key, int hash) {
	    if (count != 0) { // read-volatile // ①
	        HashEntry<K,V> e = getFirst(hash); 
	        while (e != null) {
	            if (e.hash == hash && key.equals(e.key)) {
	                V v = e.value;
	                if (v != null)  // ② 注意这里
	                    return v;
	                return readValueUnderLock(e); // recheck
	            }
	            e = e.next;
	        }
	    }
	    return null;
	}

第一步，先判断一下 count != 0；count变量表示segment中存在entry的个数。如果为0就不用找了。

假设这个时候恰好另一个线程put或者remove了这个segment中的一个entry，会不会导致两个线程看到的count值不一致呢？

看一下count变量的定义： 

	transient volatile int count;

它使用了volatile来修改。我们前文说过，Java5之后，JMM实现了对volatile的保证：**对volatile域的写入操作happens-before于每一个后续对同一个域的读写操作。**所以，每次判断count变量的时候，即使恰好其他线程改变了segment也会体现出来。

第二步，获取到要该key所在segment中的索引地址，如果该地址有相同的hash对象，顺着链表一直比较下去找到该entry。当找到entry的时候，先做了一次比较： if(v != null) ，这是为何呢？即**如果get得到的结果是null，需要加锁读，否则直接返回**

考虑一下，如果这个时候，另一个线程恰好新增/删除了entry，或者改变了entry的value，会如何？

1) 在get代码的①和②之间，另一个线程新增了一个entry

![](http://i.imgur.com/jO3Ir3q.jpg)

因为每个HashEntry中的next也是final的，没法对链表最后一个元素增加一个后续entry所以新增一个entry的实现方式只能通过头结点来插入了。

newEntry对象是通过 new HashEntry(K k , V v, HashEntry next) 来创建的。如果另一个线程刚好 <font color='red'>**new**</font> 这个对象时，当前线程来  <font color='red'>**get**</font> 它。**因为没有同步，就可能会出现当前线程得到的newEntry对象是一个没有完全构造好的对象引用。**

回想一下我们之前讨论的DCL的问题，这里也一样，没有锁同步的话，new 一个对象对于多线程看到这个对象的状态是没有保障的，这里同样有可能一个线程new这个对象的时候还没有执行完构造函数就被另一个线程得到这个对象引用。

所以才需要判断一下：if (v != null) 如果确实是一个不完整的对象，则使用锁的方式再次get一次。

**有没有可能会put进一个value为null的entry？ 不会的，已经做了检查，这种情况会抛出异常**，所以 ②处的判断完全是出于对多线程下访问一个new出来的对象的状态检测。

<font color='red'>注意：是否会出现 v！=null，但是返回的对象没有初始化，存在隐患？</font>

如果对象HashEntry没有被volatile修饰，会出现这种情况。但是查看定义，**HashEntry[]被volatile修饰。**

那么为什么不被volatile修饰，new HashEntry会出现引用不为null，但是对象未被初始化？

因为new的操作不是原子操作，分为1、分配内存 2、实例化 3、引用指向内存（此时不为null）

**但是指令重排序后，可能会出现1，3，2，此时在3是时候读取对象时不为null，但是对象未被初始化，出现隐患**

使用volatile修饰后，根据happen before原则，读操作必须在写之后，而且读线程是可见的

**所以，如果得到对象！=null，说明找到对象了，如果=null，可能是不存在，也可能是刚刚添加进去，等到put线程更新table指针，注意此时返回的元素是头指针，如果查找的就是刚刚添加的元素，释放锁后就能返回插入的结果**

2) 在get代码的①和②之间，另一个线程修改了一个entry的value

**value是用volitale修饰的，可以保证读取时获取到的是修改后的值。**

3) 在get代码的①之后，另一个线程删除了一个entry

假设我们的链表元素是：e1-> e2 -> e3 -> e4 我们要删除 e3这个entry，因为HashEntry中next的不可变，所以我们无法直接把e2的next指向e4，而是将要删除的节点之前的节点复制一份，形成新的链表。

它的实现大致如下图所示：

![](http://i.imgur.com/aNjCXWM.jpg)

如果我们get的也恰巧是e3，可能我们顺着链表刚找到e1，这时另一个线程就执行了删除e3的操作，**而我们线程还会继续沿着旧的链表找到e3返回。这里没有办法实时保证了。**

我们第①处就判断了count变量，它保障了在 ① 处能看到其他线程修改后的。①之后到②之间，如果再次发生了其他线程再删除了entry节点，就没法保证看到最新的了。

不过这也没什么关系，即使我们返回e3的时候，它被其他线程删除了，暴漏出去的e3也不会对我们新的链表造成影响。

这其实是一种乐观设计，设计者假设 ① 之后到 ② 之间发生被其它线程增、删、改的操作可能性很小，所以不采用同步设计，而是采用了事后（其它线程这期间也来操作，并且可能发生非安全事件）弥补的方式。

而<font color='red'>**因为其他线程的"改"和"删"对我们的数据都不会造成影响，所以只有对"新增"操作进行了安全检查，就是②处的非null检查**</font>，如果确认不安全事件发生，则采用加锁的方式再次get。

## 总结★★★★★

ConcurrentHashMap 的高并发性主要来自于三个方面：

* 用**分离锁**实现多个线程间的更深层次的共享访问。
* **用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。**（就是说当需要对链表结构进行改变时，对链表进行复制，先不更改原来的链表，然后操作完成之后再把table切换过去，在put/remove中体现）
* 通过对同一个 **Volatile** 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。

## Happen-Before

① **程序次序法则**：线程中的每个动作A都happens-before于该线程中的每一个动作B，其中，在程序中，所有的动作B都能出现在A之后。
② **监视器锁法则**：对一个监视器锁的解锁 happens-before于每一个后续对同一监视器锁的加锁。
③ volatile变量法则：**对volatile域的写入操作happens-before于每一个后续对同一个域的读写操作。**
④ 线程启动法则：在一个线程里，对Thread.start的调用会happens-before于每个启动线程的动作。
⑤ 线程终结法则：线程中的任何动作都happens-before于其他线程检测到这个线程已经终结、或者从Thread.join调用中成功返回，或Thread.isAlive返回false。
⑥ 中断法则：一个线程调用另一个线程的interrupt happens-before于被中断的线程发现中断。
⑦ 终结法则：一个对象的构造函数的结束happens-before于这个对象finalizer的开始。
⑧ **传递性**：如果A happens-before于B，且B happens-before于C，则A happens-before于C

**我们重点关注的是②，③，这两条也是我们通常编程中常用的。**

### 加锁

使用锁方式实现"Happens-before"是最简单，容易理解的。

![](http://i.imgur.com/Y2duLzx.jpg)

早期Java中的锁只有最基本的synchronized，它是一种互斥的实现方式。在Java5之后，增加了一些其它锁，比如ReentrantLock，它基本作用和synchronized相似，但提供了更多的操作方式，比如在获取锁时不必像synchronized那样只是**傻等，可以设置定时，轮询，或者中断**，这些方法使得它在获取多个锁的情况可以避免死锁操作。

### Volatile

JMM对Volatile的定义是：保证读写volatile都直接发生在main memory中，线程的working memory不进行缓存。它只承诺了读和写过程的可见性

Volatile可以看做一种轻量级的锁，但又和锁有些不同。

a) 它对于多线程，不是一种互斥（mutex）关系。
b) 用volatile修饰的变量，不能保证该变量状态的改变对于其他线程来说是一种"原子化操作"。

那对于"原子化操作"怎么理解呢？看下面例子：

	private static volatile int nextSerialNum = 0;
	
	public static int generateSerialNumber(){
	    return nextSerialNum++;
	}

上面代码中对nextSerialNum使用了volatile来修饰，根据前面"Happens-Before"法则的第三条Volatile变量法则，看似不同线程都会得到一个新的serialNumber

**问题出在了 nextSerialNum++ 这条语句上，它不是一个原子化的**，实际上是read-modify-write三项操作，**这就有可能使得在线程1在write之前，线程2也访问到了nextSerialNum**，造成了线程1和线程2得到一样的serialNumber。所以，在使用Volatile时，需要注意

使用场景：

* 单线程写
* 被多线程写，就是写入的值不依赖于原值

### final关键字

不变模式（immutable）是多线程安全里最简单的一种保障方式。因为你拿他没有办法，想改变它也没有机会。

不变模式主要通过final关键字来限定的。在JMM中final关键字还有特殊的语义。Final域使得确保初始化安全性（initialization safety）成为可能，初始化安全性让**不可变形对象不需要同步就能自由地被访问和共享。**

### 用Happens-Before规则理解一个经典问题：双重检测锁(DCL)为什么在java中不适用？

	public class LazySingleton {

	    private static LazySingleton instance;
	
		private LazySingleton(){}

	    public static LazySingleton getInstance() {
	        if (instance == null) {// (2)
	            synchronized (LazySingleton.class) { // (3)
	              if (instance == null) { // (4)
	                instance = new LazySingleton(); // (5)
	              }
	            }
	        }
	        return instance; // (6)
	    }
	
	}

假设线程1执行完(5)时，线程2正好执行到了(2)；

看看 new LazySingleton(); 这个语句的执行过程： 它不是一个原子操作，实际是由多个步骤，我们从我们关注的角度简化一下，简单的认为它主要有2步操作好了：

a） 在内存中分配空间，并将引用指向该内存空间。
b） 执行对象的**初始化**的逻辑(和操作)，完成对象的构建。

此时因为线程1和线程2没有用同步，他们之间不存在"Happens-Before"规则的约束，所以在线程1创建LazySingleton对象的 a),b)这两个步骤对于线程2来说会有可能出现a)可见，b)不可见
造成了**线程2获取到了一个未创建完整的lazySingleton对象引用**，为后边埋下隐患。

## 思考

多线程程序设计时需要考虑读-读、读-写、写-写

可以并发读：读不加锁
不能并发写：写加锁
写的时候不能影响正在读，或从头开始遍历的读

## 参考

http://blog.csdn.net/jianghuxiaojin/article/details/52006110

http://blog.csdn.net/seapeak007/article/details/53409618
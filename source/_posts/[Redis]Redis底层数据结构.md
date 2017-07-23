---
title: Redis底层数据结构

date: 2017-07-17 21:45:00

categories:
- Redis

tags:
- Redis

---

## Redis单进程单线程执行速度快的原因

总体来说快速的原因如下：

1）绝大部分请求是纯粹的内存操作（非常快速）   
2）采用单线程,避免了不必要的上下文切换和竞争条件
3）非阻塞IO

内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间    

这3个条件不是相互独立的，特别是第一条，如果请求都是耗时的，采用单线程吞吐量及性能可想而知了。应该说redis为特殊的场景选择了合适的技术方案。

## Redis数据结构

### 总体设计

首先，Redis没有MySQL那样的索引机制，因为其内建一个基于hash的字典，如下图：

![](http://i.imgur.com/u4CAMEI.jpg)

Redis 计算哈希值和索引值的方法如下：

\# 使用字典设置的哈希函数，计算键 key 的哈希值
hash = dict->type->hashFunction(key);

\# 使用哈希表的 sizemask 属性和哈希值，计算出索引值
\# 根据情况不同， ht[x] 可以是 ht[0] 或者 ht[1]
index = hash & dict->ht[x].sizemask;
插入数据时，根据以上算出index，然后根据index值放入table表中相应位置即可。

**QM注释：**
1、每个dictht为一个map结构，字典dict中有两个map
2、map为key、value的结构，value可以为各种数据类型，包括字符串、链表...

### 字符串

例如：Set hello world 

![](http://i.imgur.com/cb0jdYP.png)

字符串类似Java中的String对象，存储类似HashMap，key为字符串的名字，value为String对象

### list类型

例如：Lpush list aaaa bbb ccc

![](http://i.imgur.com/kGPk6LE.png)

	// 双向链表
	typedef struct listNode {
	    struct listNode *prev;
	    struct listNode *next;
	    void *value;
	} listNode;
	
	//数据结构 + 操作
	typedef struct list {
	    listNode *head;
	    listNode *tail;
		...
	} list;

链表类似Java中的LinkedList，定义每个链表时取一个名称；存储类似HashMap，key为链表的名称，value为list对象

### hash类型

例如：Hset test hello world

![](http://i.imgur.com/udIIoGc.png)

存储类似HashMap，key为hash表的名称，value为hash表

注：新建一个hash对象时开始是用zipmap(又称为small hash)来存储的。这个zipmap其实并不是hash table，但是zipmap相比正常的hash实现可以节省不少hash本身需要的一些元数据存储开销。尽管zipmap的添加，删除，查找都是O(n)，但是由于一般对象的field数量都不太多。所以使用zipmap也是很快的,也就是说添加删除平均还是O(1)。如果field或者value的大小超出一定限制后，Redis会在内部自动将zipmap替换成正常的hash实现（一个key对应一个hash表）。

### Set

集合，常用来存储不重复数据的数据结构，底层基于hashtable；在redis中为了优化存储，set的编码类型可以为：intset，hashtable。

### Sorted Set(ZSet)

链表+跳跃表

https://mp.weixin.qq.com/s/RDHebf6IfLWfw38dPvfiZQ

### 参考

http://www.cnblogs.com/renzherushe/p/4779390.html

## Redis过期检测

Redis对于过期检测，有2种方式，一个主动检测，一个是被动检测；在redis和client交互过程中，对于任何数据的操作，都会首先检测key是否已经过期，这是被动检测；主动检测是Redis启动的后台线程中，不间断的随机扫描一定量的key（randomKey），并对key进行过期检测

---
title: Java多线程面试题

date: 2017-09-04 20:15:00

categories:
- Java并发编程

tags:
- 面试题

---
## synchronized 和 lock 的区别

1、synchronized在JVM层面上实现，隐式加锁、释放锁，执行完同步代码或者出现异常都会自动释放锁，lock是代码显式加锁，释放锁，如果要保证锁定一定会被释放，就必须将unLock()放到finally{}中

2、ReentrantLock 拥有 Synchronized 相同的并发性和内存语义，此外还多**锁投票，定时锁等候和中断锁等候**；

  线程A和B都要获取对象O的锁定，假设A获取了对象O锁，B将等待A释放对O的锁定，
  如果使用 synchronized ，如果A不释放，B将一直等下去，不能被中断
  如果 使用ReentrantLock，如果A不释放，可以使B在等待了足够长的时间以后，中断等待，而干别的事情

## 线程池的好处

1、减少在创建和销毁线程上所花的时间以及系统资源的开销
2、方便线程资源的管理

## 线程池的流程

* 线程池在启动的时候初使化一定数量的线程
* 有任务过来的时候，先检测初使化的线程还有空的没有，如果有就取一个线程执行任务；没有就再看当前运行中的线程数是不是已经达到了最大数，如果没有，就新分配一个线程去处理；但如果已经达到了最大数，任务就只有等待了

## 四种线程池

**newCachedThreadPool** 

* 线程池中如果有闲置的线程将重用，没有可用的线程，则创建一个新线程并添加到池中（无上限）。
* 自动回收不使用的线程（终止并从缓存中移除那些已有 60 秒钟未被使用的线程）

优点：在小任务量，任务时间执行短的场景下能提高性能
缺点：


**newFixedThreadPool**创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
**newScheduledThreadPool**创建一个定长线程池，支持定时及周期性任务执行。
**newSingleThreadExecutor**创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

## new Thread 与 newSingleThreadPool 的区别

* 对于单个、确定的任务，使用new Thread
* 对于多个任务，并且要求FIFO执行，使用SingleThreadPool

因为执行多任务，new Thread 无法复用线程，需要频繁的创建、销毁线程，耗费系统资源

对于单一确定的任务，使用SingleThreadPool效率比new Thread低，因为SingleThreadPool需要维护任务队列等其他开销

参考：http://bbs.csdn.net/topics/391830535

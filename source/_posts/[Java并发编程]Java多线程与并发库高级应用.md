---
title: Java多线程与并发库高级应用

date: 2016-12-14 16:00:00

categories:
- Java并发编程

tags:
- Java
- 多线程
- 并发库
- 并发编程

---

## 多线程与效率

* 多线程并不一定会提高程序的运行效率，反而会降低效率
* 多线程的应用不是为了提高运行效率，而是为了提高资源使用效率；当一个资源在等待I/O输入时，另一个线程可以使用处理器，提高了资源的利用率

为什么使用多线程下载速度会变快？
多线程下载：并不是自己电脑快了，而是**抢到更多服务器资源**。例：服务器为一个客户分配一个20K的线程下载，你用多个线程，服务器以为是多个用户就分配了多个20K的资源给你

## 并行与并发

* 并行指多个事件在**同一时刻发生**，并行需要有多个CPU
* 并行是在同一个CPU上按照时间片的划分运行多个程序，在宏观上看两个任务同时执行，但是**在同一时刻只有一个任务在执行**

## 并发编程的挑战

* 并发编程的 **目的** 是让程序运行得更快，**但是并不是启动越多的线程就能让程序跑的更快**
* 如果希望多线程执行任务使程序运行得更快，需要面临许多挑战，比如：**上下文切换、死锁、资源限制问题**

### 上下文切换

***多线程***

* CPU为每个线程划分 **时间片**，每个线程在给定的时间片上交替运行。因为时间片非常短，从宏观上我们感觉多个线程同时运行
* **单核** 处理器也支持多线程执行代码

***上下文切换概念***

每个线程在给定的时间片上运行，执行结束后会 **切换** 到下一个任务。**但是在切换前会保存上一个任务的状态**，以便下次切换回来后可以 **继续加载** 这个任务的状态，从保存到加载的过程称为一次上下文切换

***多线程速度一定快？***

不一定，用得得当多线程的任务执行速度比串行快；但是线程有**创建和上下文切换**的开销，有时候并发执行速度比串行慢

***如何减小上下文切换***

* 无锁并发编程：多线程竞争锁时会引起上下文切换，可以用一些方法来避免使用锁，如数据分段，每个线程处理不同段的数据
* CAS算法：Java的Atomic包使用CAS算法来更新数据，而不需要使用锁
* 使用最少线程：避免创造不需要的线程，比如任务少，线程多，造成大量线程处于等待状态
* 协程：在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换

### 死锁

举个例子

	Thread t1 = new Thread(new Runnable(){
		@Override
		public void run(){
			synchronized(A){
				Thread.sleep(2000);
				synchronized(B){}
			}
		}
	});
	
	Thread t2 = new Thread(new Runnable(){
		@Override
		public void run(){
			synchronized(B){
				Thread.sleep(2000);
				synchronized(A){}
			}
		}
	});

线程1：持有锁A，等待锁B
线程2：持有锁B，等待锁A

线程1和线程2互相等待对方释放锁，引起死锁

***避免死锁的一些方法***

* 避免一个线程同时获取多个锁
* 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只占用一个资源
* 尝试使用定时锁，使用
* .tryLock(timeout)来替代使用内部锁机制
* 对于数据库锁，加锁和解锁必须在一个数据库连接里，否则会出现解锁失败的情况

## 线程状态

* 线程创建完毕后进入**new**状态
* 调用了线程的start方法后，线程进入**Runnable**状态，放入jVM的**可运行线程队列**中，等待CPU的执行权
* 当线程执行时，线程状态变为**Running**
* 线程执行sleep、wait、join或者进入了IO阻塞、锁等待时，则进入**Wait或Block**状态

## 创建线程的两种传统方式

传统是相对于 JDK 1.5 而言的

### 继承Thread

创建Thread的子类，覆盖其中的run方法，运行这个子类的 start 方法即可开启线程

	public class ThreadTest1 {
	
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			Thread thread = new Thread() {
				@Override
				public void run() {
					while (true) {
						// 获取当前线程对象 获取线程名字
						System.out.println(Thread.currentThread().getName());
						// 让线程暂停，休眠，此方法会抛出中断异常InterruptedException
						try {
							Thread.sleep(1000);
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					}
				}
			};
			thread.start();
		}
	}

### 实现Runnable接口

创建Thread时传递一个实现Runnable接口的对象实例(用的比较多，因为符合面向对象编程的思路)

	public class ThreadTest1 {
	
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			Thread thread = new Thread(new Runnable() {
				@Override
				public void run() {
					while (true) {
						System.out.println(Thread.currentThread().getName());
						try {
							Thread.sleep(1000);
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
					}
				}
			});
			thread.start();
		}
	}

### 面试题★★★

问题：下边的线程运行的是Thread子类中的run方法还是实现Runnable接口类的run方法


	public class ThreadTest1 {
	
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			new Thread(new Runnable() {////b、传递实现Runnable接口的对象
				
				@Override
				public void run() {
					// TODO Auto-generated method stub
					System.out.println("2");
				}
			}){//a、覆盖Thread子类run方法
				@Override
				public void run() {
					System.out.println("1");
				}
			}.start();
		}
	}

分析：查看Thread的源代码，注意run方法，在如果传入的Runnable对象不为空，则去执行Runnable对象的run方法

	public class Thread implements Runnable {
	
		private Runnable target;
	
	   	public Thread(Runnable target) {
			init(null, target, "Thread-" + nextThreadNum(), 0);
	    }
	
	    public void run() {
			if (target != null) {
			    target.run();
			}
	    }
	}

但是在这里，**子类run方法实际就是覆盖父类中的run方法**，如果覆盖了就用子类的run方法，不会再找Runnable中的run方法了，所以运行的是子类中的run方法

### 总结

由Thread类中的run方法源代码中看出，两种传统创建线程的方式都是在**调用Thread对象的run方法**

* 如果子类重写父类的run方法，则调用子类的run方法
* 如果Thread对象的run方法没有被覆盖，并且像为Thread对象传递了一个Runnable对象，就会调用Runnable对象的run方法

## 定时器

定时器在游戏上很常用，例如俄罗斯方块下降，贪吃蛇往前移动等

定时器的使用

* 一段时间后执行，第一个参数为任务，第二个参数为等待时间，单位是毫秒


	new Timer().schedule(new TimerTask() {
        @Override
        public void run() {
            System.out.println("boom!");
        }
    }, 1000);

* 一段时间后执行，以后每隔多少时间再次执行，第一个参数为任务，第二个参数为等待多长时间，第三个参数为每隔多长时间执行一次


	new Timer().schedule(new TimerTask() {
	    @Override
	    public void run() {
	        System.out.println("boom!");
	    }
	}, 0, 1000);


显示计时信息：

	//计时
	new Timer().schedule(new TimerTask() {
		
		@Override
		public void run() {
			// TODO Auto-generated method stub
			while(true){
				System.out.println(new Date().getSeconds());
				try {
					Thread.sleep(1000);
				} catch (Exception e) {
					// TODO: handle exception
				}
			}
		}
	}, 0);

## Java内存模型

![](http://i.imgur.com/1rVbVZp.png)

线程之间的**共享变量存储在主内存（main memory）**中，每个线程都有一个**私有的本地内存**（local memory），本地内存中存储了该线程以读/写共享变量的**副本**。

## 传统线程互斥技术

<br/>
<font color='red'>**如果多个线程对同一个资源操作，会引起安全问题（多卖票），使出现的结果和预期的不一样**</font>

**代码实现原子性：有一个线程正在使用这个方法的代码，别的线程就不能再使用**。就和厕所里的坑一样，已经有人在用了，别人就不能再去用了。Java中某段代码要实现**排他性**，就将这段代码用**synchronized**关键字保护起来

	synchronized（this）｛
	    for (int i=0; i<len; i++)
		    System.out.println(name.charAt(i));//逐个字符打印
		System.out.println();
	}

注意：要实现互斥，在这个位置必须使用的**锁必须是同一个对象**，即在同一个对象上进行同步，这样的话，有一个线程进入保护区域后，没出来的话，别的线程就不能进入保护区域

互斥方法：

* a、同步代码块


	synchronized (lock){}
* b、同步方法


	1、方法返回值前加synchronized，同步方法上边用的锁就是this对象
	2、静态同步方法使用的锁是该方法所在的class对象

案例：output1和output2互斥，因为互斥变量都为当前对象

```Java
class Outputer {
	public void output1(String name) {
		synchronized (this) {
			int len = name.length();
			for (int i = 0; i < len; i++)
				System.out.print(name.charAt(i));
			System.out.println();
		}

	}
	public synchronized void output2(String name) {
		int len = name.length();
		for (int i = 0; i < len; i++)
			System.out.print(name.charAt(i));
		System.out.println();

	}
}
```

此时为静态函数，要实现互斥，使用Outputer的字节码

```Java
static class Outputer {
	public static void output1(String name) {
		synchronized (Outputer.class) {
			int len = name.length();
			for (int i = 0; i < len; i++)
				System.out.print(name.charAt(i));
			System.out.println();
		}
	}

	public static synchronized void output2(String name) {
		int len = name.length();
		for (int i = 0; i < len; i++)
			System.out.print(name.charAt(i));
		System.out.println();
	}
}
```

## 锁对象Lock-同步问题更完美的处理方式

### 读写锁(ReadWriteLock)

我们会有一种需求，在对数据进行读写的时候，为了保证数据的一致性和完整性，**需要读和写是互斥的，写和写是互斥的，但是读和读是不需要互斥的**，这样读和读不互斥性能更高些

	class Data {      
	    private int data;// 共享数据  
	    private ReadWriteLock rwl = new ReentrantReadWriteLock();     
	    public void set(int data) {  
	        rwl.writeLock().lock();// 取到写锁  
	        try {  
	            System.out.println(Thread.currentThread().getName() + "准备写入数据");  
	            try {  
	                Thread.sleep(20);  
	            } catch (InterruptedException e) {  
	                e.printStackTrace();  
	            }  
	            this.data = data;  
	            System.out.println(Thread.currentThread().getName() + "写入" + this.data);  
	        } finally {  
	            rwl.writeLock().unlock();// 释放写锁  
	        }  
	    }     
	    public void get() {  
	        rwl.readLock().lock();// 取到读锁  
	        try {  
	            System.out.println(Thread.currentThread().getName() + "准备读取数据");  
	            try {  
	                Thread.sleep(20);  
	            } catch (InterruptedException e) {  
	                e.printStackTrace();  
	            }  
	            System.out.println(Thread.currentThread().getName() + "读取" + this.data);  
	        } finally {  
	            rwl.readLock().unlock();// 释放读锁  
	        }  
	    }  
	}

### Synchronized和Lock的区别

1、用sychronized修饰的方法或者语句块在代码执行完之后**锁自动释放**，而用Lock需要我们**手动释放锁**，所以为了保证锁最终被释放(发生异常情况)，要把互斥区放在try内，释放锁放在finally内。

2、synchronized简单，简单意味着不灵活，而ReentrantLock的锁机制给用户的使用提供了极大的灵活性。这点在Hashtable和ConcurrentHashMap中体现得淋漓尽致。synchronized一锁就锁整个Hash表，而ConcurrentHashMap则利用ReentrantLock实现了锁分离，锁的知识segment而不是整个Hash表

3、synchronized是不公平锁，而ReentrantLock可以指定锁是公平的还是非公平的

4、synchronized实现等待/通知机制通知的线程是随机的，ReentrantLock实现等待/通知机制可以有选择性地通知

5、和synchronized相比，ReentrantLock提供给用户多种方法用于锁信息的获取，比如可以知道lock是否被当前线程获取、lock被同一个线程调用了几次、lock是否被任意线程获取等等

## 传统线程同步通信技术（wait、notify、notifyAll）

### 双重检查

调用Object的wait方法可以让当前线程**进入等待状态，程序不能往下执行**，只有调用了**此Object**的**notify、或notifyAll方法，或者wait到达了指定的时间**后，才会**激活继续执行**

notify只是随机找处于**等待此Object**的**一个线程**；notifyAll是通知wait**此Object**的**所有线程**

为了防止**假唤醒**，提高程序的**可靠性**，wait被唤醒后，需要再次判断等待的状态是否被更改了，如果未更改，则继续进入wait状态，这种做法称作**双重检查**。查看一下代码，即**加上了while进行双重检查**，而且wait操作写在最前面

```
while (current == connections.length) {
    System.out.println(Thread.currentThread().getName() + "在等待");
    this.wait();
}
```

### wait/notify机制原理

当调用了对象的wait方法后，JVM程序**执行引擎**将**此线程放入一个wait集合中**，<font color='red'>**并释放此对象的锁**</font>（本来加上了synchronized只能有一个线程处于等待状态，现在可以多个线程处于等待状态）

* 当其他线程调用此对象的notify方法时，会从wait集合中随机找**一个**等待的此对象上的线程，并将其从wait集合中删除，**JVM线程执行引擎**就可以再次**调度**此线程了

* 当使用 notifyAll 方法时，则会删除 wait 集合中所有等待此对象的线程

### 面试题

线程同步就是线程按照**先后的次序**运行，即"我执行完之后然后你执行"

面试题：子线程循环10次，接着主线程循环100，接着又回到子线程循环10次，接着再回到主线程又循环100，如此循环50次，请写出程序
	
	
	/**
	 * 面试题：子线程循环10次，接着主线程循环100，
	 * 接着又回到子线程循环10次，接着再回到主线程又循环100，
	 * 如此循环50次，请写出程序
	 */
	
	/** 解题思路：线程交替运行，这是一个线程同步问题
	 * 1、首先保证子任务和主任务执行过程中不冲突：将子任务和主任务封装在business类中，使用关键词Synchronized保证代码块的原子性
	 * 2、子任务、主任务前后执行顺序控制：使用wait/notify机制，使用boolean类型的isSub变量控制当前应该执行哪个任务，如果不该执
	 *    行则休眠，否则执行；执行结束后必须反转isSub变量，然后通知阻塞进程继续执行
	 * 3、交替执行50次：子任务和主任务各自循环50次，在第2步时已经保证子任务和主任务的交替运行，完成要求
	 */
	
	class TwoThreadSynchronized {
	    public static void main(String[] args){
	        final Bussiness bussiness = new Bussiness();
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                for(int i = 0; i < 50; i++)
	                    bussiness.main(i+1);
	            }
	        }).start();
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                for(int i = 0; i < 50; i++)
	                    bussiness.sub(i+1);
	            }
	        }).start();
	    }
	}
	
	class Bussiness{
	
	    //共享变量，子任务先开始
	    boolean isSub = true;
	
	    public synchronized void main(int loop){
	        while(isSub){
	            try {
	                wait();
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        }
	
	        System.out.println("第" + loop + "次循环：");
	
	        System.out.print("main：");
	        for(int i = 0; i < 100; i++){
	            System.out.print(i + " ");
	        }
	        System.out.println();
	
	        isSub = true;
	        notify();
	    }
	
	    public synchronized void sub(int loop){
	
	        while(!isSub){
	            try {
	                wait();
	            } catch (InterruptedException e) {
	                e.printStackTrace();
	            }
	        }
	
	        System.out.println("第" + loop + "次循环：");
	
	        System.out.print("sub：");
	        for(int i = 0; i < 10; i++){
	            System.out.print(i + " ");
	        }
	        System.out.println();
	
	        isSub = false;
	        notify();
	    }
	
	}

## 线程间通信(Condition:await、signal)★★★★★

需求：要求线程1和线程2互不干扰，并且交替执行

设计：

* 互不干扰干扰可以让两个线程在同一个对象上同步，通过Synchronized或者Lock对象。
	* 方案一：两个方法都用Synchronized修饰，两个方法互斥
	* 方案二：同一个Lock成员变量，每个方法开始的时候使用lock.lock()，try-finally格式，finally执行lock.unlock();  
* 交替运行可以设置一个标志位，如果为true执行A，如果为false执行B；如果条件标志位不符，线程处于阻塞状态
	* 创建Condition，调用conditon的await、signal方法。当方法不满足条件时，await；满足条件后更改标志位，并且signal唤醒


	class Business {  
	    private boolean bool = true;  
	    private Lock lock = new ReentrantLock();  
	    private Condition condition = lock.newCondition();   
	    public /*synchronized*/ void main(int loop) throws InterruptedException {  
	        lock.lock();  
	        try {  
	            while(bool) {                 
	                condition.await();//this.wait();  
	            }  
	            for(int i = 0; i < 100; i++) {  
	                System.out.println("main thread seq of " + i + ", loop of " + loop);  
	            }  
	            bool = true;  
	            condition.signal();//this.notify();  
	        } finally {  
	            lock.unlock();  
	        }  
	    }     
	    public /*synchronized*/ void sub(int loop) throws InterruptedException {  
	        lock.lock();  
	        try {  
	            while(!bool) {  
	                condition.await();//this.wait();  
	            }  
	            for(int i = 0; i < 10; i++) {  
	                System.out.println("sub thread seq of " + i + ", loop of " + loop);  
	            }  
	            bool = false;  
	            condition.signal();//this.notify();  
	        } finally {  
	            lock.unlock();  
	        }  
	    }  
	}  


在Condition中，用await()替换wait()，用signal()替换notify()，用signalAll()替换notifyAll()，传统线程的通信方式，Condition都可以实现，这里注意，Condition是被绑定到Lock上的，要创建一个Lock的Condition必须用newCondition()方法。

### 生产者消费者模式

设计+互斥操作：

![](http://i.imgur.com/D0lMvMW.jpg)

线程间通信：

![](http://i.imgur.com/kM5NQqK.jpg)

这是一个处于多线程工作环境下的缓存区，缓存区提供了两个方法，put和take，put是存数据，take是取数据，内部有个缓存队列，这个缓存区类实现的功能：有多个线程往里面存数据和从里面取数据，其缓存队列(先进先出后进后出)能缓存的最大数值是100，**多个线程间是互斥的**，**当缓存队列中存储的值达到100时，将写线程阻塞，并唤醒读线程，当缓存队列中存储的值为0时，将读线程阻塞，并唤醒写线程，这也是ArrayBlockingQueue的内部实现**

<br/>
<font color='red'>**多个Condition的好处：**</font>

**假设缓存队列中已经存满，那么阻塞的肯定是写线程，唤醒的肯定是读线程，相反，阻塞的肯定是读线程，唤醒的肯定是写线程**。那么假设只有一个Condition会有什么效果呢，缓存队列中已经存满，**这个Lock不知道唤醒的是读线程还是写线程了**，如果唤醒的是读线程，皆大欢喜，如果唤醒的是写线程，那么线程刚被唤醒，又被阻塞了，这时又去唤醒，这样就浪费了很多时间。

代码：

	class BoundedBuffer{
		final Locklock = newReentrantLock();//锁对象
		final Condition notFull = lock.newCondition();//写线程条件
		final Condition notEmpty = lock.newCondition();//读线程条件
	
		final Object[] items = new Object[100];//缓存队列
		intputptr/*写索引*/,takeptr/*读索引*/,count/*队列中存在的数据个数*/;
	
		public void put(Object x) throws InterruptedException{
			lock.lock();
			try{
				while(count == items.length)//如果队列满了
					notFull.await();//阻塞写线程
				items[putptr] = x;//赋值
				if(++putptr == items.length) putptr = 0;//如果写索引写到队列的最后一个位置了，那么置为0
				++count;//个数++
				notEmpty.signal();//唤醒读线程
			}finally{
				lock.unlock();
			}
		}
	
		public Object take() throws InterruptedException{
			lock.lock();
			try{
				while(count == 0)//如果队列为空
					notEmpty.await();//阻塞读线程
				Objectx = items[takeptr];//取值
				if(++takeptr == items.length) takeptr = 0;//如果读索引读到队列的最后一个位置了，那么置为0
				--count;//个数--
				notFull.signal();//唤醒写线程
				return x;
			}finally{
				lock.unlock();
			}
		}
	}

## ThreadLocal实现线程范围内的数据共享

什么是线程范围内的数据共享？<font color='red'>**一个线程内部的各个函数都能访问该共享变量，而其他线程不能访问**</font>（区别于静态变量，静态变量能被所有线程访问，但需要解决线程间数据同步的问题）

应用：

* 用户访问Web服务器，服务器启动一条线程处理用户请求，不同用户之间不受干扰
* 用户访问数据库，每个用户使用独立的connection。不然会出现A用户开启事务，操作还没有结束，B用户使用的是相同的connection，提交事务，导致A用户还未完成操作但事务提交了

### 自己的实现的线程范围内的数据共享

使用全局的map，键为线程，值为该线程中需要共享的变量。每次取值时，传入当前的线程，即可取到当前线程共享的变量

	public class ThreadLocalTest {
		//键为当前线程，值为线程范围内的共享变量
	    static Map<Thread, Integer> map = new HashMap<Thread, Integer>();
	
	    public static void main(String[] args){
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                map.put(Thread.currentThread(),1);
	                System.out.println("A:" + new A().get());
	            }
	        }).start();
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                map.put(Thread.currentThread(),2);
	                System.out.println("B:" + new A().get());
	            }
	        }).start();
	
	
	    }
	
	    static class A{
	        public Integer get(){
	            return map.get(Thread.currentThread());
	        }
	    }
	}

### 使用 ThreadLocal 实现的线程间数据共享

	public class ThreadLocalTest {
	
	    static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>();
	
	    public static void main(String[] args){
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                threadLocal.set(1);
	                System.out.println("A:" + new A().get());
	            }
	        }).start();
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                threadLocal.set(2);
	                System.out.println("B:" + new A().get());
	            }
	        }).start();
	
	    }
	
	    static class A{
	        public Integer get(){
	            return threadLocal.get();
	        }
	    }
	}

原理：

1、1个ThreadLocal代表一个与线程绑定的值
2、线程中存储多个ThreadLocal的值，用Map存储

	public class ThreadLocal<T> {
	
		public void set(T value) {
	        Thread t = Thread.currentThread();
	        ThreadLocalMap map = getMap(t);
	        if (map != null)
	            map.set(this, value);
	        else
	            createMap(t, value);
	    }
	
		public T get() {
	        Thread t = Thread.currentThread();
	        ThreadLocalMap map = getMap(t);
	        if (map != null) {
	            ThreadLocalMap.Entry e = map.getEntry(this);
	            if (e != null)
	                return (T)e.value;
	        }
	        return setInitialValue();
	    }
	
		ThreadLocalMap getMap(Thread t) {
	        return t.threadLocals;
	    }

	}

### 使用 ThreadLocal 实现线程范围内的单例

* 每个线程都使用一个类的实例，实现这个功能需要结合ThreadLocal，与线程绑定
* 单例模式无法实现，是在整个应用中保持单例

应用：对于每一个用户请求，对应一个线程，为之创建一个容器，仅供该用户使用。该容器可以使用线程范围内的单例来实现

<pre>
public class ThreadLocalTest {

    public static void main(String[] args){
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread() + " " + Person.getThreadLocalInstance());
                System.out.println(Thread.currentThread() + " " + Person.getThreadLocalInstance());
            }
        }).start();

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread() + " " + Person.getThreadLocalInstance());
                System.out.println(Thread.currentThread() + " " + Person.getThreadLocalInstance());
            }
        }).start();
    }
}

class Person{

    <font color='red'>private static ThreadLocal<Person> threadLocal = new ThreadLocal<Person>();
    public static Person getThreadLocalInstance(){
        Person person = threadLocal.get();
        if(person == null){
            person = new Person();
            threadLocal.set(person);
        }
        return person;
    }</font>

    private String name;
    private Integer age;

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

}
</pre>

### ThreadLocal 设计

![](http://i.imgur.com/ktTa3m6.png)

### ThreadLocal原理分析

http://blog.csdn.net/huachao1001/article/details/51970237

ThreadLocal里面的实现，主要涉及到以下几个重要类：

* Thread：大家很熟悉的线程类，一个Thread类自然代表一个线程
* ThreadLocal：既然本文是要解析ThreadLocal类，自然就离不开这个类啦
* ThreadLocalMap：可以看成一个HashMap，但是它本身具体的实现并没有实现继承HashMap甚至跟java.util.Map都沾不上一点关系。只是内部的实现跟HashMap类似（通过哈希表的方式存储）
* ThreadLocalMap.Entry：把它看成是保存键值对的对象，其本质上是一个WeakReference<ThreadLocal>对象

ThreadLocal的存取机制可以用下图表示：

![](http://i.imgur.com/lOd5Awk.png)

1、**Thread中有一个ThreadLocalMap属性，存放线程局部变量**，是键值对的存储方式，以ThreadLocal对象为键，要保存的变量为值（ThreadLocalMap.Entry是保存键值对的对象）

2、<font color='red'>**ThreadLocal可以看做是ThreadLocalMap的操作类**</font>，在当前线程中通过ThreadLocal的set/get操作，对当前线程中的ThreadLocalMap进行操作

* set方法：首先获取当前线程的ThreadLocalMap的引用，然后以ThreadLocal实例作为key，保存的变量为value，保存到当前线程的ThreadLocalMap中
* get方法：以当前ThreadLocal实例作为key，取出当前线程的ThreadLocalMap变量中该key的值

3、**每个线程都拥有自己的ThreadLocalMap属性，每个线程都是从自己的ThreadLocalMap中取值，所以互不影响**

4、作用：可以实现线程范围内的单例、线程范围内数据共享

    private static ThreadLocal<Person> threadLocal = new ThreadLocal<Person>();
    public static Person getThreadLocalInstance(){
        Person person = threadLocal.get();
        if(person == null){
            person = new Person();
            threadLocal.set(person);
        }
        return person;
    }

## 多个线程之间共享数据的方式探讨

1、两个线程的操作一致，可以使用同一个Runnable对象。火车站买票适用这种情况

	public class Method1 {
	
	    public static void main(String[] args){
	
	        Runnable runnable = new sell();
	
	        new Thread(runnable).start();
	        new Thread(runnable).start();
	    }
	}
	
	class sell implements Runnable{
	
	    private int ticket = 1000000;
	
	    public synchronized void sell(){
	        if(ticket > 0){
	            ticket--;
	            System.out.println(Thread.currentThread() + " 剩余票" + ticket);
	        }
	    }
	
	    @Override
	    public void run() {
	        while (true){
	            sell();
	        }
	    }
	}

2、两个线程操作不一样。此时必须设置第三方共享数据，并调用他的方法进行数据操作（推荐）

	public class Method1 {
	
	    public static void main(String[] args){
	
	        final TrainCenter trainCenter = new TrainCenter();
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                while(true)
	                   trainCenter.sell();
	            }
	        }).start();
	
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                while(true)
	                    trainCenter.sell();
	            }
	        }).start();
	    }
	}
	
	
	
	class TrainCenter{
	    private int ticket = 100000;
	
	    public int getTicket() {
	        return ticket;
	    }
	
	    public void setTicket(int ticket) {
	        this.ticket = ticket;
	    }
	
	    public synchronized void sell(){
	        if(ticket > 0){
	            ticket--;
	            System.out.println(Thread.currentThread() + " 剩余 " + ticket);
	        }
	    }
	}

3、共享数据传入Runnable对象中，进行操作

	public class Method1 {
	
	    public static void main(String[] args){
	
	        TrainCenter trainCenter = new TrainCenter();
	
	        new Thread(new Train1(trainCenter)).start();
	        new Thread(new Train2(trainCenter)).start();
	
	    }
	}
	
	
	class Train1 implements Runnable{
	
	    private TrainCenter trainCenter;
	
	    public Train1(TrainCenter trainCenter){
	        this.trainCenter = trainCenter;
	    }
	
	    @Override
	    public void run() {
	        while(true){
	            trainCenter.sell();
	        }
	    }
	}
	
	class Train2 implements Runnable{
	
	    private TrainCenter trainCenter;
	
	    public Train2(TrainCenter trainCenter){
	        this.trainCenter = trainCenter;
	    }
	
	    @Override
	    public void run() {
	        while(true){
	            trainCenter.sell();
	        }
	    }
	}
	
	
	
	class TrainCenter{
	    private int ticket = 100000;
	
	    public int getTicket() {
	        return ticket;
	    }
	
	    public void setTicket(int ticket) {
	        this.ticket = ticket;
	    }
	
	    public synchronized void sell(){
	        if(ticket > 0){
	            ticket--;
	            System.out.println(Thread.currentThread() + " 剩余 " + ticket);
	        }
	    }
	}

## 线程池

### 线程池的作用

* 根据系统自身的环境情况，**有效的限制执行线程的数量**，超出数量的任务排队等候，等待有任务执行完毕，再从队列最前面取出任务执行
* 减少创建和销毁线程的次数，每个工作线程可以多次使用

### 常见线程池

1、newSingleThreadExecutor

单个线程的线程池，即线程池中每次只有一个线程工作，单线程串行执行任务

2、newFixedThreadExecutor(n)

固定数量的线程池，没提交一个任务就是一个线程，直到达到线程池的最大数量，然后后面**进入等待队列**直到前面的任务完成才继续执行

**3、newCacheThreadExecutor（推荐使用）**

可缓存线程池，当线程池大小超过了处理任务所需的线程，那么就会**回收部分空闲**（一般是60秒无执行）的线程，当有任务来时，又**智能的添加**新线程来执行。

4、newScheduleThreadExecutor

大小无限制的线程池，支持定时和周期性的执行线程

实例：

1、newSingleThreadExecutor

	public class TestSingleThreadExecutor {
	    public static void main(String[] args){
	        //创建一个单个线程的线程池
	        ExecutorService pool = Executors.newSingleThreadExecutor();
	        //将任务放入池中并执行
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        //关闭
	        pool.shutdown();
	    }
	}
	
	class Task implements Runnable{
	
	    @Override
	    public void run() {
	        System.out.println(Thread.currentThread().getName() + "执行中。。。");
	    }
	}

result：

	pool-1-thread-1执行中。。。
	pool-1-thread-1执行中。。。
	pool-1-thread-1执行中。。。
	pool-1-thread-1执行中。。。
	pool-1-thread-1执行中。。。


2、newFixedThreadExecutor(n)

	public class TestFixedThreadPool {
	    public static void main(String[] args){
	        //创建一个固定线程数的线程池
	        ExecutorService pool = Executors.newFixedThreadPool(2);
	        //将任务放入池中并执行
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        //关闭
	        pool.shutdown();
	    }
	}

result：

	pool-1-thread-1执行中。。。
	pool-1-thread-2执行中。。。
	pool-1-thread-1执行中。。。
	pool-1-thread-2执行中。。。
	pool-1-thread-1执行中。。。


3、newCacheThreadExecutor

	public class TestCachedThreadPool {
	    public static void main(String[] args) {
	        //创建一个可重用固定线程数的线程池
	        ExecutorService pool = Executors.newCachedThreadPool();
	        //将任务放入池中并执行
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        pool.execute(new Task());
	        //关闭
	        pool.shutdown();
	    }
	}

result：

	pool-1-thread-1执行中。。。
	pool-1-thread-2执行中。。。
	pool-1-thread-4执行中。。。
	pool-1-thread-3执行中。。。
	pool-1-thread-5执行中。。。

### JAVA线程池shutdown和shutdownNow的区别

shutDown() 

当线程池调用该方法时，线程池的状态则立刻变成SHUTDOWN状态。此时，则**不能再往线程池中添加任何任务**，否则将会抛出RejectedExecutionException异常。但是，此时线程池不会立刻退出，直到添加到线程池中的任务都**已经处理完成，才会退出**。 

shutdownNow() 

根据JDK文档描述，大致意思是：执行该方法，**线程池的状态立刻变成STOP状态，并试图停止所有正在执行的线程，不再处理还在池队列中等待的任务**，当然，它会返回那些未执行的任务。 

它试图终止线程的方法是通过调用Thread.interrupt()方法来实现的，但是大家知道，这种方法的作用有限，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要**等待所有正在执行的任务都执行完成了才能退出**。 

## 线程池定时器

这种做法从性能方面讲相对于传统的定时器要好

	public static void main(String[] args){
        Executors.newScheduledThreadPool(1).scheduleAtFixedRate(new Runnable() {
            @Override
            public void run() {
                System.out.println("测试开始");
            }
        },0,1, TimeUnit.SECONDS);
    }

## Callable和Future

Callable和Future，一个产生结果，一个拿到结果。 Callable接口类似于Runnable，从名字就可以看出来了，但是Runnable不会返回结果，并且无法抛出返回结果的异常，而Callable功能更强大一些，**被线程执行后，可以返回值，这个返回值可以被Future拿到**，也就是说，**Future可以拿到异步执行任务的返回值**，下面来看一个简单的例子：

	public class CallableAndFuture {
	    private FutureTask<Integer> future;
	
	    public static void main(String[] args) {
	
	        //复杂操作，需要耗费很长时间，但结果不是立刻需要
	        Callable<Integer> callable = new Callable<Integer>() {
	            public Integer call() throws Exception {
	                Thread.sleep(10000);
	                return new Random().nextInt(100);
	            }
	        };
	
	        FutureTask<Integer> future = new FutureTask<Integer>(callable);
	        new Thread(future).start();
	
	        try {
	            Thread.sleep(5000);// 可能做一些事情
	
	            if(!future.isDone()){
	                System.out.println("任务还没有执行完，先返回默认值0");
	            }
	
	            //用户有足够的耐心等待任务执行结束
	            System.out.println("任务执行结束：" + future.get());
	
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        } catch (ExecutionException e) {
	            e.printStackTrace();
	        }
	    }
	}

FutureTask实现了两个接口，Runnable和Future，所以它**既可以作为Runnable被线程执行，又可以作为Future得到Callable的返回值**，那么这个组合的使用有什么好处呢？假设有一个很耗时的返回值需要计算，并且这个返回值不是立刻需要的话，那么就可以使用这个组合，**用另一个线程去计算返回值，而当前线程在使用这个返回值之前可以做其它的操作，等到需要这个返回值时，再通过Future得到**，岂不美哉！这里有一个Future模式的介绍：http://openhome.cc/Gossip/DesignPattern/FuturePattern.htm。 

## 并发队列 ConcurrentLinkedQueue 和阻塞队列 LinkedBlockingQueue 用法

在Java多线程应用中，队列的使用率很高，多数生产消费模型的首选数据结构就是队列(先进先出)。Java提供的线程安全的Queue可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是BlockingQueue，非阻塞队列的典型例子是ConcurrentLinkedQueue，在实际应用中要根据实际需要选用阻塞队列或者非阻塞队列。

### LinkedBlockingQueue

由于LinkedBlockingQueue实现是线程安全的，实现了先进先出等特性，是作为生产者消费者的首选，LinkedBlockingQueue 可以指定容量，也可以不指定，不指定的话，默认最大是Integer.MAX_VALUE，其中主要用到put和take方法，put方法在队列满的时候会阻塞直到有队列成员被消费，take方法在队列空的时候会阻塞，直到有队列成员被放进来。

	package cn.thread;
	
	import java.util.concurrent.BlockingQueue;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	import java.util.concurrent.LinkedBlockingQueue;
	
	/**
	 * 多线程模拟实现生产者／消费者模型
	 */
	public class BlockingQueueTest2 {
	    /**
	     * 
	     * 定义装苹果的篮子
	     * 
	     */
	    public class Basket {
	        // 篮子，能够容纳3个苹果
	        BlockingQueue<String> basket = new LinkedBlockingQueue<String>(3);
	
	        // 生产苹果，放入篮子
	        public void produce() throws InterruptedException {
	            // put方法放入一个苹果，若basket满了，等到basket有位置
	            basket.put("An apple");
	        }
	
	        // 消费苹果，从篮子中取走
	        public String consume() throws InterruptedException {
	            // take方法取出一个苹果，若basket为空，等到basket有苹果为止(获取并移除此队列的头部)
	            return basket.take();
	        }
	    }
	
	    // 定义苹果生产者
	    class Producer implements Runnable {
	        private String instance;
	        private Basket basket;
	
	        public Producer(String instance, Basket basket) {
	            this.instance = instance;
	            this.basket = basket;
	        }
	
	        public void run() {
	            try {
	                while (true) {
	                    // 生产苹果
	                    System.out.println("生产者准备生产苹果：" + instance);
	                    basket.produce();
	                    System.out.println("!生产者生产苹果完毕：" + instance);
	                    // 休眠300ms
	                    Thread.sleep(300);
	                }
	            } catch (InterruptedException ex) {
	                System.out.println("Producer Interrupted");
	            }
	        }
	    }
	
	    // 定义苹果消费者
	    class Consumer implements Runnable {
	        private String instance;
	        private Basket basket;
	
	        public Consumer(String instance, Basket basket) {
	            this.instance = instance;
	            this.basket = basket;
	        }
	
	        public void run() {
	            try {
	                while (true) {
	                    // 消费苹果
	                    System.out.println("消费者准备消费苹果：" + instance);
	                    System.out.println(basket.consume());
	                    System.out.println("!消费者消费苹果完毕：" + instance);
	                    // 休眠1000ms
	                    Thread.sleep(1000);
	                }
	            } catch (InterruptedException ex) {
	                System.out.println("Consumer Interrupted");
	            }
	        }
	    }
	
	    public static void main(String[] args) {
	        BlockingQueueTest2 test = new BlockingQueueTest2();
	
	        // 建立一个装苹果的篮子
	        Basket basket = test.new Basket();
	
	        ExecutorService service = Executors.newCachedThreadPool();
	        Producer producer = test.new Producer("生产者001", basket);
	        Producer producer2 = test.new Producer("生产者002", basket);
	        Consumer consumer = test.new Consumer("消费者001", basket);
	        service.submit(producer);
	        service.submit(producer2);
	        service.submit(consumer);
	        // 程序运行5s后，所有任务停止
	//        try {
	//            Thread.sleep(1000 * 5);
	//        } catch (InterruptedException e) {
	//            e.printStackTrace();
	//        }
	//        service.shutdownNow();
	    }
	
	}

原理：
1、双向链表，队列中维持头指针和为指针，元素个数的类型是 AtomicInteger
2、put的时候使用重入锁，可以被中断；

### ConcurrentLinkedQueue

ConcurrentLinkedQueue是Queue的一个安全实现．Queue中元素按FIFO原则进行排序，采用**CAS**操作，来保证元素的一致性。

原理：
1、**tail节点不一定指向最后一个元素，也有可能是倒数第二个节点**。

![](http://i.imgur.com/lZcMbIg.jpg)

为什么这么设计？

* **在单线程中，插入节点后，将新插入的节点设置为tail**。但是在多线程的情况下，假设有多个插入节点操作A，B，可能会先将B插入的节点设置为tail，然后再将A插入的节点设置为tail，导致tail不是严格的指向尾部指针(如下图)。**所以设置tail指针也需要CAS操作**

![](http://i.imgur.com/LjiZ5eX.jpg)

* <font color='red'>**插入节点操作必须使用CAS操作，如果每次移动tail指针也要采用CAS，移动tail指针操作也会影响插入节点的CAS操作（因为在插入节点的CAS操作时，要保证tail为最后一个节点），那样会特别影响性能，导致操作容易失败，所以采用延时移动tail指针**</font>
* 如果tail指针离最后一个节点的距离很大，**每次要遍历到最后一个节点**，特别耗时，所以**当tail指针离最后一个节点的距离超过2，让tail节点指向最后一个节点**


2、如何插入节点★★★★★★

* 如果tail为最后一个节点，采用CAS操作加入新节点（预期p的next节点为null，如果是则将p的next设置为新节点；否则重试）
* 如果tail不是最后一个节点，p后移一位指向最后一个元素，**继续循环**
* 如果tail指针与新插入的指针距离相差2（代码中由p！=t体现），则CAS更新tail指针

**<font color='red'>总结：</font>**
1、插入节点必须采用CAS操作，更新tail也必须采用CAS操作，影响性能，所以延迟tail节点更新
2、tail指针距离最后一个节点太远，需要遍历。所以当tail指针距离最后一个节点大于等于2时，CAS移动tail指针为最后一个节点
3、插入节点时，如果tail指针指向最后一个节点，则CAS增加节点；如果不是，则移动p为下一个节点，然后循环增加节点；如果距离超过2，才CAS更新tail指针

源码：

<pre>
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    for (Node<E> t = tail, p = t;;) {
        Node<E> q = p.next;
        <font color='red'>if (q == null) {//最后一个节点</font>
            // p is last node
            if (p.casNext(null, newNode)) {
                // Successful CAS is the linearization point
                // for e to become an element of this queue,
                // and for newNode to become "live".
                if (p != t) // hop two nodes at a time
                    casTail(t, newNode);  // Failure is OK.
                return true;
            }
            // Lost CAS race to another thread; re-read next
        }
        else if (p == q)
            // We have fallen off list.  If tail is unchanged, it
            // will also be off-list, in which case we need to
            // jump to head, from which all live nodes are always
            // reachable.  Else the new tail is a better bet.
            p = (t != (t = tail)) ? t : head;
        else
            // Check for tail updates after two hops.
            <font color='red'>p = (p != t && t != (t = tail)) ? t : q;//p不是最后一个节点，后移一位</font>
    }
}
</pre>

出队：

![](http://i.imgur.com/oTjvfpw.jpg)

从上图可知，并不是每次出队时都更新head节点，当head节点里有元素时，直接弹出head节点里的元素，而不会更新head节点。只有当head节点里没有元素时，出队操作才会更新head节点。这种做法也是通过hops变量来减少使用CAS更新head节点的消耗，从而提高出队效率。让我们再通过源码来深入分析下出队过程。

队列判空：

有些人在判断队列是否为空时喜欢用queue.size()==0，让我们来看看size方法：

	public int size() 
	{
		int count = 0;
		for (Node<E> p = first(); p != null; p = succ(p))
			if (p.item != null)
				// Collection.size() spec says to max out
				if (++count == Integer.MAX_VALUE)
					break;
		return count;
	}

可以看到这样在队列在结点较多时会依次遍历所有结点，这样的性能会有较大影响，因而可以考虑empty函数，它只要判断第一个结点(注意不一定是head指向的结点)。

	public boolean isEmpty() {
		return first() == null;
	}

应用：

	import java.util.concurrent.ConcurrentLinkedQueue;
	import java.util.concurrent.CountDownLatch;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.Executors;
	
	public class ConcurrentLinkedQueueTest {
	    private static ConcurrentLinkedQueue<Integer> queue = new ConcurrentLinkedQueue<Integer>();
	    private static int count = 2; // 线程个数
	    //CountDownLatch，一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
	    private static CountDownLatch latch = new CountDownLatch(count);
	
	    public static void main(String[] args) throws InterruptedException {
	        long timeStart = System.currentTimeMillis();
	        ExecutorService es = Executors.newFixedThreadPool(4);
	        ConcurrentLinkedQueueTest.offer();
	        for (int i = 0; i < count; i++) {
	            es.submit(new Poll());
	        }
	        latch.await(); //使得主线程(main)阻塞直到latch.countDown()为零才继续执行
	        System.out.println("cost time " + (System.currentTimeMillis() - timeStart) + "ms");
	        es.shutdown();
	    }
	    
	    /**
	     * 生产
	     */
	    public static void offer() {
	        for (int i = 0; i < 100000; i++) {
	            queue.offer(i);
	        }
	    }
	
	
	    /**
	     * 消费
	     * @author 林计钦
	     * @version 1.0 2013-7-25 下午05:32:56
	     */
	    static class Poll implements Runnable {
	        public void run() {
	            // while (queue.size()>0) {
	            while (!queue.isEmpty()) {
	                System.out.println(queue.poll());
	            }
	            latch.countDown();
	        }
	    }
	}

### 参考

Java线程(九)：Condition-线程通信更高效的方式
http://blog.csdn.net/ghsau/article/details/7481142

Java并发编程（七）ConcurrentLinkedQueue的实现原理和源码分析
http://blog.csdn.net/itachi85/article/details/52205256

聊聊并发（六）——ConcurrentLinkedQueue的实现原理分析
http://www.infoq.com/cn/articles/ConcurrentLinkedQueue/

无锁队列的实现
http://coolshell.cn/articles/8239.html

## CAS操作

1、读取当前内存值，作为预估值
2、当执行CAS指令时，如果内存当前值等于预估值，则更新；否则不执行，重试

示例：

1、读取 ——> 2、执行+1操作，指令1和指令2不是原子操作，所以在指令1的时候有个**预估值**，在**执行**的时候，**判断当前值和预估值是否一样，如果一样说明之前没有别的线程对该变量操作**，可以更改当前值

代码如下：

	if (this == expect) {
		this = update
		return true;
	} else {
		return false;
	}

### 参考

JAVA CAS原理深度分析
http://blog.csdn.net/hsuxu/article/details/9467651

## 线程中断、线程让步、线程睡眠、线程合并

### 线程中断 interrupt()

interrupt()方法并不是中断线程的执行，而是为调用该方法的线程对象**打上一个标记，设置其中断状态为true，通过isInterrupted()方法可以得到这个线程状态**

<pre>
public class InterruptTest {  
    public static void main(String[] args) throws InterruptedException {  
        MyThread t = new MyThread("MyThread");  
        t.start();  
        Thread.sleep(100);// 睡眠100毫秒  
        <font color='red'>t.interrupt();//中断t线程</font>
    }  
}  
class MyThread extends Thread {  
    int i = 0;  
    public MyThread(String name) {  
        super(name);  
    }  
    public void run() {  
        <font color = 'red'>while(!isInterrupted()) {// 当前线程没有被中断，则执行  </font>
            System.out.println(getName() + getId() + "执行了" + ++i + "次");  
        }  
    }  
}  
</pre>

这样的话，线程被顺利的中断执行了。很多人实现一个线程类时，都会再加一个 flag 标记，以便控制线程停止执行，其实完全没必要，**通过线程自身的中断状态，就可以完美实现该功能**

Thread.interrupted()方法是一个静态方法，它是判断当前线程的中断状态，需要注意的是，**线程的中断状态会由该方法清除**。换句话说，**如果连续两次调用该方法，则第二次调用将返回 false**（在第一次调用已清除了其中断状态之后，且第二次调用检验完中断状态前，当前线程再次中断的情况除外）。

### 线程让步 yield()

程让步用于正在执行的线程，在某些情况下让出CPU资源，让给其它线程执行
注意，如果存在synchronized线程同步的话，线程让步不会释放锁(监视器对象)。

### 线程睡眠 sleep

线程睡眠的过程中，如果是在synchronized线程同步内，是持有锁(监视器对象)的，也就是说，线程是关门睡觉的，别的线程进不来

### 线程合并 join

线程合并是优先执行调用该方法的线程，再执行当前线程，来看一个小例子：

	public class JoinTest {  
	    public static void main(String[] args) throws InterruptedException {  
	        JoinThread t1 = new JoinThread("t1");  
	        JoinThread t2 = new JoinThread("t2");  
	        t1.start();  
	        t2.start();  
	        t1.join();  
	        t2.join();  
	        System.out.println("主线程开始执行！");  
	    }  
	}  
	class JoinThread extends Thread {  
	    public JoinThread(String name) {  
	        super(name);  
	    }  
	    public void run() {  
	        for(int i = 1; i <= 10; i++)  
	            System.out.println(getName() + getId() + "执行了" + i + "次");  
	    }  
	} 

t1和t2都执行完才继续主线程的执行，所谓合并，就是等待其它线程执行完，再执行当前线程，执行起来的效果就好像把其它线程合并到当前线程执行一样。

### 参考

http://blog.csdn.net/ghsau/article/details/17560467

## 并发编程的设计模式

### Future 模式

考虑这样一个情况，使用者可能快速翻页浏览文件中，而图片档案很大，如此在浏览到有图片的页数时，就会导致图片的载入，因而造成使用者浏览文件时会有停顿的现象，所以**我们希望在文件开启之后，会有一个默认背景**（可以是纯色图片，也可以是正在加载的图片），**与此同时，程序开始加载图片的工作，如此使用者在快速浏览页面时，所造成的停顿可以获得改善。**

Future模式在请求发生时，会先产生一个Future物件给发出请求的客户，而同时间，真正的目标物件之生成，由一个新的执行持续进行（即 Worker Thread），真正的目标物件生成之后，将之设定至Future之中，而当客户端真正需要目标物件时，目标物件也已经准备好，可以让客户提取使用。 

![](http://i.imgur.com/487kq32.jpg)

一个简单的Java程式片段示范可能像是这样：

	public Future request() {
		final Future future = new Future();
	
		new Thread() {
		    public void run() {
		        // 下面這個動作可能是耗時的
		        RealSubject subject = new RealSubject();
		        future.setRealSubject(subject);
		    }
		}.start();
	
		return future;
	} 


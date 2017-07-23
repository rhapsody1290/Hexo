---
title: JVM性能调优

date: 2017-04-01 18:20:00

categories:
- JVM

tags:
- 性能调优

---

## jps

列出正在运行的虚拟机进程

使用案例：

	jps -l

语法：

jps [-option] [hostid]

	q	只输出LVMID，省略主类的名称
	m	输出main method的参数
	l	输出完全的包名，应用主类名，jar的完全路径名
	v	输出jvm参数

## jstat

监视虚拟机运行状态信息

监控堆使用率，每秒显示一次

	jstat -gcutil 25444 1000 

监控堆容量的变化，每秒显示一次

	jstat -gc 25444 1000

语法结构：

	Usage: 
	jstat -help|-options
	jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]

参数解释：

C:capacity U：usage

	Options — 选项，我们一般使用** -gcutil** 查看gc 情况
	vmid    — VM 的进程号，即当前运行的java 进程号
	interval– 间隔时间，单位为秒或者毫秒
	count   — 打印次数，如果缺省则打印无数次
	 
	S0  — Heap 上的 Survivor space 0 区已使用空间的百分比 
	S1  — Heap 上的 Survivor space 1 区已使用空间的百分比 
	E   — Heap 上的 Eden space 区已使用空间的百分比 
	O   — Heap 上的 Old space 区已使用空间的百分比 
	P   — Perm space 区已使用空间的百分比 
	YGC — 从应用程序启动到采样时发生 Young GC 的次数 
	YGCT– 从应用程序启动到采样时 Young GC 所用的时间( 单位秒 )
	FGC — 从应用程序启动到采样时发生 Full GC 的次数 
	FGCT– 从应用程序启动到采样时 Full GC 所用的时间( 单位秒 )
	GCT — 从应用程序启动到采样时用于垃圾回收的总时间( 单位秒)

	S0C：年轻代中第一个survivor（幸存区）的容量 (字节) 
	S1C：年轻代中第二个survivor（幸存区）的容量 (字节) 
	S0U：年轻代中第一个survivor（幸存区）目前已使用空间 (字节) 
	S1U：年轻代中第二个survivor（幸存区）目前已使用空间 (字节) 
	EC：年轻代中Eden（伊甸园）的容量 (字节) 
	EU：年轻代中Eden（伊甸园）目前已使用空间 (字节) 
	OC：Old代的容量 (字节) 
	OU：Old代目前已使用空间 (字节) 
	PC：Perm(持久代)的容量 (字节) 
	PU：Perm(持久代)目前已使用空间 (字节) 

jstat的option参数：

	-class：统计class loader行为信息
	-compile：统计编译行为信息
	-gc：统计jdk gc时heap信息
	-gccapacity：统计不同的generations（不知道怎么翻译好，包括新生区，老年区，permanent区）相应的heap容量情况
	-gccause：统计gc的情况，（同-gcutil）和引起gc的事件
	-gcnew：统计gc时，新生代的情况
	-gcnewcapacity：统计gc时，新生代heap容量
	-gcold：统计gc时，老年区的情况
	-gcoldcapacity：统计gc时，老年区heap容量
	-gcpermcapacity：统计gc时，permanent区heap容量
	-gcutil：统计gc时，heap情况
	-printcompilation：不知道干什么的，一直没用过。

## jmap

jmap（Memory Map for Java）命令用于生成堆转储快照（一般称为heapdump或dump文件）。如果不使用jmap命令，想要获取Java堆转储快照还有一些比较"暴力"的手段：

* 可以使用` -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=C:\ `参数，可以让虚拟机在OOM异常出现之后自动生成dump文件
* 通过-XX:+HeapDumpOnCtrlBreak参数则可以使用[Ctrl]+[Break]键让虚拟机生成dump文件
* 在Linux系统下通过Kill -3命令发送进程退出信号"恐吓"一下虚拟机，也能拿到dump文件。

使用案例，详细参数见案例说明：

	jmap -heap pid #查看进程堆内存使用情况，包括使用的GC算法、堆配置参数和各代中堆内存使用情况
    jmap -histo[:live] pid #查看堆内存中的对象数目、大小统计直方图，如果带上live则只统计活对象
	jmap -dump:format=b,file=eclipse.bin 7240 #生成Java程序dump快照文件

语法：

	jmap [ -option ] <pid>

option参数：

	dump	生成堆存储快照，格式为：-dump:[live, ]format=b, file=<filename>，live说明是否只dump出存活的对象。
	heap	显示java堆详细信息，如使用那种回收器、参数配置、分代状况等。
	histo	显示堆中对象统计信息，包括类、实例数量、合计容量。

案例说明：

	C:\Users\Asus>jmap -heap 10416 | more
	Attaching to process ID 10416, please wait...
	Debugger attached successfully.
	Server compiler detected.
	JVM version is 20.45-b01
	
	using thread-local object allocation. #使用本地线程分配缓存
	Parallel GC with 4 thread(s) # 并行垃圾回收，4个线程
	
	Heap Configuration: #堆设置
	   MinHeapFreeRatio = 40 #GC后，如果发现空闲堆内存占到整个预估堆内存的40%，则放大堆内存的预估最大值，但不超过固定最大值。
	   MaxHeapFreeRatio = 70 #GC后，如果发现空闲堆内存占到整个预估堆内存的70%，则收缩堆内存预估最大值。什么是预估堆内存？1、预估堆内存是堆大小动态调控的重要选项之一 2、堆内存预估最大值一定小于或等于固定最大值(-Xmx指定的数值) 3、前者会根据使用情况动态调大或缩小，以提高GC回收的效率。
	   MaxHeapSize      = 2124414976 (2026.0MB)
	   NewSize          = 1310720 (1.25MB)
	   MaxNewSize       = 17592186044415 MB
	   OldSize          = 5439488 (5.1875MB)
	   NewRatio         = 2 #新生代和老年代占用比例，-XX:NewRatio=2设置
	   SurvivorRatio    = 8 # Eden和一块Survivor的大小比例是8：1，新生代采用复制算法，垃圾回收时将Eden和一块Survivor中存活的对象复制到另一块Survivor中，并清空上一块Survivor。因此新生代可用容量为90%（80%+10%），10%的Survivor总是空的
	   PermSize         = 21757952 (20.75MB)
	   MaxPermSize      = 85983232 (82.0MB)
	
	Heap Usage:
	PS Young Generation
	Eden Space:
	   capacity = 49020928 (46.75MB)
	   used     = 30167344 (28.769821166992188MB)
	   free     = 18853584 (17.980178833007812MB)
	   61.53972442137366% used
	From Space:
	   capacity = 4325376 (4.125MB)
	   used     = 376912 (0.3594512939453125MB)
	   free     = 3948464 (3.7655487060546875MB)
	   8.713970762310606% used
	To Space:
	   capacity = 5898240 (5.625MB)
	   used     = 0 (0.0MB)
	   free     = 5898240 (5.625MB)
	   0.0% used
	PS Old Generation
	   capacity = 88473600 (84.375MB)
	   used     = 56496112 (53.87889099121094MB)
	   free     = 31977488 (30.496109008789062MB)
	   63.85646339699074% used
	PS Perm Generation
	   capacity = 21757952 (20.75MB)
	   used     = 10613560 (10.121879577636719MB)
	   free     = 11144392 (10.628120422363281MB)
	   48.78014254282756% used

## jstack：Java堆栈跟踪工具



## 垃圾收集器选择（交互优先、吞吐量优先）

1、ParNew + CMS

需要与**用户交互**的程序，良好的响应速度能提升用户的体验

配置：

	java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
	-XX:+UseConcMarkSweepGC：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
	-XX:+UseParNewGC: 设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。

	java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
	-XX:CMSFullGCsBeforeCompaction：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
	-XX:+UseCMSCompactAtFullCollection：打开对年老代的压缩。可能会影响性能，但是可以消除碎片

2、Parallel Scavenge + Paralle Old

后台任务，**吞吐量优先**，最高效率地利用 CPU 时间，尽快地完成程序的运算任务

典型配置：

	java -server -Xms3800m -Xmx3800m -Xmn2g -Xss128k -XX:+UseParallelOldGC -XX:ParallelGCThreads=4 -XX:GCTimeRatio=99 -XX:+UseAdaptiveSizePolicy
	-XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。

垃圾收集器介绍：

Parallel Scavenge 收集器的目标则是达到一个**可控制的吞吐量（Throughput）**。所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量 = 运行用户代码时间 /（运行用户代码时间 + 垃圾收集时间），虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。

由于与吞吐量关系密切，Parallel Scavenge收集器也经常被称为**"吞吐量优先"**收集器。

该垃圾收集器，是JAVA虚拟机在 Server 模式下的默认值，使用 **Server** 模式后，Java虚拟机使用 **Parallel Scavenge收集器（新生代）+ Serial Old收集器（老年代）** 的收集器组合进行内存回收。

<font color='red'>**参数说明：**</font>

重要的参数有三个，其中两个参数用于**精确控制吞吐量**，分别是控制最大垃圾收集停顿时间的**-XX:MaxGCPauseMillis**参数及直接设置吞吐量大小的 **-XX:GCTimeRatio** 参数。另外一个是 **UseAdaptiveSizePolicy** 开关参数。

**MaxGCPauseMillis** 参数允许的值是一个大于0的毫秒数，收集器将尽力保证内存回收花费的时间不超过设定值。不过大家不要异想天开地认为如果把这个参数的值设置得稍小一点就能使得系统的垃圾收集速度变得更快，GC停顿时间缩短是以牺牲吞吐量和新生代空间来换取的：系统把新生代调小一些，收集300MB新生代肯定比收集500MB快吧，这也直接导致垃圾收集发生得**更频繁一些，停顿间隔减少**，原来10秒收集一次、每次停顿100毫秒，现在变成5秒收集一次、每次停顿70毫秒。**停顿时间的确在下降，但吞吐量也降下来了。**

**GCTimeRatio** 参数的值应当是一个大于0小于100的整数，**是用户线程时间与垃圾收集时间比率**。如果把此参数设置为19，那允许的最大GC时间就占总时间的5%（即1 /（1+19）），**默认值为99**，就是允许最大1%（即1 /（1+99））的垃圾收集时间，经过计算**GCTimeRation = 吞吐量/(1-吞吐量)**，如要求吞吐量为99%，则GCTimeRatio = 0.99/（1-0.99） = 99，吞吐量95%，GCTimeRatio = 0.95/(1-0.95) = 19

**-XX:+UseAdaptiveSizePolicy** 是一个开关参数，当这个参数打开之后，就**不需要手工指定新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX:SurvivorRatio）、晋升老年代对象年龄（-XX:PretenureSizeThreshold）等细节参数了**，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大的吞吐量，这种调节方式称为GC自适应的调节策略（GC Ergonomics）。

如果对于收集器运作原理不太了解，**手工优化存在困难**的时候，使用Parallel Scavenge收集器能够配合自适应调节策略，把内存管理的调优任务交给虚拟机去完成，也是一个不错的选择。只需要把**基本的内存数据**设置好（如-Xmx设置最大堆），然后使用 **MaxGCPauseMillis **参数（更关注最大停顿时间）或 **GCTimeRatio** 参数（更关注吞吐量）给虚拟机设立一个优化目标，那具体细节参数的调节工作就由虚拟机完成了。自适应调节策略也是Parallel Scavenge收集器与ParNew收集器的一个重要区别。

## GC 相关参数总结

1、与串行回收器相关的参数

<pre>
-XX:+UseSerialGC:虚拟机运行在 Client 模式下的默认值，打开此开关后，使用 Serial + Serial Old 的收集器组合进行内存回收
-XX:SurvivorRatio: 新生代中 Eden 区域与 Survivor 区域的容量比值，默认为 8，代表 Eden：Survivor = 8 : 1
-XX:PretenureSizeThreshold:设置大对象直接进入老年代的阈值。当对象的大小超过这个值时，将直接在老年代分配。
-XX:MaxTenuringThreshold:设置对象进入老年代的年龄的最大值。每一次 Minor GC 后，对象年龄就加 1。任何大于这个年龄的对象，一定会进入老年代。
-XX:+HandlePromotionFailure:是否允许分配担保失败。即老年代的剩余空间不足以应付新生代的整个 Eden 和 Survivor 区的所有对象都存活的极端情况，java5以前是默认关闭，java6后默认启用
</pre>

2、与并行 GC 相关的参数

<pre>
-XX:+UseParNewGC: 打开此开关后，使用 ParNew + Serial Old 收集器组合进行内存回收
-XX:+UserParallelGC:虚拟机运行在 Server 模式下的默认值，打开此开关后，使用 Parallel Scavenge + Serial Old （PS MarkSweep）的收集器组合进行内存回收
-XX:+UseParallelOldGC: 打开此开关后，使用 Parallel Scavenge + Parallel Old 的收集器组合进行内存回收
-XX:ParallelGCThreads：设置用于垃圾回收的线程数。通常情况下可以和 CPU 数量相等。但在 CPU 数量比较多的情况下，设置相对较小的数值也是合理的。
-XX:MaxGCPauseMills：设置最大垃圾收集停顿时间。它的值是一个大于 0 的整数。收集器在工作时，会调整 Java 堆大小或者其他一些参数，尽可能地把停顿时间控制在 MaxGCPauseMills 以内。仅在使用 Parallel Scavenge 收集器时生效
-XX:GCTimeRatio:设置吞吐量大小，它的值是一个 0-100 之间的整数。假设 GCTimeRatio 的值为 n（用户线程时间与垃圾回收时间的比值），那么系统将花费不超过 1/(1+n) 的时间用于垃圾收集。仅在使用 Parallel Scavenge 收集器时生效
-XX:+UseAdaptiveSizePolicy:打开自适应 GC 策略。在这种模式下，新生代的大小，eden 和 survivor 的比例、晋升老年代的对象年龄等参数会被自动调整，以达到在堆大小、吞吐量和停顿时间之间的平衡点。
</pre>

3、与 CMS 回收器相关的参数

<pre>
-XX:+UseConcMarkSweepGC: 打开此开关后，使用 ParNew + CMS + Serial Old 的收集器组合进行内存回收。Serial Old 收集器作为 CMS 收集器出现 Concurrent Mode Failure 失败后的后备收集器使用
-XX:+ParallelCMSThreads: 设定 CMS 的线程数量。
-XX:+CMSInitiatingOccupancyFraction:设置 CMS 收集器在老年代空间被使用多少后触发，默认为 68%。仅在使用 CMS 收集器时生效
-XX:+UseFullGCsBeforeCompaction:设定进行多少次 CMS 垃圾回收后，进行一次内存压缩。仅在使用 CMS 收集器时生效
-XX:+CMSClassUnloadingEnabled:允许对类元数据进行回收。
-XX:+CMSParallelRemarkEndable:启用并行重标记。
-XX:CMSInitatingPermOccupancyFraction:当永久区占用率达到这一百分比后，启动 CMS 回收 (前提是-XX:+CMSClassUnloadingEnabled 激活了)。
-XX:UseCMSInitatingOccupancyOnly:表示只在到达阈值的时候，才进行 CMS 回收。
-XX:+CMSIncrementalMode:使用增量模式，比较适合单 CPU。
</pre>

4、与 G1 回收器相关的参数

<pre>
-XX:+UseG1GC：使用 G1 回收器。
-XX:+UnlockExperimentalVMOptions:允许使用实验性参数。
-XX:+MaxGCPauseMills:设置最大垃圾收集停顿时间。
-XX:+GCPauseIntervalMills:设置停顿间隔时间。
</pre>

5、其他参数

<pre>
-XX:+DisableExplicitGC: 禁用显示 GC。
</pre>

6、堆设置

<pre>
-Xms:初始堆大小
-Xmx:最大堆大小
-XX:NewSize=n:设置年轻代大小
-XX:NewRatio=n:设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
-XX:SurvivorRatio=n:年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
-XX:MaxPermSize=n:设置持久代大小
</pre>

7、垃圾回收统计信息

<pre>
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:filename
</pre>
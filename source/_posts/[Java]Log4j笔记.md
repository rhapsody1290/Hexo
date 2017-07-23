---
title: Log4j笔记

date: 2016-08-08 00:00:00

categories:
- Java

tags:
- Java
- Log4j

---
## 依赖

	<dependency>
	    <groupId>log4j</groupId>
	    <artifactId>log4j</artifactId>
	    <version>1.2.17</version>
	</dependency>
	<dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.1.1</version>
    </dependency>

## 快速入门

1、新建一个JAva工程，导入包log4j-1.2.17.jar，整个工程最终目录如下

![](http://i.imgur.com/8ZkEHMI.png)

2、src同级创建并设置log4j.properties

	### 设置###
	log4j.rootLogger = debug,stdout,D,E
	
	### 输出信息到控制抬 ###
	log4j.appender.stdout = org.apache.log4j.ConsoleAppender
	log4j.appender.stdout.Target = System.out
	log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
	log4j.appender.stdout.layout.ConversionPattern = %d{yyyy-MM-dd HH:mm:ss,SSS} [%p] method:%l%n%m%n
	
	### 输出DEBUG级别以上的日志到=E://logs/error.log ###
	log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
	log4j.appender.D.File = E://logs/log.log
	log4j.appender.D.Append = true
	log4j.appender.D.Threshold = DEBUG 
	log4j.appender.D.layout = org.apache.log4j.PatternLayout
	log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
	
	### 输出ERROR级别以上的日志到=E://logs/error.log ###
	log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
	log4j.appender.E.File =E://logs/error.log 
	log4j.appender.E.Append = true
	log4j.appender.E.Threshold = ERROR 
	log4j.appender.E.layout = org.apache.log4j.PatternLayout
	log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

3、设置日志内容

	package com.mucfc;
	import org.apache.log4j.Logger;
	/**
	 *@author linbingwen
	 *@2015年5月18日9:14:21
	 */
	public class Test {
		private static Logger logger = Logger.getLogger(Test.class);  
	
	    /** 
	     * @param args 
	     */  
	    public static void main(String[] args) {  
	        // System.out.println("This is println message.");  
	
	        // 记录debug级别的信息  
	        logger.debug("This is debug message.");  
	        // 记录info级别的信息  
	        logger.info("This is info message.");  
	        // 记录error级别的信息  
	        logger.error("This is error message.");  
	    }  
	
	}

## Log4j的架构

Log4j由三个重要的组件构成：
* 日志写入器：控制日志信息的优先级，日志信息的优先级从高到低有ERROR、WARN、 INFO、DEBUG
* 日志输出终端：指定了日志将打印到控制台还是文件中
* 日志布局模式：控制日志信息的输出格式

![](http://i.imgur.com/5z0m4oX.png)

Logger类是日志包的核心，Logger的名称是大小写敏感的，并且名称之间有继承关系。子名由父名做前缀，用点号"."分隔，如x.y是x.y.z的父亲Logger。

## Log4j基本使用方法

### 定义配置文件

***1、配置根Logger，其语法为：***

	log4j.rootLogger = [ level ] , appenderName, appenderName, …

* level是日志记录的优先级，Log4j建议只使用四个级别，优先级从高到低分别是ERROR、WARN、INFO、DEBUG。通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关。比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来
* appenderName就是指日志信息输出到哪个地方。您可以同时指定多个输出目的地

***2.配置日志信息输出目的地Appender，其语法为：***

	log4j.appender.appenderName = org.apache.log4j.DailyRollingFileAppender

其中，Log4j提供的appender有以下几种：

* org.apache.log4j.ConsoleAppender（控制台），  
* org.apache.log4j.FileAppender（文件），  
* org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件），  
* org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件），  
* org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）

***3.配置日志信息的格式（布局），其语法为：***

	log4j.appender.appenderName.layout = org.apache.log4j.PatternLayout

其中，Log4j提供的layout有以e几种：

* org.apache.log4j.HTMLLayout（以HTML表格形式布局），  
* org.apache.log4j.PatternLayout（可以灵活地指定布局模式），  
* org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串），  
* org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）

### 在代码中使用Log4j

***1.得到记录器***

	static Logger logger = Logger.getLogger(Class clazz)；//在某对象中，用该对象所属的类作为参数
 
***2.读取配置文件()***

	PropertyConfigurator.configure ( String configFilename) ：读取使用Java的特性文件编写的配置文件

例如：

	PropertyConfigurator.configure("config/properties/log4j.properties");

在项目下面建立一个文件夹名为config即可，这是标准写法。注意log4j默认的相对路径是工程下面，非src或者bin

***3.插入记录信息（格式化日志信息）***

	Logger.debug ( Object message ) ;  
	Logger.info ( Object message ) ;  
	Logger.warn ( Object message ) ;  
	Logger.error ( Object message ) ;

### 配置文件相对路径

	log4j.appender.R.File=${user.dir}/logs/log.log

## commons-logging 和 log4j 之间的关系

### Log4j与通用日志包commons-logging的结合使用

其实commons-logging中***默认都支持*** Log4j，因此只要同时加载commons-logging包和log4j包，可以不用配置即可用在应用中使用commons-logging的接口方法。	

当然，标准的应用的是需要的配置，如果你log4j则这个配置是可选的。下面我说明如何通过配置文件来组合commons-logging和log4j。

配置文件内容很简单，就指定一个日志实现类即可，下面是个示例文件commons-logging.properties：

	org.apache.commons.logging.Log=org.apache.commons.logging.impl.Log4JLogger
	org.apache.commons.logging.LogFactory=org.apache.commons.logging.impl.LogFactoryImpl

### commons-logging 和 log4j 之间的关系

我们在做项目时，日志的记录是必不可少的一项任务，而我们通常是使用 apache 的 log4j 日志管理工具。然而，在项目中，我们经常会看到两个 jar 包：commons-logging.jar 和 log4j.jar。为什么我们在使用 log4j 的同时还要引入 commons-logging.jar 呢，或者说不用 commons-logging.jar 可不可以，这两者之间到底是怎么的一种关系呢？ 

作为记录日志的工具，它至少应该包含如下几个组成部分(组件)： 

1. Logger 
	记录器组件负责产生日志，并能够对日志信息进行分类筛选，控制什么样的日志应该被输出，什么样的日志应该被忽略。它还有一个重要的属性 － 日志级别。不管何种日志记录工具，大概包含了如下几种日志级别：DEBUG, INFO, WARN, ERROR 和 FATAL。 
2. Level 
    日志级别组件。 
3. Appender 
    日志记录工具基本上通过 Appender 组件来输出到目的地的，一个 Appender 实例就表示了一个输出的目的地。 
4. Layout 
    Layout 组件负责格式化输出的日志信息，一个 Appender 只能有一个 Layout。 

我们再来看看 log4j.jar，打开 jar 包，我们可以看到 Logger.class(Logger)，Level.class(Level), FileAppender.class(Appender)， HTMLLayout.class(Layout)。其它的我们先忽略不看，这几个字节码文件正好是记录日志必不可少的几个组件。 

接下来看看 commons-logging 中的 org.apache.commons.logging.Log.java 源码： 

	package org.apache.commons.logging;  
	public interface Log {  
	    public boolean isDebugEnabled();  
	    public boolean isErrorEnabled();  
	    public boolean isFatalEnabled();  
	    public boolean isInfoEnabled();  
	    public boolean isTraceEnabled();  
	    public boolean isWarnEnabled();  
	    public void trace(Object message);  
	    public void trace(Object message, Throwable t);  
	    public void debug(Object message);  
	    public void debug(Object message, Throwable t);  
	    public void info(Object message);  
	    public void info(Object message, Throwable t);  
	    public void warn(Object message);  
	    public void warn(Object message, Throwable t);  
	    public void error(Object message);  
	    public void error(Object message, Throwable t);  
	    public void fatal(Object message);  
	    public void fatal(Object message, Throwable t);  
	}  


很显然，只要实现了 Log 接口，它就是一个名副其实的 Logger 组件，也验证了 Logger 组件具有日志级别的属性。继续看 commons-logging org.apache.commons.logging.impl 包下的几个类的源码片段： 

	package org.apache.commons.logging.impl;  
	  
	import org.apache.commons.logging.Log;  
	import org.apache.log4j.Logger;  
	import org.apache.log4j.Priority;  
	import org.apache.log4j.Level;  
	import ......  
	  
	public class Log4JLogger implements Log, Serializable {  
	    // 对 org.apache.commons.logging.Log 的实现  
	    ......  
	}  
	  
	------------------------------------------------------------------  
	  
	package org.apache.commons.logging.impl;  
	  
	import org.apache.commons.logging.Log;  
	import java.io.Serializable;  
	import java.util.logging.Level;  
	import java.util.logging.Logger;  
	  
	public class Jdk14Logger implements Log, Serializable {  
	     // 对 org.apache.commons.logging.Log 的实现  
	    ......  
	}  


好了，分析到这里，我们应该知道，<font color='red'>真正的记录日志的工具是 log4j 和 sun 公司提供的日志工具。而 commons-logging 把这两个(实际上，在 org.apache.commons.logging.impl 包下，commons-logging 仅仅为我们封装了 log4j 和 sun logger)记录日志的工具重新封装了一遍(Log4JLogger.java 和 Jdk14Logger.java)，可以认为 org.apache.commons.logging.Log 是个傀儡，它只是提供了对外的统一接口。因此我们只要能拿到 org.apache.commons.logging.Log，而不用关注到底使用的是 log4j 还是 sun logger。</font>正如我们经常在项目中这样写： 

	// Run 是我们自己写的类，LogFactory 是一个专为提供 Log 的工厂(abstract class)  
	private static final Log logger = LogFactory.getLog(Run.class);  


既然如此，我们向构建路径加了 commons-logging.jar 和 log4j.jar 两个 jar 包，那我们的应用程序到底使用的 log4j 还是 sun logger 呢？我们能不能认为由于加了 log4j.jar 包，就认为系统使用的就是 log4j 呢？事实上当然不是这样的，那我还认为我正在使用 jdk 而认为系统使用的是 sun logger 呢。使用 Spring 的朋友可以在 web.xml 中看到如下 listener 片段： 

	<listener>  
	    <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>  
	</listener>  


这是由 Spring 为我们提供的实现了标准的 servlet api 中的 javax.servlet.ServletContextListener 接口，用于在 web 容器启动时做一些初始化操作。我们逐层进入 Spring 的源码，可以看到如下代码： 

	Log4jConfigurer.initLogging(location, refreshInterval);  


终于找到了 org.springframework.util.Log4jConfigurer，这正是 log4j 提供给我们的初始化日志的类。至此，我们终于明白了我们系统的的确确使用的是 log4j 的日志工具。 

可是问题又来了，org.apache.commons.logging.Log 和 org.apache.log4j.Logger 这两个类，通过包名我们可以发现它们都是 apache 的项目，既然如下，为何要动如此大的动作搞两个东西(指的是 commons-logging 和 log4j)出来呢？事实上，在 sun 开发 logger 前，apache 项目已经开发了功能强大的 log4j 日志工具，并向 sun 推荐将其纳入到 jdk 的一部分，可是 sun 拒绝了 apache 的提议，sun 后来自己开发了一套记录日志的工具。可是现在的开源项目都使用的是 <font color='red'>***log4j***</font>，log4j 已经成了事实上的标准，但由于又有一部分开发者在使用 <font color='red'>***sun logger***</font>，因此 apache 才推出 commons-logging，<font color='red'>***使得我们不必关注我们正在使用何种日志工具***</font>。

## slf4j-api、slf4j-log4j12以及log4j之间的关系

几乎在每个jar包里都可以看到log4j的身影，在多个子工程构成项目中，slf4j相关的冲突时不时就跳出来让你不爽，那么slf4j-api、slf4j-log4j12还有log4j是什么关系？ 
    
***slf4j:Simple Logging Facade for Java，为java提供的简单日志Facade。***Facade门面，更底层一点说就是**接口**。<font color='red'>***它允许用户以自己的喜好，在工程中通过slf4j接入不同的日志系统***</font>。更直观一点，slf4j是个数据线，一端嵌入程序，另一端链接日志系统，从而实现将程序中的信息导入到日志系统并记录。
 
因此slf4j入口就是众多接口的集合，它不负责具体的日志实现，只在编译时负责寻找合适的日志系统进行绑定。具体有哪些接口，全部都定义在slf4j-api中。查看slf4j-api源码就可以发现，里面除了public final class LoggerFactory类之外，都是接口定义。因此slf4j-api本质就是一个接口定义。

下图比较清晰的描述了它们之间的关系，例子为当系统采用log4j作为日志框架实现的调用关系：

![](http://i.imgur.com/0GvNDWm.png)
 
①首先系统包含slf4j-api作为日志接入的接口。<font color='red'>compile时slf4j-api中public final class LoggerFactor类中private final static void bind()方法会寻找具体的日志实现类绑定，主要通过StaticLoggerBinder.getSingleton()的语句调用。</font>

②slf4j-log4j12是链接slf4j-api和log4j中间的适配器。它实现了slf4j-apiz中StaticLoggerBinder接口，从而使得在编译时绑定的是slf4j-log4j12的getSingleton()方法。

③log4j是具体的日志系统。通过slf4j-log4j12初始化Log4j，达到最终日志的输出。

## 参考文献

http://www.codeceo.com/article/log4j-usage.html
http://blog.csdn.net/hpf911/article/details/5852127
http://www.tuicool.com/articles/U7ZjUni
http://zachary-guo.iteye.com/blog/361177

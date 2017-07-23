---
title: Servlet笔记

date: 2016-07-06 11:17:00

categories:
- Servlet

tags:
- JavaEE
- Servlet

---

## 常用
  
	#网页输出
	PrintWriter out = response.getWriter();
	out.println("Hello World");
	#请求头
	request.getHeader("host")//参数见http请求头那节
	//获取所有请求体
	Enumeration<String> names = request.getHeaderNames();
	while(names.hasMoreElements()){
		String name = names.nextElement();
		System.out.println(name + "：" + request.getHeader(name));
	}
	#浏览器返回
	response.setContentType("text/html;charset=utf-8");
	response.setCharacterEncoding("utf-8");
	#获得请求参数
	String username = request.getParameter("username");
	Enumeration<String> e = request.getParameterNames()
	#跳转
	1、request.getRequestDispatcher("/资源URI").forward(request,response)
	2、response.sendRedirect("/web应用/资源URI");
	#获得web应用根路径
	String path = this.getServletContext().getRealPath("/");
	#获得资源路径
	String path = this.getServletContext().getRealPath("/image/无标题.png");
	#session，可以存字符串和对象
	request.getSession().setAttribute("username",username);
	request.getSession().getAttribute("username")

## 为什么需要Servlet技术？

　　普通的java技术很难完成网站开发，sun 就开发了servlet技术供程序员使用

## servlet的介绍

- servlet 其实就是java程序(java类)
- 该 java程序(java 类)要遵循servlet开发规范，继承servlet类
- serlvet是运行在服务端
- serlvet功能强大,几乎可以完成网站的所有功能
- 是学习jsp基础

## Tomcat 和 servlet 在网络中的位置

![](http://i.imgur.com/jvuMyhS.png)

### Tomcat三大功能

- Web服务器，与浏览器通信，解析和处理HTTP请求，处理静态页面
- Servlet容器（Catalina），处理Servlet
- JSP容器，把JSP页面翻译成一般的Servlet

### Servlet容器与Servlet关系★★★★★

* Servlet技术的核心是Servlet，所有的Servlet类必须直接或间接实现Servlet接口
* ***Servlet接口定义了Servlet与Servlet容器之间的契约***，即Servlet容器将Servlet类载入内存，并在Servlet实例是调用具体的方法，如用户请求时Servlet调用Servlet的Service方法，并传入一个ServletRequest实例和一个ServletResponse实例

## Web程序目录结构

![](http://i.imgur.com/0HhKiSj.png)

- 静态页面、jsp直接放在web目录下
- Servlet是Java程序，必须放在WEB-INF/classes目录下

## Servlet工作流程★★★

![](http://i.imgur.com/6yEbnhC.png)

    用户输入的URL为http://localhost:8088/hspWeb1/MyFirstServlet
    浏览器解析主机名（host文件，dns）
    浏览器尝试连接web服务器（三次握手）
    浏览器发送http请求
    web服务器开始工作，首先解析主机名，选择使用哪个主机（引擎下有多个主机，有默认主机）
    web服务器解析web应用，确定使用主机下的context
    web服务器解析资源名，一个web应用下有多个资源，这里是MyFirstServlet
    查询web.xml文件，确定MyFirstServlet在哪个包下
    Web服务器利用反射机制，创建实例，并调用init方法（该方法只调用一次）
    web服务器把接收到的http请求封装成Request对象，作为service的参数传入。service会被调用多次，没访问一次Servlet，它的service就会被调用一次
    Servlet获得response对象，返回结果
    web服务器把request的信息拆除，形成http响应格式
    当在某些情况下（tomcat重启、reload该webapp、重启电脑）web服务器会调用Servlet的destroy方法，将该servlet销毁

## 面试题: 请简述servlet的生命周期(工作流程)

答:  
　　当第一次访问某个servlet，web服务器将会创建一个该servlet的实例，并且调用 servlet的init()方法，init函数只会被调用一次；如果当服务器已经存在了一个servlet实例，那么，将直接使用此实例；每次请求都会调用service()方法，service()方法将根据客户端的请求方式来决定调用对应的doXXX()方法；当 web应用 reload 或者 关闭 tomcat 或者 关机，web服务器将调用destroy()方法，将该servlet从服务器内存中删除。

## 开发Servlet程序

***开发servlet有三种方法★★★***

- (1)	实现 Servlet接口(对Servlet的工作过程有清晰的认识)
- (2)	通过继承 GenericServlet
- (3)	通过继承 HttpServlet

### ①实现servlet接口的方式

需求如下: 请使用实现接口的方式，来开发一个Servlet，要求该Servlet可以显示Hello，world，同时显示当前时间

#### 步骤

1、在webapps下建立一个web应用my  
2、在my下建立 WEB-INF->web.xml [web.xml可以从 ROOT/WEB-INF/web.xml拷贝]  
3、在WEB-INF下建立 classes 目录(我们的Servlet 就要在该目录开发)，建立lib文件夹  
4、开发MyServlet.java  

<pre>
package com.apeius;
import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;
public class MyServlet implements Servlet
{
	//该函数用于初始化servlet,就是把该servlet装载到内存中,该函数只会被调用一次
	<font color='red'>/*调用这个方法，Servlet容器会传入一个ServletConfig，
	* 一般会将ServletConfig赋给一个类级对象，这样可以在Servlet类中其他点来使用，
	* 但Servlet实例会被一个应用程序的所有用户共享，使用类级变量须是只读的，或者是
	* java.util.concurrent.atomic包的成员
	* /</font>
	public void init(ServletConfig config)
          throws ServletException{
	}

	//得到ServletConfig对象
	public ServletConfig getServletConfig(){
		return null;
	}
	
	//该函数是服务函数,我们的业务逻辑代码就是写在这里
	//该函数每次都会被调用
	public void service(ServletRequest req,
                    ServletResponse res)
             throws ServletException,
                    java.io.IOException{
        //在控制台输出时间
		System.out.println(new java.util.Date());
	}
	//该函数时得到servlet配置信息
	public java.lang.String getServletInfo(){
		return null;
	}
	//销毁该servlet,从内存中清除,该函数被调用一次
	public void destroy(){

	}
}
</pre>

5、编译

如果使用javac去编译一个带package的java文件，则需要带命令参数`javac –d . java文件`

6、根据Servlet规范，我们还需要部署Servlet

	<?xml version="1.0" encoding="ISO-8859-1"?>
	<web-app xmlns="http://java.sun.com/xml/ns/javaee"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	   version="2.5">
	
		<!--根据serlvet规范，需要将Servlet部署到web.xml文件,该部署配置可以从examples下拷贝-->
		<servlet>
		  <!--servlet-name 给该Servlet取名, 该名字可以自己定义:默认就使用该Servlet的名字-->
	      <servlet-name>MyServlet</servlet-name><!--③-->
		  <!--servlet-class要指明该Servlet 放在哪个包下 的,形式是 包/包/../类-->
	      <servlet-class>com.apeius.MyServlet</servlet-class> <!--注意:后面不要带.java④-->
	    </servlet>
			<!--Servlet的映射-->
		<servlet-mapping>
			<!--这个Servlet-name要和上面的servlet-name名字一样-->
	        <servlet-name>MyServlet</servlet-name><!--②-->
			<!--url-pattern 这里就是将来访问该Servlet的资源名部分，默认命名规范就是Servlet的名字-->
	        <url-pattern>/ABC</url-pattern><!--①则访问的url为localhost:8080/my/ABC-->
	    </servlet-mapping>
	</web-app>
	
	服务器调用流程：http://localhost:8088/my/ABC--->①--->②--->③--->④

7、在浏览器中测试

　　在浏览器中输入http://localhost:8080/my/ABC

### ②使用GenericServlet开发servlet（了解即可）

#### 为什么使用GenericServlet

1、实现Servlet接口***必须实现接口中的所有方法***，即使有一些方法根本没有包含任何代码
2、此外还***需要将ServletConfig对象保存到类级变量中***

#### GenericServlet完成的任务

* 将init方法中个ServletConfig赋给一个类级变量，以便可以通过getServletConfig获取
* 为Servlet接口中的所有方法提供默认的实现
* 提供方法，包围ServletConfig中的方法

#### GenericServlet原理

GenericServlet通过将ServletConfig赋给init方法中的类级变量private transient ServletConfig config；来保存ServletConfig

	public void init(ServletConfig config) throws ServletException {
		this.config = config;
		this.init();
    }

但是，如果在子类中覆盖了这个方法，就会调用Servlet中的init方法，并且还必须调用super.init(servletConfig)来保存ServletConfig，为了避免上述麻烦，***GenericServlet提供了第二个init方法，它不带参数。***这个方法是在ServletConfig被赋给servletConfig后，由第一个init方法调用，子类改写后调用的是子类的无参数init方法


**总结：**
Tomcat调用Servlet接口的init(ServletConfig config)方法，其实现类是用户继承的子类，该子类重写了无参数的init方法进行初始化工作。首先调用父类中的init(ServletConfig config)方法，将config对象保存为类级变量，然后调用this.init()方法，this指针指向子类实例，init方法被重写，所以调用子类的init方法，若需要访问GenericServlet中的无参数init方法，则需要在子类中使用super.init()；

#### 开发步骤

***继承GenericServlet***

	package com.hsp;
	import javax.servlet.*;
	import javax.servlet.http.*;
	import java.io.*;
	public class MyGenericServlet extends GenericServlet
	{
		public  void service(ServletRequest req,
	                             ServletResponse res)
	                      throws ServletException,
	                             java.io.IOException{
			res.getWriter().println("hello,world,i am geneirc servlet");
		}
	}

***将该Servlet部署到web.xml文件中:***

	<!--根据serlvet规范，需要将Servlet部署到web.xml文件,该部署配置可以从examples下拷贝-->
	 <servlet>
		<!--servlet-name 给该Servlet取名, 该名字可以自己定义:默认就使用该Servlet的名字-->
      <servlet-name>MyGenericServlet</servlet-name>
	  <!--servlet-class要指明该Servlet 放在哪个包下 的,形式是 包/包/../类-->
      <servlet-class>com.hsp.MyGenericServlet</servlet-class>
    </servlet>
		<!--Servlet的映射-->
	 <servlet-mapping>
		<!--这个Servlet-name要和上面的servlet-name名字一样-->
        <servlet-name>MyGenericServlet</servlet-name>
		<!--url-pattern 这里就是将来访问该Servlet的资源名部分,默认命名规范:
		就是该Servlet的名字-->
        <url-pattern>/MyGenericServlet</url-pattern>
	</servlet-mapping>


### ③使用继承 HttpServlet 的方法来开发Serlvet

- 在软件公司 90%都是通过该方法开发.
- 举例说明，还是显示 hello,world 当前日期

#### 原理

* HttpServlet类继承了GenericServlet类，并重写了service(<font color='red'>ServletRequest</font> req,<font color='red'>ServletResponse</font> res)方法，Tomcat调用Servlet接口的service(<font color='red'>ServletRequest</font> req,<font color='red'>ServletResponse</font> res)方法，实际调用的是HttpServlet类中的的service(<font color='red'>ServletRequest</font> req,<font color='red'>ServletResponse</font> res)方法，方法体内将ServletRequest和ServletResponse强转成HttpServletRequest和HttpServletResponse，并调用新添加的service方法


	public void service(ServletRequest req, ServletResponse res)
		throws ServletException, IOException
	    {
			HttpServletRequest	request;
			HttpServletResponse	response;
		
		try {
		    request = (HttpServletRequest) req;
		    response = (HttpServletResponse) res;
		} catch (ClassCastException e) {
		    throw new ServletException("non-HTTP request or response");
		}
	
		service(request, response);
	}

* HttpServlet中扩展了一个protected void service(HttpServletRequest req, HttpServletResponse resp)方法，根据不同的请求方法，调用不同的doXXX方法


	protected void service(HttpServletRequest req, HttpServletResponse resp)
		throws ServletException, IOException
	    {
		String method = req.getMethod();
	
		if (method.equals(METHOD_GET)) {
		    long lastModified = getLastModified(req);
		    if (lastModified == -1) {
			// servlet doesn't support if-modified-since, no reason
			// to go through further expensive logic
			doGet(req, resp);
		    } else {
			long ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
			if (ifModifiedSince < (lastModified / 1000 * 1000)) {
			    // If the servlet mod time is later, call doGet()
	                    // Round down to the nearest second for a proper compare
	                    // A ifModifiedSince of -1 will always be less
			    maybeSetLastModified(resp, lastModified);
			    doGet(req, resp);
			} else {
			    resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
			}
		    }
	
		} else if (method.equals(METHOD_HEAD)) {
		    long lastModified = getLastModified(req);
		    maybeSetLastModified(resp, lastModified);
		    doHead(req, resp);
	
		} else if (method.equals(METHOD_POST)) {
		    doPost(req, resp);
		    
		} else if (method.equals(METHOD_PUT)) {
		    doPut(req, resp);	
		    
		} else if (method.equals(METHOD_DELETE)) {
		    doDelete(req, resp);
		    
		} else if (method.equals(METHOD_OPTIONS)) {
		    doOptions(req,resp);
		    
		} else if (method.equals(METHOD_TRACE)) {
		    doTrace(req,resp);
		    
		} else {
		    //
		    // Note that this means NO servlet supports whatever
		    // method was requested, anywhere on this server.
		    //
	
		    String errMsg = lStrings.getString("http.method_not_implemented");
		    Object[] errArgs = new Object[1];
		    errArgs[0] = method;
		    errMsg = MessageFormat.format(errMsg, errArgs);
		    
		    resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
		}
	}

* 子类继承后，通过改写doGet和doPost方法，实现方法的调用

#### 开发步骤

***代码***

	package com.apeius;
	
	import javax.servlet.*;
	import javax.servlet.http.*;
	import java.io.*;
	
	public class MyHttpServlet extends HttpServlet
	{
		//在HttpServlet 中，设计者对post 提交和 get提交分别处理
		//回忆 <form action="提交给?" method="post|get"/>,默认是get
	
		protected void doGet(HttpServletRequest req,
	                     HttpServletResponse resp)
	              throws ServletException,
	                     java.io.IOException{
			resp.getWriter().println("i am httpServet doGet()");
			
		}
		protected void doPost(HttpServletRequest req,
	                      HttpServletResponse resp)
	               throws ServletException,
	                      java.io.IOException{ 
			resp.getWriter().println("i am httpServet doPost() post name="+req.getParameter("username"));
		}
	}

***还有一个login.html***

	<html>
	<body>
	<form action="/my/MyHttpServlet" method="post">
	    u:<input type="text" name="username"/>
	    <input type="submit" value="login"/>
	</body>
	</html>

***部署***

	<!--根据serlvet规范，需要将Servlet部署到web.xml文件,该部署配置可以从examples下拷贝-->
		<servlet>
		  <!--servlet-name 给该Servlet取名, 该名字可以自己定义:默认就使用该Servlet的名字-->
	      <servlet-name>MyHttpServlet</servlet-name><!--③-->
		  <!--servlet-class要指明该Servlet 放在哪个包下 的,形式是 包/包/../类-->
	      <servlet-class>com.apeius.MyHttpServlet</servlet-class> <!--注意:后面不要带.java④-->
	    </servlet>
			<!--Servlet的映射-->
		<servlet-mapping>
			<!--这个Servlet-name要和上面的servlet-name名字一样-->
	        <servlet-name>MyHttpServlet</servlet-name><!--②-->
			<!--url-pattern 这里就是将来访问该Servlet的资源名部分，默认命名规范就是Servlet的名字-->
	        <url-pattern>/MyHttpServlet</url-pattern><!--①则访问的url为localhost:8080/my/ABC-->
	    </servlet-mapping>


## 使用myeclipse来开发servlet，IDEA类似

***(1)	建立web工程***

![](http://i.imgur.com/6IcaPJQ.gif)

***(2)	在Src 目录下创建了一个包 com.hsp.servlet***

![](http://i.imgur.com/VYwK3wL.png)

***添加Package，方法一般只需创建doGet()和doPost()方法，修改Servlet/JSP Mapping URL***

![](http://i.imgur.com/ogGRxkN.png)


***(3)	开发一个Servlet***

	MySerlvet 的代码:
	public void doGet(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {
	
			response.setContentType("text/html");
			PrintWriter out = response.getWriter();
			out.println("hello "+new java.util.Date().toString() );
		}
	
		public void doPost(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {
	
			this.doGet(request, response);
		}
	}
	

## 使用IDEA来开发Servlet程序
	
　　见另一篇博文:<a href="https://rhapsody1290.github.io/2016/07/07/[Servlet]Intellij%20IDEA%E5%88%9B%E5%BB%BAMaven%E7%AE%A1%E7%90%86%E7%9A%84Java%20Web%E9%A1%B9%E7%9B%AE/">Intellij IDEA15创建Maven管理的Java Web项目</a>

## Servlet的细节问题[映射、单例、通配符、自启动]

### ① 一个已经注册的Servlet可以被多次映射即:

	<servlet>
	<description>This is the description of my J2EE component</description>
	<display-name>This is the display name of my J2EE component</display-name>
	<!-- servlet的注册名 -->
	<servlet-name>MyServlet1</servlet-name>
	<!-- servlet类的全路径(包名+类名) -->
	<servlet-class>com.hsp.servlet.MyServlet1</servlet-class>
	</servlet>
	
	<!-- 对一个已经注册的servlet的映射 -->
	<!-- 映射1 -->
	<servlet-mapping>
		<!-- servelt的注册名 -->
		<servlet-name>MyServlet1</servlet-name>
		<!-- servlet的访问路径 -->
		<url-pattern>/MyServlet1</url-pattern>
	</servlet-mapping>
	
	<!-- 映射2 -->
	<servlet-mapping>
		<servlet-name>MyServlet1</servlet-name>
		<url-pattern>/hsp</url-pattern>
	</servlet-mapping>

---

### ② 映射一个servlet时候，可以多层，比如

	<url-pattern>/servlet/index.html</url-pattern>

　　从这里还可以看出，后缀名是 html 不一定就是 html,可能是假象.


### ③ 用通配符在servlet映射到URL中

***有两种格式:***

* 第一种格式  `*.扩展名`，比如 `*.do`，`*.ss`
* 第二种格式  以 `/` 开头，同时以 `/*` 结尾。比如  `/*`,`/news/*` 

***在匹配的时候，要参考的标准:***

- 看谁的匹配度高，谁就被选择
- `*.do`的优先级最低

***通配符练习题：***

    ● Servlet1 映射到 /abc/* 
    ● Servlet2 映射到 /* 
    ● Servlet3 映射到 /abc 
    ● Servlet4 映射到 *.do 
    问题(面试题)：
    当请求URL为'/abc/a.html'，'/abc/*'和'/*'都匹配，哪个servlet响应
    	Servlet引擎将调用Servlet1。
    当请求URL为'/abc'时，'/abc/*'和'/abc'都匹配，哪个servlet响应
    	Servlet引擎将调用Servlet3。
    当请求URL为“/abc/a.do”时，“/abc/*”和“*.do”都匹配，哪个servlet响应
    	Servlet引擎将调用Servlet1。
    当请求URL为“/a.do”时，“/*”和“*.do”都匹配，哪个servlet响应
    	Servlet引擎将调用Servlet2。
    当请求URL为“/xxx/yyy/a.do”时，“/*”和“*.do”都匹配，哪个servlet响应
    	Servlet引擎将调用Servlet2。

### ④Servlet单例问题

* 当Servlet被第一次访问后，就被加载到内存，以后该实例对各个请求服务，即在使用中是单例
* ***Servlet实例会在一个应用程序中被所有用户共享，因此不建议使用类级变量*** ，除非它们是只读的，或者是java.util.concurrent.atomic包的成员

***证明：***

　　在Servlet中定义一个变量i，当浏览器访问时i++，并输出i；如果Servlet是单例，则每次输出i都会增加

***问题：***

　　因为 Servlet是单例，因此会出现线程安全问题: 比如:售票系统. 如果不加同步机制，则会出现问题:

***原则:***

* （1）如果一个变量需要多个用户共享，则应当在访问该变量的时候，加同步机制  

<pre>
synchronized (对象){
  //同步代码
}
</pre>

* （2）如果一个变量不需要共享，则直接在 doGet() 或者 doPost()定义.这样不会存在线程安全问题  

### ⑤servlet中的&lt;load-on-startup&gt;配置

***需求***

　　当我们的网站启动的时候，可能会要求初始化一些数据，(比如创建临时表), 在比如：我们的网站有一些要求定时完成的任务[ 定时写日志，定时备份数据.. 定时发送邮件..]

***解决方法***

　　可以通过 < load-on-startup > 配合线程知识搞定.
　　一般在有用户访问该Servlet时才会被加载进内存，现在需要在网站启动的时候自动启动Servlet。首先在web.xml下，该Servlet下进行配置
`<load-on-startup >1（数字Servlet启动优先级）</load-on-startup>`
　　这样该Servlet在网站启动时将会被自动创建.

##  ServletConfig对象

* 调用Servlet的init方法时，Servlet容器会传入一个ServletConfig实例，该对象主要用于读取 servlet的配置信息

***案例***

	<servlet>
	    <servlet-name>ServletConfigTest</servlet-name>
	    <servlet-class>com.hsp.servlet.ServletConfigTest</servlet-class>
	    <!-- 这里可以给servlet配置信息,这里配置的信息，只能被该servlet 读取 -->
	    <init-param>
	        <param-name>encoding</param-name>
	        <param-value>utf-8</param-value>
	    </init-param>
	</servlet>

	获得配置参数
	String encoding=this.getServletConfig().getInitParameter("encoding");


***补充说明***

　　这种配置参数的方式，只能被某个Servlet独立使用.如希望让所有的Servlet都去读取某个参数,这样配置:

	<!-- 如果这里配置参数，可被所有servlet读取 -->
	<context-param>
	    <param-name>encoding</param-name>
	    <param-value>utf-8</param-value>
	</context-param>

	获得配置参数
	String encoding = this.getServletContext().getInitParameter("encoding")

***如果要把所有的参数都读取，则使用 如下方法 ：***

	Enumeration<String> names=this.getServletConfig().getInitParameterNames();
	while(names.hasMoreElements()){
		String name=names.nextElement();
		System.out.println(name);
		System.out.println(this.getServletConfig().getInitParameter(name));
	}

## ServletContext★★★★★（包含读取文件路径）
 
在访问某个网站时，首页会显示您是第几个浏览者，这个怎么实现的？除了数据库，文件等方式，最方便的是使用ServletContext。  
<font color='red'>ServletContext是一个公共的空间，可以被所有客户访问</font>

![](http://i.imgur.com/ZyoAE8n.png)

* Web容器在启动时，它会为每个Web应用程序创建一个对应的ServletContext对象，它代表当前Web应用。当web应用关闭/tomcat关闭/对web应用reload会造成servletContext销毁.
* ServletContext对象通过ServletConfig.getServletContext方法获得对ServletContext对象的引用，也可以通过this.getServletContext()来获得其对象的引用
* ***每个Web应用程序只有一个上下文，一个Web应用中所有的Servlet共享同一个ServletContext对象***，因此Servlet对象之间可以通过ServletContext对象实现通讯。ServletContext对象通常也被称为context域对象，公共聊天室就会用到它

### 基本使用

	获取:
	this.getServletContext();  #在HttpServlet中直接获取
	this.getServletConfig().getServletContext(); # 在ServletConfig中获取
	request.getSession().getServletContext(); #通过HttpRequest获得 
	添加属性:
	servletcontext.setAttribute(string,object);
	取出属性
	servletcontext.getAttribute(“属性名”)
	删除
	setvletContext.removeAttribute(“属性名”);

### ServletContext的应用

#### 获取WEB应用的初始化参数

	<!-- 如果希望所有的servlet都可以访问该配置. -->
	<context-param>
		<param-name>name</param-name>
		<param-value>scott</param-value>
	</context-param>
	如何获取
	String val= this.getServletContext().getInitParameter("name");

#### 使用ServletContext实现跳转

	//目前我们跳转到下一个页面的方法
	//1 response.sendRedirect("/web应用名/资源名");
	//2 request.getRequestDispatcher("/资源名").forward(request, response);
	/*
	 * 区别1. getRequestDispatcher 一个跳转发生在web服务器 sendRedirect发生在浏览器
	 *     2. 如果request.setAttribute("name","顺平") 希望下一个页面可以使用 属性值，则使用 getRequestDispatcher
	 *	   3. 如果session.setAttribute("name2","顺平3"), 希望下一个页面可以使用 属性值，则两个方法均可使用,但是建议使用 getRequestDispatcher
	 *     4. 如果我们希望跳转到本web应用外的一个url,应使用sendRedirect
	 */
	//3.这种方法和2一样
	this.getServletContext().getRequestDispatcher("/资源url").forward(request, response);

#### 读取文件，和获取文件全路径
	
　　读Web目录下和WEB-INF目录下的文件

	//首先读取到文件dbinfo.properties放在web根目录下
	InputStream inputStream=this.getServletContext().getResourceAsStream("dbinfo.properties");

	//创建Properties
	Properties pp=new Properties();
	pp.load(inputStream);
	
	out.println("name="+pp.getProperty("username"));

　　如果文件放在src目录下(如果是maven项目，则resources目录下为根目录)，则使用类加载器

	//如果文件放在上述目录下，我们应该使用类加载器来读取
	InputStream is=Servlet5.class.getClassLoader().getResourceAsStream("dbinfo.properties")
	//如果放在包下，则带上包名，例如cn/apeius/xx.properties
	

　　获取文件全路径（以WEB目录为根目录）

	//如果读取到一个文件的全路径（dbinfo.properties在web目录下）
	String path=this.getServletContext().getRealPath("dbinfo.properties");
	out.println("paht = "+path);

#### 网站计数器

* 我们建立一个文件recoder.txt文件，用于保存访问量,可以可以保证稳定增长.
* 建立InitServlet,Web项目启动时自动加载该Servlet，读取record.txt，初始化Servletcontext中的访问量，和在关闭tomcat时保存访问量如record.txt
* 如果我们的tomcat异常退出，使用线程定时把ServletContext的值，刷新到recorder.txt

## http请求消息头

    1.	Accept: text/html,image/*   [告诉服务器，我可以接受 文本，网页，图片]
    2.	Accept-Charset: ISO-8859-1 [接受字符编码 iso-8859-1]
    3.	Accept-Encoding: gzip,compress [可以接受 gzip,compress压缩后数据.]
    4.	Accept-Language: en-us,zh-cn [浏览器支持中，英文]
    5.	Host: www.sohu.com:80 [我要找主机是 www.sohu.com:80]
    6.	If-Modified-Since: Tue, 11 Jul 2000 18:23:51 GMT [ 告诉服务器，我的缓冲中有这个资源文件，该文件的时间是 。。。，文件有更新才发送]
    7.	Referer: http://www.sohu.com/index.jsp  [告诉服务器，我来自哪里,该消息头，常用于防止盗链]
    8.	User-Agent: Mozilla/4.0 (compatible; MSIE 5.5; Windows NT 5.0)[告诉服务器，浏览器内核]
    9.	Cookie [cookie后面介绍]
    10.	Connection: close/Keep-Alive   [保持连接，发完数据后，我不关闭连接]
    11.	Date: Tue, 11 Jul 2000 18:23:51 GMT [浏览器发送该http请求的时间]

## http响应消息头

    Location: http://www.baidu.org/index.jsp  【让浏览器重新定位到url】
    Server:apache tomcat 【告诉浏览器我是tomcat】
    Content-Encoding: gzip 【告诉浏览器我使用 gzip】
    Content-Length: 80  【告诉浏览器会送的数据大小80节】
    Content-Language: zh-cn 【支持中文】
    Content-Type: text/html; charset=GB2312 [内容格式text/html; 编码gab2312]
    Last-Modified: Tue, 11 Jul 2000 18:23:51 GMT 【告诉浏览器，该资源上次更新时间】
    Refresh: 1;url=http://www.baidu.com 【过多久去，刷新到 http://www.baidu.com】
    Content-Disposition: attachment; filename=aaa.zip 【告诉浏览器，有文件下载】
    Transfer-Encoding: chunked  [传输的编码]
    Set-Cookie:SS=Q0=5Lb_nQ; path=/search[后面详讲]
    Expires: -1[告诉浏览器如何缓存页面IE]
    Cache-Control: no-cache  [告诉浏览器如何缓存页面火狐]
    Pragma: no-cache   [告诉浏览器如何缓存页面]
    Connection: close/Keep-Alive   [保持连接 1.1是Keep-Alive]
    Date: Tue, 11 Jul 2000 18:23:51 GMT

## http响应的状态行
    200 就是整个请求和响应过程没有发生错误，这个最常见.
    302: 表示当你请求一个资源的时候，服务器返回302 表示，让浏览器转向到另外一个资源，比如: response.sendRedirect(“/web应用/资源名”)
    
    案例:
    	response.setStatus(302);
    	response.setHeader("Location", "/servletPro/Servlet2");
    	// 上面两句话等价 response.sendRedirect("/servletPro/Servlet2");
    
    404： 找不到资源
    500: 服务器端错误

## http响应头应用★★★[防盗链、定时、文件下载、缓存]

### 防盗链 - Referer

	//获取用户浏览器Referer
	String referer=request.getHeader("Referer");
	if(referer==null||!referer.startsWith("http://localhost:8088/servletPro")){
		response.sendRedirect("/servletPro/Error");
		return;
	}

### 定时刷新Refresh使用

	response.setHeader("Refresh", "5;url=/servletPro/Servlet2");

### 文件下载 Content-Disposition

	public void doGet(HttpServletRequest request, HttpServletResponse response)
				throws ServletException, IOException {
	
		response.setContentType("text/html");
		//★★★需要注释
		//PrintWriter out = response.getWriter();
		
		//演示下载文件
		response.setHeader("Content-Disposition", "attachment; filename=winter.jpg");
		
		//打开文件.说明一下web 站点下载文件的原理
		//1.获取到要下载文件的全路径
		String path=this.getServletContext().getRealPath("/images/Winter.jpg");
		//System.out.println("path="+path);
		//2创建文件输入流
		FileInputStream fis=new FileInputStream(path);
		//做一个缓冲字节数组
		byte buff[]=new byte[1024];
		int len=0;//表示实际每次读取了多个个字节
		OutputStream os=response.getOutputStream();
		while((len=fis.read(buff))>0){
			os.write(buff, 0, len);
		}
		//关闭
		os.close();
		fis.close();
	}

### 缓存讲解

　　提出问题：浏览器默认情况下，会缓存我们的页面，这样出现一个问题：如果我们的用户习惯把光标停留在地址栏，然后回车来取页面，就会默认调用cache中取数据（刷新还是会重新向服务器请求数据）。

***（1）有些网站要求及时性很高，因此要求我们不缓存页面***

	//指定该页面不缓存 Ie
	response.setDateHeader("Expires", -1);//【针对IE浏览器设置不缓存】
	//为了保证兼容性.
	response.setHeader("Cache-Control", "no-cache");//【针对火狐浏览器等】
	response.setHeader("Pragma", "no-cache");//【其他浏览器】

***（2）有些网站要求网页缓存一定时间,比如缓存一个小时***

	response.setDateHeader("Expires", System.currentTimeMillis()+3600*1000*24);后面一个参数表示设置的缓存保持时间，-1表示永远缓存


## HttpServletResponse的再说明

HttpServletResponse中输出流两个方法：

* getWriter()
* getOutputStream()

***区别***

1、	getWriter() 用于向客户机回送字符数据

	PrintWriter out = response.getWriter();
	out.println("hello,world");

2、	getOutputStream() 返回的对象，可以回送字符数据，也可以回送字节数据(二进制数据)，但也可以输出文本内容

	OutputStream os=response.getOutputStream();
	os.write("hello,world".getBytes());

***如何选择:***

* 如果我们是回送字符数据，则使用PrintWriter对象,效率高
* 如果我们是回送字节数据(binary date),则只能使用OutputStream
* 这两个流不能同时使用.

***比如：***

    OutputStream os=response.getOutputStream();
    os.write("hello,world".getBytes());
    PrintWriter out=response.getWriter();
    out.println("abc");
    就会报错:
    java.lang.IllegalStateException: getOutputStream() has already been called for this response
    
***注意：***
　　Web服务器会自动检查并关闭流，为什么我们没有主动关闭流，程序也没有问题的原因.当然：你主动关闭流，更好.
　　
## 参数的传递方式sendRedirect()和session()

![这里写代码片](http://img.blog.csdn.net/20160607204306776)

***需求:***  
　　当用户登录成功后，把该用户名字显示在登录成功页面;  

***解决方法***

* 使用 java 基础 static，专门用一个类来存储静态数据

* 使用 sendRedirect()，在url上加上需要传递的参数

<pre>
response.sendRedirect("/UsersManager/MainFrame?uname="+username+"&pwd="+password);

说明:
基本格式:
response.sendRedirect("/Context/servlet的?参数名=参数值&参数名=参数值...");
</pre>

* 使用session传递

　　session既可以传递字符串，也可以传递对象

	A.传递字符串
	放入session   
	    request.getSession.setAttribute("loginUser",username); 
	取出session	 
	    在JSP中通过session取出 request.getSession.getAttribute("loginUser");
	    
	B．传递对象
	User user= new User();
	user.setName("xiaoli");
	user.setPassWord(“123”);
	
	放入session   
	    request.getSession.setAttribute("userObj",userObj); 
	取出session	 
	    User user=(User)request.getSession.getAttribute("userObj");


## 中文乱码处理

　　发生中文乱码有三种情况，我们应当尽量使用post方式提交

### ①表单form

![](http://i.imgur.com/zplY6DJ.png)

* (1)表单以post方式提交<br/>
　　浏览器把请求发送给web服务器[utf-8]，web服务器以ISO-8859-1编码方式进行接收，产生乱码，之后进行传递也都是乱码。  
　　在接收参数时，采用正确的编码，即可解决问题，在服务器端设置成浏览器端的编码方式。

<pre>request.setCharacterEncoding("utf-8"); //gbk gb2312 big5</pre>

* (2)表单以get方式提交<br/>
　　请求内容是以请求行URL进行提交，而不是用请求体，所以使用setCharacterEncoding无效

写一个工具类:

	package com.hsp.utils;
	public class MyTools {
		public static String getNewString(String str) {
			String newString="";
			try {
				newString=new String(str.getBytes("iso-8859-1"),"utf-8");
			} catch (Exception e) {
				e.printStackTrace();
				// 把iso-8859-1 转换成 utf-8
			} 
			return newString;
		}
	}

### ②超链接
　　数据以get方式进行传递，该方法和get处理方法一样.

	<a href=”http://www.sohu.com?name=函数后”>测试</a>


### ③sendRedirect() 发生乱码
　　客户端会重新发送一个http请求，该方法和get处理方法一样

	response.sendRedirect("servlet地址?username=顺平"); 

### 版本低导致的乱码 
　　特别说明，如果你的浏览器是 ie6 或以下版本，则我们的 ② 和 ③中情况会出现乱码(当中文是奇数的时候)
解决方法是 ：

	String info=java.net.URLEncoder.encode("你好吗.jpg", "utf-8");
	<a href="http://www.sohu.com?name="+ info >测试</a>
	response.sendRedirect("servlet地址?username=" + info);

### ★★★★返回浏览器显示乱码 
　　在服务端是中文，在response的时候，也要考虑浏览器显示是否正确,一般我们通过

	　response.setContentType("text/html;charset=utf-8");

### 下载提示框中文乱码
　　补充一个知识点: 当我们下载文件的时候，可能提示框是中文乱码 

	String temp=java.net.URLEncoder.encode("传奇.mp3","utf-8");
	response.setHeader("Content-Disposition","attachment; filename="+temp);


## HttpServletRequest对象的详解
　　该对象表示浏览器的请求(http请求), 当web服务器得到该请求后，会把请求信息封装成一个HttpServletRequest 对象

* getRequestURL方法返回客户端发出请求时的完整URL。
* getRequestURI方法返回请求行中的资源名部分。
* getQueryString方法返回请求行中的参数部分(参数名+值)。该函数可以获取请求部分的数据，比如
`http://localhost/web名?username=abc&pwd=123request.getQueryString();`
就会得到  username=abc&pwd=123
* getRemoteAddr方法返回发出请求的客户机的IP地址
* getRemoteHost方法返回发出请求的客户机的完整主机名，如果该客户机没有在dns注册，则返回ip地址
* getRemotePort方法返回客户机所使用的网络端口号，客户机的端口号是随机选择的，web服务器的端口号是一定的
* getLocalPort方法返回web服务器所使用的网络端口号
* getLocalAddr方法返回WEB服务器的IP地址。
* getLocalName方法返回WEB服务器的主机名

## url 和 uri 的区别

    比如：
    	Url=http://localhost:8088/servletPort3/GetinfoServlet 完整的请求
    	Uri=/servletPort3/GetinfoServlet web应用的名称+资源的名称

## 请求转发getRequestDispatcher

	#请求转发
	requeset.getRequestDispatcher(资源地址).forward(request,response);
	#可以在request的域对象中存储数据，request中的attribute在一次请求中有效
	request.setAttribute("username",username);

　　资源地址：不需要项目名。因为它只是在WEB服务器内部转发。
　　
![这里写图片描述](http://img.blog.csdn.net/20160608153021012)

　　Servlet接收到数据后，可以把数据放入到request域对象，Request中的Attribute在一次请求有效。  
　　一次请求：浏览器发送一次http请求到接收到响应成为一次http请求，只要没有停止，也没有回到浏览器重定向，就算一次  

***请求转发的的(uml)图***

![这里写图片描述](http://img.blog.csdn.net/20160608154011510)

1. 使用forward不能转发到该web应用外的url
2. 因为 forward 是发生在web服务器，所以 Servlet1 和 Servlet 2使用的是用一个request 和response.
3. 使用sendRedirect() 方法不能通过request.setAttribute() 把 属性传递给下一个Servlet

***面试题：请问sendRedirect和 forward的区别是什么？***  
答:  

	1. 叫法sendRedirect()请求重定向，转发forward()叫请求转发  
	2. 实际发生的位置不一样    
	　　sendRedirect发生在浏览器，由浏览器重新发出http请求  
	　　forward发生web服务器，请求在web服务器转发  
	3. 用法不一样    
	　　request.getRequestDispatcher(“/资源URI”).forward(request,response)
	　　response.sendRedirect(“/web应用/资源URI”);  
	4. 能够去URL范围不一样  
	　　sendRedirect可以去外边URL  
	　　forward只能去当前的WEB应用的资源

## 会话技术cookie

### 什么是会话

　　基本概念: 指用户开一个浏览器，访问一个网站,只要不关闭该浏览器，不管该用户点击多少个超链接，访问多少资源，直到用户关闭浏览器，整个这个过程我们称为一次会话。比如打电话


***为什么需要cookie技术(会话技术)***  

* 如何保存用户上次登录时间
* 如何显示用户浏览历史
* 如何把登录的用户名和密码电脑，下次登录，不需要重新输入

### cookie的原理图

![](http://i.imgur.com/ihpk6OJ.png)

### 创建cookie

	//创建cookie
	Cookie cookie = new Cookie("name","qm");
	//设置cookie的生命周期
	cookie.setMaxAge(3600);
	//把cookie会写给浏览器
	response.addCookie(cookie);

### 读取cookie

	Cookie[] cookies = request.getCookies();
    System.out.println(cookies.length);
    for(Cookie c : cookies){
        out.println(c.getName() + " " + c.getValue() + "<br/>");
    }

### cookie的小结★★★

* cookie 是在服务端创建
* cookie 是保存在浏览器这端
* cookie 的生命周期可以通过cookie.setMaxAge(2000);如果不设置setMaxAge则该cookie的生命周期当浏览器关闭时，就消亡
* <font color='red'>同一个浏览器***多个实例***共享cookie，若cookie没到期，关闭浏览器后还能共享该cookie。当然不同浏览器之间是不能共享cookie的，因为每个浏览器存放cookie的路径不一样(与session的区别，session在关闭浏览器后，不采取任何方式，无法访问到session)</font>
* 我们可以把cookie想成一张表  
![](http://i.imgur.com/jnEcC7g.png)
* 如果cookie重名就会替换存在的cookie值
* 一个web应用可以保存多个cookie,但保存在客户端浏览器下同一个cookie文本中
* 浏览器访问web应用时，会携带该web应用相关的cookie
* cookie存放的时候是以明文方式存放，因此安全较低，我们可以通过加密后保存[MD5算法见Java基础常用-MD5]

### 举例 - 保存上次登录时间★★★
 
	//先获取cookie
	// 假设我们 保存上次登录时间的cookie "lasttime" "2011-11-11 12:12:12";
	// 这里我们要考虑一个情况: 用户第一次登录 '您是第一次登录..'
	Cookie[] cookies = request.getCookies();
	boolean b = false;//假设没有lasttime cookie
	if (cookies != null) { //保证有cookie,取遍历
	    for (Cookie cookie : cookies) {
	        //取出名
	        String name = cookie.getName();
	        if ("lasttime".equals(name)) {
	            //显示
	            out.println("您上次登录时间是 " + cookie.getValue());
	            //更新时间
	            //把当前日期保存cookie
	            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	            String nowTime = simpleDateFormat.format(new java.util.Date());
	            cookie.setValue(nowTime);
	            cookie.setMaxAge(7 * 3600 * 24);//保存一周
	            response.addCookie(cookie);
	            b = true;
	            break;
	        }
	    }
	}
	
	if (!b) {
	    //没有找到
	    out.println("您是第一次登录..");
	    //把当前日期保存cookie
	    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	    String nowTime = simpleDateFormat.format(new java.util.Date());
	    Cookie cookie = new Cookie("lasttime", nowTime);
	    cookie.setMaxAge(7 * 3600 * 24);//保存一周
	    response.addCookie(cookie);
	}
 
### cookie的细节★★★
① 一个浏览器最多放入300cookie，一个web站点，最多20cookie，而且一个cookie大小限制子4k
② cookie生命周期的再说明:

* <font color = 'red'>cookie默认生命周期是会话级别，即浏览器关闭，cookie生命周期结束</font>
* 通过setMaxAge() 可以设置生命周期
	- setMaxAge(正数)，即多少秒后该cookie失效
	- setMaxAge(0)，删除该cookie
	- setMaxAge(负数)，相当于该cookie生命周期是会话级别
* cookie的生命周期，指的是累计时间，例如设置生命周期为30s，那么在30s之后cookie消亡（与session区别，session是发呆时间，即在一段时间内没有访问session就会消亡）

***案例 ：***

	//先得到该cookie
	Cookie cookies[]=request.getCookies();
	for(Cookie cookie: cookies){
		if(cookie.getName().equals("id")){
			System.out.println("id");
			//删除
			cookie.setMaxAge(0);
			response.addCookie(cookie);//一定带上这句话，否则不能删除
		}
	}

特别说明: 如果该web应用只有一个cookie ，则删除该cookie后，在浏览器的临时文件夹下没有该cookie文件，如果该web应用有多个cookie,则删除一个cookie后，文件还在，只是该cookie没有

③	cookie存放中文，怎么处理

进行URLEncoder

	存放:
	String val=java.net.URLEncoder.encode("顺平","utf-8");
	Cookie cookie=new Cookie("name",val);
	取出:
	String val=java.net.URLDecoder.decode(cookie.getValue(), "utf-8");
	out.println("name ="+val);

## 会话技术session（生成验证码）

### session有什么用?  

　　问题1: 如何实现在不同的页面，可以去查看信息(比如说购物车)，同时还要实现不同的用户看到的信息是自己.  
　　Session是服务端技术，可以为每一个用户的浏览器创建一个独享的session对象

### session工作原理图

 ![](http://i.imgur.com/FlOMxdb.png)

* session对象一行就代表一个属性，键值对  
* request.getSession()获得session，若第一次访问session会自动被创建，该浏览器第二次访问时将字节返回之前创建的session，不会创建新的。
* <font color='red'>一个浏览器关联一个session。</font>如果此时有新的浏览器2访问Servlet1，那么就会创建一个新的sesssion与浏览器2对应
* session默认的生命周期是30分钟
* session中使用setAttribute时使用相同的属性名，属性值会被替换

### session基本使用

	//访问session[当发现没有session时候，就会自动创建session]
    HttpSession session = request.getSession();
    //向session中添加属性
    session.setAttribute("name","姓名");
	//从session中得到某个属性
    String name = (String) session.getAttribute("name");
    out.println(name + "<br/>");
	//从session中删除某个属性
    session.removeAttribute("name");
    out.println((String) session.getAttribute("name") + "<br/>");

### session小结

① session是存在服务器的内存中  
② 一个用户浏览器，独享一个session域对象（不能浏览器会创建新的session）  
③ session中的属性的默认生命周期是30min，你可以通过web.xml来修改  
④ 3种session生命周期的设置

	（1）一个地方是 tomcat/conf/web.xml
		<session-config>
			<session-timeout>30</session-timeout>//表示30分钟的意思
		</session-config>
		对所有的web应用生效
	（2）另外一个地方，就是在单个web应用的下去修改 web.xml
		<session-config>
			<session-timeout>30</session-timeout>session精确到分钟,cookie精确到秒
		</session-config>
		如果发生冲突，则以自己的web应用优先级高
	（3）session.setMaxInactiveinterval(60) 六十秒为发呆时间，即在这六十秒内没有访问过session，则session中所有属性失效

![](http://i.imgur.com/LarH0Wk.png)

* session周期是发呆时间，如果我们设置session是10s，是指在10s内，没有访问过session，session属性失效，如果在9s时候，你访问session，session就会重新计时
* 如果重启tomcat，或者reload web应用，或者关机了，session失效
* 我们也可以通过函数，让session失效。invalidate()方法让所有属性失效，通常用于用户安全退出
* 如果你希望某个session属性失效，可以使用方法removeAttribute


⑤ session中可以存放多个属性  
⑥ session可以存放对象  
⑦ 如果session.setAttribute("name",val)，如果名字重复，则会替换该属性.

### session的更深入理解

***为什么服务器能够为不同的浏览器提供不同session？***  

　　图中浏览器A第一次访问Servlet1的时候，没有携带JSESSIONID，调用getSession()就会自动给你创建一个session，id为110，并创建cookie：JSESSIONID=110.  
　　第二次浏览器访问Servlet2时，携带cookie：JSESSIONID=110，说明session已经创建，getSession（）方法返回id=110的session，能够获得session中的数据  
　　同理，浏览器B访问servlet1时会新建一个新的session，id=119  
　　每个浏览器独享一个session对象  
　　<font color='red'>session自动返回的cookie不会被写入文件，所以不同浏览器访问servlet会获取不同的session</font>


![](http://i.imgur.com/Klh1fzm.png)

### 生成验证码案例

	public class check_code extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        // 7.禁止浏览器缓存随机图片
	        response.setDateHeader("Expires", -1);
	        response.setHeader("Cache-Control", "no-cache");
	        response.setHeader("Pragma", "no-cache");
	        // 6.通知客户机以图片方式打开发送过去的数据
	        response.setHeader("Content-Type", "image/jpeg");
	        // 1.在内存中创建一副图片
	        BufferedImage image = new BufferedImage(60,30,BufferedImage.TYPE_INT_RGB);
	        // 2.向图片上写数据
	        Graphics g = image.getGraphics();
	        // 设背景色
	        g.setColor(Color.BLACK);
	        g.fillRect(0, 0, 60, 30);
	        // 3.设置写入数据的颜色和字体
	        g.setColor(Color.RED);
	        g.setFont(new Font(null, Font.BOLD, 20));
	        // 4.向图片上写数据
	        String num = makeNum();
	        //这句话就是把随机生成的数值，保存到session
	        request.getSession().setAttribute("check_code", num); //通过session就可以直接去到随即生成的验证码了
	        g.drawString(num, 5, 22);
	        // 5.把写好数据的图片输出给浏览器
	        ImageIO.write(image, "jpg", response.getOutputStream());
	    }
	    //该函数时随机生成4位数字
	    public String makeNum() {
	        Random r = new Random();
	        //9999999 可以生成7位
	        String num = r.nextInt(9999) + "";
	        StringBuffer sb = new StringBuffer();
	        //如果不够4位，前面补零
	        for (int i = 0; i < 4 - num.length(); i++) {
	            sb.append("0");
	        }
	        num = sb.toString() + num;
	        return num;
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        this.doPost(request, response);
	    }
	}

### IE上购买商品后关闭，再打开IE，要求上次的商品还在

***分析***

* 设session生命周期为30min,该session不会随浏览器的关闭而自动销毁。而会到30min后，才会被服务器销毁
* 关闭浏览器后，再打开。由于服务器端传回来的cookie：JSESSIONID没有设置生命周期，那么在浏览器关闭后cookie的生命周期结束。下次打开浏览器时，没有携带cookie：JSESSIONID，访问Servlet又会创建一个新的session
* 我们使用代码来实现该功能(session + cookie结合使用)

***分析实现的思路:***

	public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		response.setContentType("text/html;charset=utf-8");
		PrintWriter out = response.getWriter();
		//创建一个session，并放入一个属性
		HttpSession session = request.getSession();
		session.setAttribute("name", "xxx");
		out.println("创一个session并放入姓名属性");
		//把session_id保存在cookie，cookie名字必须按照规范命名，必须大写JSESSIONID
		Cookie cookie = new Cookie("JSESSIONID", session.getId());
		////如果不设置时间生命周期，cookie在浏览器关闭后就消亡
		cookie.setMaxAge(60*30);
		response.addCookie(cookie);
	}

### ie禁用cookie后使用session的方法

***简易购物车的实例***

![](http://i.imgur.com/mqreaNv.png)

***思路***

	当用户点击购买商品时，我们把该商品保存到session中，该session的结构是:
	name 			val
	mybookds    hashMap对象
	而hashmap的结构是	
	key 	val
	书号   书对象.

若禁用cookie后，每次访问Servlet不携带cookie数据，创建一个新的session，购物车商品不能保存

***解决方法***

URL重写

* response.encodeURL(java.lang.String url)用于对表单action和超链接的url地址进行重写
* response.encodeRedirectURL(java.lang.String url)用于对sendRedirect方法后的url地址进行重写

Servlet.java

	request.getSession();//必须访问一下sesion
	String url = response.encodeURL("/ServletStudy/cl");
	out.println("<form method = 'post' action = '"+url+"'>");
	out.println("<input type = 'submit'/>");
	out.println("</form>");

cl.java

	HttpSession session = request.getSession();
	ArrayList<String> array = (ArrayList<String>) session.getAttribute("array");
	if(null == array){
	    array = new ArrayList<String>();
	    session.setAttribute("array",array);
	}
	SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	String time = simpleDateFormat.format(new java.util.Date());
	array.add(time);
	
	for(String s : array){
	    out.println(s + "<br/>");
	}
	String url = response.encodeURL("/ServletStudy/Servlet");
	out.println("<a href = '"+url+"'>返回</a>");
	
## cookie vs session

### 存在的位置

* cookie存在客户端的临时文件夹
* session存在服务器的内存中，一个sessio与对象为一个用户浏览器服务

### 安全性

* cookie是以明文方式存放在客户端，安全性弱，可以通过加密md5再存放
* session是存放在服务端的内存中，安全性好

### 网络传输量

* cookie会传递信息，给服务端
* session的属性值不会给客户端

### 生命周期

* cookie的生命周期是累计时间，即如果我们给cookie设置setMaxAge（30），则cookie在30s后失效
* session额生命周期是间隔时间，如果我们设置session 20min，指在20min内，如果没有访问session，则session失效（指的是session属性失效），在关闭tomcat,reload web应用，时间到，invalidate也会让session失效

### 使用原则

　　因为session会占用服务器的内存，因此不要向session存放过多的对象，会影响性能

## 过滤器 Filter

* 客户端请求request在**抵达Servlet之前**、服务器响应response在**从Servlet抵达客户端浏览器**会经过过滤器，过滤器用于在Servlet之外**对request或者response进行修改**
* Filter体现的是设计模式中的Filter模式

### 过滤器链 FilterChain

一个过滤器链包括多个Filter，客户端请求request在**抵达Servlet之前**会经过FilterChain里的所有Filter，服务器响应response在**从Servlet抵达客户端浏览器**之前也会经过FilterChain里的所有Filter

![](http://i.imgur.com/5S0gzS0.png)

### 防盗链Filter

	public class ImageRedirectFilter implements Filter {
	
		public void init(FilterConfig config) throws ServletException {
		}
	
		public void doFilter(ServletRequest req, ServletResponse res,
				FilterChain chain) throws IOException, ServletException {
	
			HttpServletRequest request = (HttpServletRequest) req;
			HttpServletResponse response = (HttpServletResponse) res;
	
			// 禁止缓存
			response.setHeader("Cache-Control", "no-store");
			response.setHeader("Pragrma", "no-cache");
			response.setDateHeader("Expires", 0);
	
			// 链接来源地址
			String referer = request.getHeader("referer");
	
			if (referer == null || !referer.contains(request.getServerName())) {
	
				/**
				 * 如果 链接地址来自其他网站，则返回错误图片
				 */
				request.getRequestDispatcher("/error.gif").forward(request,
						response);
	
			} else {
	
				/**
				 * 图片正常显示
				 */
				chain.doFilter(request, response);
			}
	
		}
	
		public void destroy() {
		}
	}

配置：

	<filter>
		<filter-name>imageRedirectFilter</filter-name>
		<filter-class>
			com.helloweenvsfei.filter.ImageRedirectFilter
		</filter-class>
	</filter>

### 字符编码 Filter

#### 自定义

	public class CharacterEncodingFilter implements Filter {

		private String characterEncoding;
		private boolean enabled;
	
		@Override
		public void init(FilterConfig config) throws ServletException {
	
			characterEncoding = config.getInitParameter("characterEncoding");
	
			enabled = "true".equalsIgnoreCase(characterEncoding.trim())
					|| "1".equalsIgnoreCase(characterEncoding.trim());
		}
	
		@Override
		public void doFilter(ServletRequest request, ServletResponse response,
				FilterChain chain) throws IOException, ServletException {
	
			if (enabled || characterEncoding != null) {
				request.setCharacterEncoding(characterEncoding);
				response.setCharacterEncoding(characterEncoding);
			}
	
			chain.doFilter(request, response);
		}
	
		@Override
		public void destroy() {
			characterEncoding = null;
		}
	}

配置：

	<filter>
		<filter-name>characterEncodingFilter</filter-name>
		<filter-class>
			com.helloweenvsfei.filter.CharacterEncodingFilter
		</filter-class>
		<init-param>
			<param-name>characterEncoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>enable</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>

#### Spring mvc自带

	<!-- 编码过滤器，UTF8编码，对POST有效 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

### 异常捕捉 Filter

	public class ExceptionHandlerFilter implements Filter {
	
		public void destroy() {}
	
		public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
			try {
				chain.doFilter(request, response);
			} catch (Exception e) {
				Throwable rootCause = e;
				while (rootCause.getCause() != null) {
					rootCause = rootCause.getCause();
				}
				String message = rootCause.getMessage();
				message = message == null ? "异常：" + rootCause.getClass().getName()
						: message;
				request.setAttribute("message", message);
				request.setAttribute("e", e);
	
				if (rootCause instanceof AccountException) {
					request.getRequestDispatcher("/accountException.jsp").forward(
							request, response);
				} else if (rootCause instanceof BusinessException) {
					request.getRequestDispatcher("/businessException.jsp").forward(
							request, response);
				} else {
					request.getRequestDispatcher("/exception.jsp").forward(request,
							response);
				}
			}
		}
	
		public void init(FilterConfig arg0) throws ServletException {
		}
	}

### 内容替换 Filter

需求：有时候需要对网站的内容进行控制，防止输出非法内容或者敏感内容
解决方案：

* 方案一：在Servlet里输出到客户端时进行内容替换，这种方案需要对每个Servlet
都进行替换，工作量大，业务耦合比较严重
* 方案二：在Servlet将内容输出到response时，response将内容缓存起来，
在Filter中进行替换，然后再输出到客户端浏览器。但是默认的response并不能严格的
缓存输出内容，因此需要<font color='red'>***自定义一个具有缓存功能的response***</font>

要点：自定义的一个response只是一个伪装的response。Servlet会通过它输出内容到客户端，但是它内部只是将内容缓存起来了（<font color='red'>***使用自己创建的PrintWriter***</font>），并没有真正输出到客户端。最终输出到客户端还是通过原来的resonse完成的

框图：

![](http://i.imgur.com/OirpKFm.png)

具体操作：

1、通过扩展HttpServletResponseWrapper类来<font color='red'>***实现自定义的response***</font>，该类覆盖了getWriter()方法，当Servlet使用该response对象调用getWriter()莱输出内容时，内容将被输出到CharArrayWriter对象中，达到缓存的效果

<pre>
public class HttpCharacterResponseWrapper extends HttpServletResponseWrapper {

	<font color='red'>private CharArrayWriter charArrayWriter = new CharArrayWriter();</font>

	public HttpCharacterResponseWrapper(HttpServletResponse response) {
		super(response);
	}

	<font color='red'>@Override
	public PrintWriter getWriter() throws IOException {
		return new PrintWriter(charArrayWriter);
	}</font>

	public CharArrayWriter getCharArrayWriter() {
		return charArrayWriter;
	}
}
</pre>

2、Filter将自定义的response传进Servlet中

<pre>
public class OutputReplaceFilter implements Filter {

	private Properties pp = new Properties();

	public void init(FilterConfig config) throws ServletException {
		String file = config.getInitParameter("file");
		String realPath = config.getServletContext().getRealPath(file);
		try {
			pp.load(new FileInputStream(realPath));
		} catch (IOException e) {
		}
	}

	public void doFilter(ServletRequest req, ServletResponse res,
			FilterChain chain) throws IOException, ServletException {

		<font color='red'>// 自定义的 response
		HttpCharacterResponseWrapper response = new HttpCharacterResponseWrapper(
				(HttpServletResponse) res);

		// 提交给 Servlet 或者下一个 Filter
		chain.doFilter(req, response);</font>

		// 得到缓存在自定义 response 中的输出内容
		String output = response.getCharArrayWriter().toString();

		// 修改，替换
		for (Object obj : pp.keySet()) {
			String key = (String) obj;
			output = output.replace(key, pp.getProperty(key));
		}
		
		<font color='red'>// 输出
		PrintWriter out = res.getWriter();
		out.write(output);</font>
		out.println("<!-- Generated at " + new java.util.Date() + " -->");
	}

	public void destroy() {
	}
}
</pre>

### GZIP 压缩 Filter

Servlet中操作输入输出流：

<pre>
String path = request.getSession().getServletContext().getRealPath("/error.jpg");
InputStream inputStream = new FileInputStream(path);
int len = -1;
byte[] buffer = new byte[1024];
<font color='red'>OutputStream outputStream = response.getOutputStream();</font>
while((len = inputStream.read(buffer)) != -1){
    <font color='red'>outputStream.write(buffer,0,len);</font>
}
<font color='red'>outputStream.close();</font>
inputStream.close();
</pre>

要点：
1、替换response，并且替换的response中重写response中的getOutputStream()方法[也需要重写getWrite()，因为除了压缩二进制文件，还要压缩文本文件]
2、重写ServletOutputStream，当调用write方法时，使用JDK自带的GZIPOutputStream类进行数据压缩★★★★★★★★（核心）

总体操作框图：

![](http://i.imgur.com/V7Ec9aU.png)

<pre>
①自定义的GZipResponseWrapper替换tomcat传入的respone
    <font color='red'>GZipResponseWrapper gzipResponse = new GZipResponseWrapper(response);</font>
    chain.doFilter(request, gzipResponse);
②Servlet中调用response中的getOutputStream()获得输出流，数据写入缓存，调用close方法进行压缩
    OutputStream outputStream = response.<font color='red'>getOutputStream();</font>
    while((len = inputStream.read(buffer)) != -1){
        outputStream.write(buffer,0,len);
    }
    <font color='red'>outputStream.close();</font>
    inputStream.close();
③输出压缩数据
    gzipResponse.finishResponse();
</pre>

GZIPOutputStream类API：

	void finish() 
	      完成将压缩数据写入输出流的操作，无需关闭底层流。 
	void write(byte[] buf, int off, int len) 
	      将字节数组写入压缩输出流。 

1、GZipFilter，如果客户端支持GZip自动解压，则进行GZIP压缩，否则不压缩

<pre>
public class GZipFilter implements Filter {

	public void destroy() {
	}

	public void doFilter(ServletRequest req, ServletResponse res,
			FilterChain chain) throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		String acceptEncoding = request.getHeader("Accept-Encoding");
		System.out.println("Accept-Encoding: " + acceptEncoding);

		if (acceptEncoding != null
				&& acceptEncoding.toLowerCase().indexOf("gzip") != -1) {

			// 如果客户浏览器支持 GZIP 格式, 则使用 GZIP 压缩数据
			<font color='red'>GZipResponseWrapper gzipResponse = new GZipResponseWrapper(response);
			chain.doFilter(request, gzipResponse);</font>

			<font color='red'>/*
			* 输出压缩数据，一般由用户在调用outputStream.close()自己关闭输出流，
			* 此处调用是为了防止用户忘记
			*/
			gzipResponse.finishResponse();</font>

		} else {
			// 否则, 不压缩
			chain.doFilter(request, response);
		}
	}

	public void init(FilterConfig arg0) throws ServletException {
	}
}
</pre>

2、GZipResponseWrapper伪装response，GZipOutputStream为它的成员变量，Servlet中的输出就是对GZipOutputStream的操作

* GZipResponseWrapper为自定义的response类，内部将对输出的内容进行GZIP压缩。它继承HttpServletResponseWrapper，也是一个伪装的response，不真正输出内容到客户端
* 由于该response要处理二进制内容，又要处理字符内容，因此需要覆盖getOutputStream和getWriter
* **GZipResponseWrapper中的方法是实现对GZipOutputStream的操作**


<pre>
public class GZipResponseWrapper extends HttpServletResponseWrapper {

	<font color='red'>/*
	* 传入默认的response，保存起来作为成员变量
	*/</font>
	private HttpServletResponse response;

	<font color='red'>/*
	* 可以不保存，如果用户自己关闭输入流，则无需调用finishResponse方法来关闭文本或二进制输出流。
	* 但是为了增加可靠性，在response抵达客户端浏览器前进行关闭流操作
	*/</font>
	// 自定义的 outputStream, 执行close()的时候对数据压缩，并输出
	private GZipOutputStream gzipOutputStream;
	// 自定义 printWriter，将内容输出到 GZipOutputStream 中
	private PrintWriter writer;

	public GZipResponseWrapper(HttpServletResponse response) throws IOException {
		super(response);
		this.response = response;
	}

	public ServletOutputStream getOutputStream() throws IOException {
		if (gzipOutputStream == null)
			<font color='red'>gzipOutputStream = new GZipOutputStream(response);</font>
		return gzipOutputStream;
	}

	public PrintWriter getWriter() throws IOException {
		if (writer == null)
			<font color='red'>writer = new PrintWriter(new OutputStreamWriter(
					new GZipOutputStream(response), "UTF-8"));</font>
		return writer;
	}

	// 压缩后数据长度会发生变化 因此将该方法内容置空
	public void setContentLength(int contentLength) {
	}

	public void flushBuffer() throws IOException {
		gzipOutputStream.flush();
	}

	<font color='red'>public void finishResponse() throws IOException {
		if (gzipOutputStream != null)
			gzipOutputStream.close();
		if (writer != null)
			writer.close();
	}</font>
}
</pre>

3、自定义GZipOutputStream类，继承ServletOutputStream，使用JDK自带的GZIP压缩类将数据缓存起来，之后调用finish函数进行数据压缩，并输出到客户端浏览器

<pre>
public class GZipOutputStream extends ServletOutputStream {

	private HttpServletResponse response;

	// JDK 自带的压缩数据的类
	private GZIPOutputStream gzipOutputStream;

	// 将压缩后的数据存放到 ByteArrayOutputStream 对象中
	private ByteArrayOutputStream byteArrayOutputStream;

	<font color='red'>/*
	* GZipResponseWrapper中调用构造函数，创建实例，数据缓存在ByteArrayOutputStream
	*/</font>
	public GZipOutputStream(HttpServletResponse response) throws IOException {
		this.response = response;
		byteArrayOutputStream = new ByteArrayOutputStream();
		gzipOutputStream = new GZIPOutputStream(byteArrayOutputStream);
	}

	<font color='red'>/*
	* Servlet中调用write方法，将数据写入缓存
	*/</font>
	public void write(int b) throws IOException {
		gzipOutputStream.write(b);
	}

	<font color='red'>/*
	* Servlet中调用close方法，并不是直接关闭输入流，而是先执行压缩操作，然后调用浏览器本身的输出流，在写入到客户端浏览器
	*/</font>
	public void close() throws IOException {

		// 压缩完毕 一定要调用该方法
		gzipOutputStream.finish();

		// 将压缩后的数据输出到客户端
		byte[] content = byteArrayOutputStream.toByteArray();

		// 设定压缩方式为 GZIP, 客户端浏览器会自动将数据解压
		response.addHeader("Content-Encoding", "gzip");
		response.addHeader("Content-Length", Integer.toString(content.length));

		// 输出
		ServletOutputStream out = response.getOutputStream();
		out.write(content);
		out.close();
	}

	public void flush() throws IOException {
		gzipOutputStream.flush();
	}

	public void write(byte[] b, int off, int len) throws IOException {
		gzipOutputStream.write(b, off, len);
	}

	public void write(byte[] b) throws IOException {
		gzipOutputStream.write(b);
	}
}
</pre>

### 图像水印 Filter

使用Filter在图像上动态打上一个水印LOGO，工作原理与GZIP压缩类似，先把图像数据缓存起来，然后对图像进行水印处理后输出到客户端浏览器

图像水印Filter需要自定义response与servletOutputStream

1、WaterMarkFilter，在Filter初始化参数里设置水印图片文件路径

<pre>
public class WaterMarkFilter implements Filter {

	// 水印图片，配置在初始化参数中
	private String waterMarkFile;

	public void init(FilterConfig config) throws ServletException {
		String file = config.getInitParameter("waterMarkFile");
		waterMarkFile = config.getServletContext().getRealPath(file);
	}

	public void doFilter(ServletRequest req, ServletResponse res,
			FilterChain chain) throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		// 自定义的response
		WaterMarkResponseWrapper waterMarkRes = new WaterMarkResponseWrapper(response, waterMarkFile);

		chain.doFilter(request, waterMarkRes);

		// 打水印，输出到客户端浏览器
		waterMarkRes.finishResponse();
	}

	public void destroy() {
	}

}
</pre>

2、WaterMarkResponseWrapper继承HttpServletResponseWrapper，改写了getOutputStream方法，finishResponse方法将缓存的数据进行水印处理，并输出到客户端浏览器。水印处理的代码被封装到ImageUtil类的静态方法waterMark()中

<pre>
public class WaterMarkResponseWrapper extends HttpServletResponseWrapper {

	// 水印图片位置
	private String waterMarkFile;

	// 原response
	private HttpServletResponse response;

	// 自定义servletOutputStream，用于缓冲图像数据
	private WaterMarkOutputStream waterMarkOutputStream;

	public WaterMarkResponseWrapper(HttpServletResponse response,
			String waterMarkFile) throws IOException {
		super(response);
		this.response = response;
		this.waterMarkFile = waterMarkFile;
		this.waterMarkOutputStream = new WaterMarkOutputStream();
	}

	// 覆盖getOutputStream()，返回自定义的waterMarkOutputStream
	public ServletOutputStream getOutputStream() throws IOException {
		return waterMarkOutputStream;
	}

	public void flushBuffer() throws IOException {
		waterMarkOutputStream.flush();
	}

	// 将图像数据打水印，并输出到客户端浏览器
	public void finishResponse() throws IOException {

		// 原图片数据
		byte[] imageData = waterMarkOutputStream.getByteArrayOutputStream()
				.toByteArray();

		// 打水印后的图片数据
		byte[] image = ImageUtil.waterMark(imageData, waterMarkFile);

		// 将图像输出到浏览器
		response.setContentLength(image.length);
		response.getOutputStream().write(image);

		waterMarkOutputStream.close();
	}
}
</pre>

3、WaterMarkOutputStream类将图像数据缓存起来

<pre>
public class WaterMarkOutputStream extends ServletOutputStream {

	// 缓冲图片数据
	private ByteArrayOutputStream byteArrayOutputStream;

	public WaterMarkOutputStream() throws IOException {
		byteArrayOutputStream = new ByteArrayOutputStream();
	}

	public void write(int b) throws IOException {
		byteArrayOutputStream.write(b);
	}

	public void close() throws IOException {
		byteArrayOutputStream.close();
	}

	public void flush() throws IOException {
		byteArrayOutputStream.flush();
	}

	public void write(byte[] b, int off, int len) throws IOException {
		byteArrayOutputStream.write(b, off, len);
	}

	public void write(byte[] b) throws IOException {
		byteArrayOutputStream.write(b);
	}

	public ByteArrayOutputStream getByteArrayOutputStream() {
		return byteArrayOutputStream;
	}

}
</pre>

4、ImageUtil类使用JDK的图像处理类完成添加水印的操作

<pre>
public class ImageUtil {

	/**
	 * 
	 * @param imageData
	 *            JPG 图像文件
	 * @param waterMarkFile
	 *            水印图片
	 * @return 加水印后的图像数据
	 * @throws IOException
	 */
	public static byte[] waterMark(byte[] imageData, String waterMarkFile)
			throws IOException {

		// 水印图片的右边距 下边距
		int paddingRight = 10;
		int paddingBottom = 10;

		// 原始图像
		Image image = new ImageIcon(imageData).getImage();
		int imageWidth = image.getWidth(null);
		int imageHeight = image.getHeight(null);

		// 水印图片
		Image waterMark = ImageIO.read(new File(waterMarkFile));
		int waterMarkWidth = waterMark.getWidth(null);
		int waterMarkHeight = waterMark.getHeight(null);

		// 如果图片尺寸过小，则不打水印，直接返回
		if (imageWidth < waterMarkWidth + 2 * paddingRight
				|| imageHeight < waterMarkHeight + 2 * paddingBottom) {
			return imageData;
		}
		BufferedImage bufferedImage = new BufferedImage(imageWidth,
				imageHeight, BufferedImage.TYPE_INT_RGB);

		Graphics g = bufferedImage.createGraphics();

		// 绘制原始图像
		g.drawImage(image, 0, 0, imageWidth, imageHeight, null);
		// 绘制水印图片
		g.drawImage(waterMark, imageWidth - waterMarkWidth - paddingRight,
				imageHeight - waterMarkHeight - paddingBottom, waterMarkWidth,
				waterMarkHeight, null);
		g.dispose();

		// 转成JPEG格式
		ByteArrayOutputStream out = new ByteArrayOutputStream();
		JPEGImageEncoder encoder = JPEGCodec.createJPEGEncoder(out);
		encoder.encode(bufferedImage);
		byte[] data = out.toByteArray();
		out.close();
		return data;
	}
}
</pre>

5、配置文件

	<filter>
		<filter-name>imageRedirectFilter</filter-name>
		<filter-class>
			com.helloweenvsfei.filter.ImageRedirectFilter
		</filter-class>
	</filter>

	<filter>
		<filter-name>waterMarkFilter</filter-name>
		<filter-class>
			com.helloweenvsfei.watermark.WaterMarkFilter
		</filter-class>
		<init-param>
			<param-name>waterMarkFile</param-name>
			<param-value>/WEB-INF/logo.png</param-value>
		</init-param>
	</filter>

### 内容替换、GZIP、图像水印 Filter 总结★★★★★★

以图像水印 Filter 为例

1、过滤器中自定义response包裹类，将原生response作为参数传入。自定义的response替换原生response传入Servlet；response返回客户端之前，调用新定义的finishResponse方法，输出到客户端浏览器

	// 自定义的response
	WaterMarkResponseWrapper waterMarkRes = new WaterMarkResponseWrapper(response, waterMarkFile);
	
	chain.doFilter(request, waterMarkRes);
	
	// 打水印，输出到客户端浏览器
	waterMarkRes.finishResponse();

2、自定义response包裹类有一个存放原生response的成员变量，一个自定义的输出流，起到缓存数据的功能。重写getOutputStream()方法或getWriter()方法，返回自定义输出流，添加finishResponse方法，将缓存中的数据进行处理，输出到客户端浏览器

3、重写ServletOutputStream类对数据进行，对数据缓存

### 缓存 Filter

***对于访问量比较大的网站（淘宝），反复地查询数据库要消耗很多时间***。如果第一次访问某页面查询了数据库，那么就可以把页面的内容缓存起来，下一次访问的时候直接返回缓存结果就行。使用缓存能将数据库读写次数较少都最少，从而提高服务器的响应速度

缓存Filter的工作流程：
1、截获浏览器提交的request
2、如果request为POST方式，则不进行缓存
3、**如果request为GET方式，且请求的页面有缓存并且缓存没有过期**，则直接返回缓存结果，这样就避免读取数据库
4、如果没有缓存或者缓存已过期，则重新请求Servlet，将Servlet返回的内容缓存并输出到客户端浏览器

使用缓存注意点：

1、缓存Filter不易用于数据会实时变化的数据，如报表、股票等，它适用于数据变化不大，但是访问次数多的内容，如论坛、博客、新闻等
2、缓存Filter不能用于POST方式提交数据，如登录、发表文章等
3、**当缓存更新后，要更新缓存，或者直接将缓存删掉**
4、被缓存的内容不能依赖于Session，而要依赖于Cookie。即要使用Cookie来记录客户身份而不要使用Session，并且**无论客户身份是管理员还是普通浏览者，Servlet输出内容都是一样的，只能在浏览器使用js根据cookie来决定显示什么内容**。但是注意：由于菜单的显示与否并没有在服务器端进行权限检查，<font color='red'>***因此当客户单击链接操作的时候，一定要做权限检查，否则会引发安全问题***</font>

框图：

![](http://i.imgur.com/S75GBO8.png)

①如果存在缓存文件，直接缓存文件中读取数据并输出到客户端浏览器，不进入Servlet
②没有缓存或缓存过期，在Servlet将输出的内容缓存起来
③缓存Filter将缓存内容写入到缓存文件，并读取缓存文件将数据输出

程序流程图：

![](http://i.imgur.com/SSL3f5N.png)

1、缓存Filter

<pre>
public class CacheFilter implements Filter {

	private ServletContext servletContext;

	// 缓存文件夹，使用Tomcat工作目录
	private File temporalDir;

	// 缓存时间，配置在Filter初始化参数中
	private long cacheTime = Long.MAX_VALUE;

	public void init(FilterConfig config) throws ServletException {
		temporalDir = (File) config.getServletContext().getAttribute(
				"javax.servlet.context.tempdir");
		servletContext = config.getServletContext();
		cacheTime = new Long(config.getInitParameter("cacheTime"));
	}

	public void doFilter(ServletRequest req, ServletResponse res,
			FilterChain chain) throws IOException, ServletException {

		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		<font color='red'>// 如果为 POST, 则不经过缓存</font>
		if ("POST".equals(request.getMethod())) {
			chain.doFilter(request, response);
			return;
		}

		<font color='red'>// 请求的 URI，忽略应用程序名称</font>
		String uri = request.getRequestURI();
		if (uri == null)
			uri = "";
		uri = uri.replace(request.getContextPath() + "/", "");
		uri = uri.trim().length() == 0 ? "index.jsp" : uri;
		uri = request.getQueryString() == null ? uri : (uri + "?" + request
				.getQueryString());

		// 对应的缓存文件
		File cacheFile = new File(temporalDir, URLEncoder.encode(uri, "UTF-8"));
		System.out.println(cacheFile);

		// 如果缓存文件不存在 或者已经超出缓存时间 则请求 Servlet
		if (!cacheFile.exists()
				|| cacheFile.length() == 0
				|| cacheFile.lastModified() < System.currentTimeMillis()
						- cacheTime) {

			CacheResponseWrapper cacheResponse = new CacheResponseWrapper(
					response);

			chain.doFilter(request, cacheResponse);

			// 将内容写入缓存文件
			char[] content = cacheResponse.getCacheWriter().toCharArray();

			temporalDir.mkdirs();<font color='red'>//递归创建文件夹</font>
			cacheFile.createNewFile();<font color='red'>//创建文件</font>

			Writer writer = new OutputStreamWriter(new FileOutputStream(
					cacheFile), "UTF-8");
			writer.write(content);
			writer.close();
		}

		// 请求的ContentType
		String mimeType = servletContext.getMimeType(request.getRequestURI());
		response.setContentType(mimeType);

		// 读取缓存文件的内容，写入客户端浏览器
		Reader ins = new InputStreamReader(new FileInputStream(cacheFile),
				"UTF-8");
		StringBuffer buffer = new StringBuffer();
		char[] cbuf = new char[1024];
		int len;
		while ((len = ins.read(cbuf)) > -1) {
			buffer.append(cbuf, 0, len);
		}
		ins.close();
		// 输出到客户端
		response.getWriter().write(buffer.toString());
	}

	public void destroy() {
	}
}
</pre>

2、CacheResponseWrapper强Servlet中输出的内容缓存起来，然后被缓存Filter写入到缓存文件中。本缓存Filter只缓存字符类网页，因此只覆盖了getWriter()方法

	public class CacheResponseWrapper extends HttpServletResponseWrapper {
	
		// 缓存字符类输出
		private CharArrayWriter cacheWriter = new CharArrayWriter();
	
		public CacheResponseWrapper(HttpServletResponse response)
				throws IOException {
			super(response);
		}
	
		@Override
		public PrintWriter getWriter() throws IOException {
			return new PrintWriter(cacheWriter);
		}
	
		@Override
		public void flushBuffer() throws IOException {
			cacheWriter.flush();
		}
	
		public void finishResponse() throws IOException {
			cacheWriter.close();
		}
	
		public CharArrayWriter getCacheWriter() {
			return cacheWriter;
		}
	
		public void setCacheWriter(CharArrayWriter cacheWriter) {
			this.cacheWriter = cacheWriter;
		}
	}

3、配置

	<filter>
		<filter-name>cacheFilter</filter-name>
		<filter-class>
			com.helloweenvsfei.cache.CacheFilter
		</filter-class>
		<init-param>
			<param-name>cache</param-name>
			<param-value>true</param-value>
		</init-param>
		<init-param>
			<param-name>cacheTime</param-name>
			<param-value>1000000</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>cacheFilter</filter-name>
		<url-pattern>*.jsp</url-pattern>
		<url-pattern>*.html</url-pattern>
		<url-pattern>*.do</url-pattern>
		<dispatcher>REQUEST</dispatcher>
	</filter-mapping>

### XSLT 转换 Filter

XSLT转换时XML文件的功能之一，是利用XSLT样式文件将XML文件转换成其他格式，使用XSLT 转换 Filter，浏览器访问请求XML格式，返回浏览器时已经是转换后的HTML文件了


1、该Filter使用JDK自带的标准XML工具包进行XML格式转换。MSN的聊天记录是用XML形式保存的，浏览器访问XML格式，返回转换后的HTML文件

	public class XSLTFilter implements Filter {
	
		private ServletContext servletContext;
	
		public void init(FilterConfig config) throws ServletException {
			servletContext = config.getServletContext();
		}
	
		public void doFilter(ServletRequest req, ServletResponse res,
				FilterChain chain) throws IOException, ServletException {
	
			HttpServletRequest request = (HttpServletRequest) req;
			HttpServletResponse response = (HttpServletResponse) res;
	
			// 格式样本文件：/book.xsl
			Source styleSource = new StreamSource(servletContext
					.getRealPath("/MessageLog.xsl"));
	
			// 请求的 xml 文件
			Source xmlSource = new StreamSource(servletContext.getRealPath(request
					.getRequestURI().replace(request.getContextPath() + "", "")));
			try {
	
				// 转换器工厂
				TransformerFactory transformerFactory = TransformerFactory
						.newInstance();
	
				// 转换器
				Transformer transformer = transformerFactory
						.newTransformer(styleSource);
	
				// 将转换的结果保存到该对象中
				CharArrayWriter charArrayWriter = new CharArrayWriter();
				StreamResult result = new StreamResult(charArrayWriter);
	
				// 转换
				transformer.transform(xmlSource, result);
	
				// 输出转换后的结果
				response.setContentType("text/html");
				response.setContentLength(charArrayWriter.toString().length());
				PrintWriter out = response.getWriter();
				out.write(charArrayWriter.toString());
	
			} catch (Exception ex) {
			}
		}
	
		public void destroy() {
		}
	}

2、配置

	<filter>
		<filter-name>xsltFilter</filter-name>
		<filter-class>com.helloweenvsfei.xml.XSLTFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>xsltFilter</filter-name>
		<url-pattern>/msn/*</url-pattern>
	</filter-mapping>

## 监听器 Listener

Listener用于监听Java Web程序中的事件，例如创建、修改、删除Session、request、context等

### Listener使用

使用Listener需要实现相应的Listener接口，***触发Listener事件时，Tomcat会自动调用Listener的方法***

#### 实现Listener接口

创建Session服务器会调用sessionCreated()方法，销毁Session（包括sesson超时自动销毁）服务器会调用sessionDestroyed()方法

	public class SessionListenerTest implements HttpSessionListener {
	
	    @Override
	    public void sessionCreated(HttpSessionEvent httpSessionEvent) {
	        //Session创建时被调用
	        HttpSession session = httpSessionEvent.getSession();
	        System.out.println("创建了一个session：" + session);
	    }
	
	    @Override
	    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
	        //销毁Session前被调用
	        HttpSession session = httpSessionEvent.getSession();
	        System.out.println("销毁了一个Session：" + session);
	    }
	}

#### Listener配置
	
&lt;listener>标签一般配置在&lt;servlet>前面

	<listener>
        <listener-class>cn.apeius.listener.SessionListenerTest</listener-class>
    </listener>

### Listener的分类★★★★★★

#### 监听对象的创建与销毁

* HttpSessionListener：监听Session的创建与销毁。创建Session时执行sessionCreated方法，超时或执行session.invalidate()时执行sessionDestroyed方法。<font color='red'>***该Listener可以收集在线者信息***</font>
* ServletContextListener：监听context的创建与销毁。**context代表当前的Web应用程序**，服务器启动或者热部署war包时执行contextInitialized，服务器关闭或关闭该Web时会执行contextDestroyed方法。<font color='red'>***该Listener可用于启动时读取web.xml里配置的初始化参数***</font>
* ServletRequestListener：监听request的创建与销毁，用户每次请求都会执行requestInitialized方法，request处理完毕自动销毁前执行requestDestroyed。注意如果一个HTML页面包含多个图片，则一次请求可能会多次触发request事件

#### 实例：监听Session、request、servletContext

自定义监听器类同时实现HttpSessionListener、ServletContextListener、ServletRequestListener接口，使得多种监听器一块工作

	public class SessionListenerTest implements HttpSessionListener,
	        ServletContextListener, ServletRequestListener {
	
	
	    //Log log = LogFactory.getLog(getClass());
	    Logger log = Logger.getLogger(SessionListenerTest.class);
	
	    // 创建 session
	    public void sessionCreated(HttpSessionEvent se) {
	        HttpSession session = se.getSession();
	        log.info("新创建一个session, ID为: " + session.getId());
	    }
	
	    // 销毁 session
	    public void sessionDestroyed(HttpSessionEvent se) {
	        HttpSession session = se.getSession();
	        log.info("销毁一个session, ID为: " + session.getId());
	    }
	
	    // 加载 context
	    public void contextInitialized(ServletContextEvent sce) {
	        ServletContext servletContext = sce.getServletContext();
	        log.info("即将启动" + servletContext.getContextPath());
	    }
	
	    // 卸载 context
	    public void contextDestroyed(ServletContextEvent sce) {
	        ServletContext servletContext = sce.getServletContext();
	        log.info("即将关闭" + servletContext.getContextPath());
	    }
	
	    // 创建 request
	    public void requestInitialized(ServletRequestEvent sre) {
	
	        HttpServletRequest request = (HttpServletRequest) sre
	                .getServletRequest();
	
	        String uri = request.getRequestURI();
	        uri = request.getQueryString() == null ? uri : (uri + "?" + request
	                .getQueryString());
	
	        request.setAttribute("dateCreated", System.currentTimeMillis());
	
	        log.info("IP " + request.getRemoteAddr() + " 请求 " + uri);
	    }
	
	    // 销毁 request
	    public void requestDestroyed(ServletRequestEvent sre) {
	
	        HttpServletRequest request = (HttpServletRequest) sre
	                .getServletRequest();
	
	        long time = System.currentTimeMillis()
	                - (Long) request.getAttribute("dateCreated");
	
	        log.info(request.getRemoteAddr() + "请求处理结束, 用时" + time + "毫秒. ");
	    }
	}

配置到web.xml
	
	<listener>
        <listener-class>cn.apeius.listener.SessionListenerTest</listener-class>
    </listener>

#### 监听对象的属性变化

* 另一类Listener用于监听Session、context、request的属性变化，接口名称格式为xxxAttributeListener，包括HttpSessionAttributeListener、ServletContextAttributeListener、ServletRequestAttributeListener
* 当想被监听对象中添加、更新、移除属性时，会分别执行xxxAdded()、xxxReplace()、xxxRemoved()方法，xxx分别代表Session、Context、request


	public class SessionAttributeListenerTest implements
			HttpSessionAttributeListener {
	
		Log log = LogFactory.getLog(getClass());
	
		// 添加属性
		public void attributeAdded(HttpSessionBindingEvent se) {
			HttpSession session = se.getSession();
			String name = se.getName();
			log.info("新建session属性：" + name + ", 值为：" + se.getValue());
		}
	
		// 删除属性
		public void attributeRemoved(HttpSessionBindingEvent se) {
			HttpSession session = se.getSession();
			String name = se.getName();
			log.info("删除session属性：" + name + ", 值为：" + se.getValue());
		}
	
		// 修改属性
		public void attributeReplaced(HttpSessionBindingEvent se) {
			HttpSession session = se.getSession();
			String name = se.getName();
			Object oldValue = se.getValue();
			log.info("修改session属性：" + name + ", 原值：" + oldValue + ", 新值："
					+ session.getAttribute(name));
		}
	}

#### 监听Session内的对象

除了上面的6中Listener，有两种Listener用于监听Session内的对象，分别是HttpSessionBindingListener、HttpSessionActivationListener，他们的触发时机是：

* HttpSessionBindingListener：当对象被放到Session里是执行valueBound方法，当对象从Session里移除时执行valueUnbound，***对象必须实现该Listener接口***
* HttpSessionActivationListener：服务器关闭时，会将Session里的内容保存到硬盘上，这个过程叫做钝化。服务器重启时，会将Session里的内容从硬盘上重新加载，钝化时会执行sessionWillPassivate方法，对象被重新加载时执行sessionDidActivate方法，***对象必须实现该Listener接口***

这两个Listener监听的是Session中的对象而非Session等，因此不需要再web.xml中声明

PersonInfo对象被放进、移出Session或者启动、关闭服务器都会触发PersonInfo内的Listener时间：

	public class PersonInfo implements HttpSessionActivationListener,
			HttpSessionBindingListener, Serializable {
	
		private static final long serialVersionUID = -4780592776386225973L;
	
		Log log = LogFactory.getLog(getClass());
	
		private String name;
	
		private Date dateCreated;
	
		public String getName() {
			return name;
		}
	
		public void setName(String name) {
			this.name = name;
		}
	
		public Date getDateCreated() {
			return dateCreated;
		}
	
		public void setDateCreated(Date dateCreated) {
			this.dateCreated = dateCreated;
		}
	
		// 从硬盘加载后
		public void sessionDidActivate(HttpSessionEvent se) {
			HttpSession session = se.getSession();
			log.info(this + "已经成功从硬盘中加载。sessionId: " + session.getId());
		}
	
		// 即将被钝化到硬盘时
		public void sessionWillPassivate(HttpSessionEvent se) {
			HttpSession session = se.getSession();
			log.info(this + "即将保存到硬盘。sessionId: " + session.getId());
		}
	
		// 被放进session前
		public void valueBound(HttpSessionBindingEvent event) {
			HttpSession session = event.getSession();
			String name = event.getName();
			log.info(this + "被绑定到session \"" + session.getId() + "\"的" + name
					+ "属性上");
	
			// 记录放到session中的时间
			this.setDateCreated(new Date());
		}
	
		// 从session中移除后
		public void valueUnbound(HttpSessionBindingEvent event) {
			HttpSession session = event.getSession();
			String name = event.getName();
			log.info(this + "被从session \"" + session.getId() + "\"的" + name
					+ "属性上移除");
		}
	
		@Override
		public String toString() {
			return "PersonInfo(" + name + ")";
		}
	
	}

### Listener使用案例

#### 单态登录

* <font color='red'>**单态登录就是一个账号只能在一台机器上登录，如果在其他机器上登录了，则原来的登录无效**</font>
* 单态登录的目的是为了防止多台机器同时使用一个账号

##### Listener方式实现

* 见JavaWeb王者归来205页
* 思路：当成功验证用户信息，准备往session中存放用户的信息时，被监听器监听到，调用attributeAdded或attributeReplaced方法，方法中需要判断用户是否在别的机器上登录过，如果登录了则使以前的登录失效
* 使用这种方式Listener与登录模块没有代码耦合，部署Listener后将实现单态登录，拆掉该Listener后登录模块照常工作，只是不再保证是单态登录


<pre>
public class LoginSessionListener implements HttpSessionAttributeListener {

	Log log = LogFactory.getLog(this.getClass());

	Map<String, HttpSession> map = new HashMap<String, HttpSession>();

	public void attributeAdded(HttpSessionBindingEvent event) {

		String name = event.getName();

		// 登录
		if (name.equals("personInfo")) {

			PersonInfo personInfo = (PersonInfo) event.getValue();

			if (map.get(personInfo.getAccount()) != null) {

				// map 中有记录，表明该帐号在其他机器上登录过，将以前的登录失效
				HttpSession session = map.get(personInfo.getAccount());
				PersonInfo oldPersonInfo = (PersonInfo) session
						.getAttribute("personInfo");

				log.info("帐号" + oldPersonInfo.getAccount() + "在"
						+ oldPersonInfo.getIp() + "已经登录，该登录将被迫下线。");

				session.removeAttribute("personInfo");
				session.setAttribute("msg", "您的帐号已经在其他机器上登录，您被迫下线。");
			}

			// 将session以用户名为索引，放入map中
			map.put(personInfo.getAccount(), event.getSession());
			log.info("帐号" + personInfo.getAccount() + "在" + personInfo.getIp()
					+ "登录。");
		}
	}

	public void attributeRemoved(HttpSessionBindingEvent event) {

		String name = event.getName();

		// 注销
		if (name.equals("personInfo")) {
			// 将该session从map中移除
			PersonInfo personInfo = (PersonInfo) event.getValue();
			map.remove(personInfo.getAccount());
			log.info("帐号" + personInfo.getAccount() + "注销。");
		}
	}

	public void attributeReplaced(HttpSessionBindingEvent event) {

		String name = event.getName();

		// 没有注销的情况下，用另一个帐号登录
		if (name.equals("personInfo")) {

			// 移除旧的的登录信息
			PersonInfo oldPersonInfo = (PersonInfo) event.getValue();
			map.remove(oldPersonInfo.getAccount());

			// 新的登录信息
			PersonInfo personInfo = (PersonInfo) event.getSession()
					.getAttribute("personInfo");

			// 也要检查新登录的帐号是否在别的机器上登录过
			if (map.get(personInfo.getAccount()) != null) {
				// map 中有记录，表明该帐号在其他机器上登录过，将以前的登录失效
				HttpSession session = map.get(personInfo.getAccount());
				session.removeAttribute("personInfo");
				session.setAttribute("msg", "您的帐号已经在其他机器上登录，您被迫下线。");
			}
			map.put("personInfo", event.getSession());
		}

	}

}
</pre>

##### 自己的实现

* 如果没有要求单态登录，成功验证用户信息后，返回客户端token，在服务器端创建session，并将用户信息存入到session，之后客户端再次访问，根据token恢复session，如果session中存放了用户信息，则用户合法，否则退回登录页面
* 实现单态登录，需要在验证用户信息后，<font color='red'>***判断账号是否在别的机器上登录***</font>。
* 具体实现：用一个HashMap存储在线用户信息，主键为用户名，键值为session。成功验证用户信息后，根据用户名获取session，如果session不为空，说明用户已经在别的机器上登录，则使sesson无效，在HashMap上移除该值，这样之前登录的用户操作前，会检查session中没有用户信息，则必须重新登录，做到强制退出的目的
* 图示：红框是单态登录加入的模块

![](http://i.imgur.com/4vvjT4z.png)

<pre>
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    request.setCharacterEncoding("utf-8");
    response.setCharacterEncoding("utf-8");
    response.setContentType("text/html;charset=utf-8");
    PrintWriter out = response.getWriter();

    String action = request.getParameter("action");
    if("login".equalsIgnoreCase(action)){
        String name = request.getParameter("name");
        String password = request.getParameter("password");
        User user = new User();
        user.setName(name);
        user.setPassword(password);
        if("qm".equals(name) && "123".equals(password)){
            <font color='red'>//判断帐号是否在别的机器登录
            HttpSession oldSession = map.get(name);
            if(oldSession != null){//1、别的机器登录 2、重复登录
                oldSession.removeAttribute("user");
                <font color='blue'>//oldSession.invalidate();不要删除session除了用户信息的其他属性，下次用户在本机登录还能获得先前的数据</font>
                map.remove(name);
            }</font>

            HttpSession session = request.getSession();
            session.setAttribute("user",user);
            map.put(name,session);
            out.println("登录成功");
        }else{
            out.println("登录失败");
        }
    }else if("logout".equalsIgnoreCase(action)){
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        if(user == null){
            out.println("请先登录");
            return;
        }
        //session.invalidate();
        session.removeAttribute("user");
        map.remove(user.getName());
        out.println("注销成功");
    }else if("main".equalsIgnoreCase(action)){
        HttpSession session = request.getSession();
        User user = (User) session.getAttribute("user");
        if(user == null){
            out.println("请先登录");
            return;
        }
        //往session添加其他属性
        Integer count = (Integer) session.getAttribute("count");
        if(count == null){
            count = 0;
            session.setAttribute("count", count);
        }
        count ++;
        session.setAttribute("count",count);

        //在线用户
        out.println(session.getAttribute("count"));
    }
}
</pre>

#### 显示在线用户

存放服务器信息、用户信息的类：

	public class ApplicationConstants {
	
		// 所有的 Session，session_id与session组成键值对
		public static Map<String, HttpSession> SESSION_MAP = new HashMap<String, HttpSession>();
	
		// 当前登录的用户总数
		public static int CURRENT_LOGIN_COUNT = 0;
	
		// 历史访客总数
		public static int TOTAL_HISTORY_COUNT = 0;
	
		// 服务器启动时间
		public static Date START_DATE = new Date();
	
		// 最高在线时间
		public static Date MAX_ONLINE_COUNT_DATE = new Date();
	
		// 最高在线人数
		public static int MAX_ONLINE_COUNT = 0;
	}

使用ServletContextListener来监听服务器的启动与关闭，记录服务器启动时间等

	public class MyContextListener implements ServletContextListener {
	
		public void contextInitialized(ServletContextEvent event) {
			// 启动时，记录服务器启动时间
			ApplicationConstants.START_DATE = new Date();
		}
	
		public void contextDestroyed(ServletContextEvent event) {
			// 关闭时，将结果清除。也可以将结果保存到硬盘上，下次启动时再加载到ApplicationConstants中
			ApplicationConstants.START_DATE = null;
			ApplicationConstants.MAX_ONLINE_COUNT_DATE = null;
		}
	}

对Session的监听，维护在线用户列表、总访问人数：

* 使用Map来索引所有的Session，Session创建的时候放到Map中，Session销毁时从Map中剔除
* 什么时候用户数改变？成功验证用户信息后往session中存入用户信息、用户注销、用户被强退，attributeAdded和attributeReplaced的区别就是用户人数是否需要增加

<pre>
public class MySessionListener implements HttpSessionListener,
		HttpSessionAttributeListener {

	public void sessionCreated(HttpSessionEvent sessionEvent) {

		HttpSession session = sessionEvent.getSession();

		<font color='red'>// 将 session 放入 map
		ApplicationConstants.SESSION_MAP.put(session.getId(), session);</font>
		// 总访问人数++
		ApplicationConstants.TOTAL_HISTORY_COUNT++;

		// 如果当前在线人数超过历史记录，则更新最大在线人数，并记录时间
		if (ApplicationConstants.SESSION_MAP.size() > ApplicationConstants.MAX_ONLINE_COUNT) {
			ApplicationConstants.MAX_ONLINE_COUNT = ApplicationConstants.SESSION_MAP
					.size();
			ApplicationConstants.MAX_ONLINE_COUNT_DATE = new Date();
		}
	}

	public void sessionDestroyed(HttpSessionEvent sessionEvent) {
		HttpSession session = sessionEvent.getSession();
		// 将session从map中移除
		ApplicationConstants.SESSION_MAP.remove(session.getId());
	}

	public void attributeAdded(HttpSessionBindingEvent event) {

		if (event.getName().equals("personInfo")) {

			// 当前登录用户数++
			ApplicationConstants.CURRENT_LOGIN_COUNT++;
			HttpSession session = event.getSession();

			// 查找该帐号有没有在其他机器上登录
			for (HttpSession sess : ApplicationConstants.SESSION_MAP.values()) {

				// 如果该帐号已经在其他机器上登录，则以前的登录失效
				if (event.getValue().equals(sess.getAttribute("personInfo"))
						&& session.getId() != sess.getId()) {
					sess.invalidate();
				}
			}
		}
	}

	public void attributeRemoved(HttpSessionBindingEvent event) {

		// 注销 当前登录用户数--
		if (event.getName().equals("personInfo")) {
			ApplicationConstants.CURRENT_LOGIN_COUNT--;
		}
	}

	public void attributeReplaced(HttpSessionBindingEvent event) {

		// 重新登录，但人数不用增加
		if (event.getName().equals("personInfo")) {
			HttpSession session = event.getSession();
			for (HttpSession sess : ApplicationConstants.SESSION_MAP.values()) {
				// 如果新帐号在其他机器上登录过，则以前登录失效
				if (event.getValue().equals(sess.getAttribute("personInfo"))
						&& session.getId() != sess.getId()) {
					sess.invalidate();
				}
			}
		}
	}

}
</pre>

监听request主要是记录客户的IP地址、访问次数等，也可以记录用户访问的URI：

<pre>
public class MyRequestListener implements ServletRequestListener {

	public void requestDestroyed(ServletRequestEvent event) {
	}

	public void requestInitialized(ServletRequestEvent event) {

		HttpServletRequest request = (HttpServletRequest) event
				.getServletRequest();

		<font color='red'>/*
		* 如果session为空，则重新创建一个session，主要是之前session.invalidate()会销毁session		
		*/</font>
		HttpSession session = request.getSession(true);

		// 记录IP地址
		session.setAttribute("ip", request.getRemoteAddr());

		// 记录访问次数，只记录访问 .html, .do, .jsp, .action 的累计次数
		String uri = request.getRequestURI();
		String[] suffix = { ".html", ".do", ".jsp", ".action" };
		for (int i=0; i&lt;suffix.length; i++) {
			if (uri.endsWith(suffix[i])) {
				break;
			}
			if(i == suffix.length-1)
				return;
		}

		Integer activeTimes = (Integer) session.getAttribute("activeTimes");

		if (activeTimes == null) {
			activeTimes = 0;
		}

		session.setAttribute("activeTimes", activeTimes + 1);
	}
}
</pre>

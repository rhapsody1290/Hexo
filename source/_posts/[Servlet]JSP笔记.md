---
title: JSP笔记

date: 2016-07-14 11:09:00

categories:
- Servlet

tags:
- JSP
- Servlet

---
## 常用

***Java片段***  

	<%!  %> jsp声明，在这里面声明的变量是全局变量，也可以函数定义
	<%  %> Java片段，在这里面声明的变量是局部变量，
	注释 <%-- --%>，<%// % >，<%/* */% >
	<%=  %> 表达式

## JSP简介

* 在web开发过程中，发现servlet做界面比较麻烦（out.println），于是又有一个新的技术JSP
* JSP（Java Servlet Page）运行在服务器的语言，响应客户端请求，动态生成网页的技术

## JSP原理

![](http://i.imgur.com/WeHcjuc.png)

* 如果是第一次访问jsp文件，web服务器就会把showTime.jsp ***翻译*** 成一个showTime_jsp.java（IDEA中showTIme_jsp.java的目录：
<pre>C:\Users\Asus\.IntelliJIdea15\system\tomcat\Unnamed_JSP_Study\work\Catalina\localhost\JSP_Study\org\apache\jsp\showTime_jsp.java）</pre>
* <font color='red'>再将其编译成一个showTime_jsp.class，并把class加载到内存</font>
* 然后创建一个该Servlet的实例，调用其jspInit方法，该方法在Servlet生命周期中只被执行一次，并调用实例的jspService()方法
* 如果是第二次或者以后，就直接访问内存中的实例的jspService()方法。JSP也是单例，所以第一次访问JSP网站速度比较慢，后面访问JSP的速度就变快了
* 如果某个JSP文件被修改了，就相当于重新访问JSP（相当于第一次访问）


## JSP显示页面

Jsp页面中的html排版标签是如何被发送到客户端的？答：JSP中被翻译成Servlet时，HTML标签会<font color='red'>***以out.write()的形式打印出来***，</font>例如：

	out.write("<table border=1>\r\n");
	out.write("<tr><td>apple</td><td>melon</td><td>orange</td></tr>\r\n");
	out.write("<tr><td>apple</td><td>melon</td><td>orange</td></tr>\r\n");
	out.write("<tr><td>apple</td><td>melon</td><td>orange</td></tr>\r\n");
	out.write("<tr><td>apple</td><td>melon</td><td>orange</td></tr>\r\n");
	out.write("</table>\r\n");

## JSP中的Java片段

Jsp页面中的java代码，服务器是如何执行的？比如JSP中的Java代码:

	<%
		int i=90;
		int j=i+90;
	%>
	<h1>测试.</h1>
	<%
		out.println("j="+j);
	%>

  当被翻译成Servlet后，格式如下

	public void _jspService(HttpServletRequest request, HttpServletResponse response)
        throws java.io.IOException, ServletException {
		int i=90;
		int j=i+90;
		out.println("j="+j);
	}

1.	就是有多个<% %> 其实相当于是一个大的 <% %>，所有代码会***放在一个_jspService函数中***
2.	在<% %> 中定义的变量，会成为service函数的局部变量.


## JSP九大内置对象

Web服务器在调用jsp时，会给jsp提供一些内置对象，***这些内置对象无需创建可直接使用***。前五个较常用，分别与Servlet中的几个对象对应

|对象名|						类型|								作用域| 
|------------- |------------- |------------- |
|request：请求对象| 	javax.servlet.ServletRequest的子类|		Request|
|response：响应对象| javax.servlet.ServletResponse的子类|     Page|
|pageContext：页面上下文对象，也是一个域对象，可以setAttribute，其作用范围只是本页面| javax.servlet.jsp.PageContext|  Page|
|session：会话对象，用于保存用户信息，跟踪用户行为| javax.servlet.http.HttpSession|			Session|
|application：应用程序对象，多个用户共享该对象，可以做计数器| javax.servlet.ServletContext|     Application|
|out：输出对象| javax.servlet.jsp.JspWriter|			        Page|
|config：配置对象| javax.servlet.ServletConfig|				Page|
|page：页面对象，代表JSP实例本身，使用较少|	 java.lang.Object|							Page|
|exception：异常对象| java.lang.Throwable|					Page|


## JSP的语法

### 指令元素

　　概念: 用于从jsp发送一个信息到容器，比如设置全局变量,文字编码,引入包

#### ①page指令

	<%@ page contentType="text/html;charset=utf-8"%>	

***常用的属性***  

	language = "xx"，jsp中嵌入代码语言，通常是java
	import = "包.类名"，在jsp页面引入类
	errorPage="err.jsp"，当JSP页面出现错误时，自动跳转到指定页面
	
	contentType 和 pageEncoding的区别：
		contentType = "text/html;charset=utf-8" 指定网页以什么方式显示页面
		pageEncoding="utf-8" 指定Servlet引擎以什么方法翻译jsp->servlet并指定网页以什么方式显示页面

#### ②include指令

<pre>
&lt;%@ include file="文件路径" %&gt;
</pre>

* 该指令用于引入一个文件（通常是JSP文件），JSP引擎会把两个JSP文件翻译成一个Servlet文件，因此也称为静态引入
* 被引入的JSP文件，只需保留page指令即可，html，body等均可省略★★★

#### ③taglib指令 

　　允许在JSP页面使用自定义的标签

	<mytag:yourTag num1 = "123" /> 


### 脚本元素(理解为脚本片段)

#### java片段

	<% java 代码 %>

#### 表达式

	<%=表达式 %>，例如<%=i*78-23%>

#### 定义变量

全局变量

	<%! int i=90; %>

局部变量

	<% int i=90;%>
	
#### 定义函数★★★★★

	<%!
		public int getResult(int a,int b){
			return a+b;
		}
	%>
	注意：函数不能在<% %> 定义.

### 动作元素

***jsp:forward***

	<jsp:forword file="xxx"></jsp:forword>页面跳转

　　在开发JSP的过程中，我们通常把JSP放入WEB-INF目录，目的是为了防止用户直接访问这些jsp文件.  
　　在WebRoot下我们有一个入口页面,它的主要转发

	<jsp:forword file="/WEB-INF/xx.jsp"></jsp:forword>

***jsp:incluce***  

	<%@ include file=""%> 静态引入
	<jsp:incluce  file=""></jsp:incule> 动态引入
	相同点： 把一个文件引入到另外一个文件
	区别：
		静态引入，把两个jsp翻译成一个Servlet,所以被引入的文件不要包含<body><html>
		动态引入，把两个jsp分别翻译,所以被引入的jsp包含有<html><body>也可以

## EL表达式语言

* EL表达式语言可以***方便读取应用程序中的数据***，JSP2.0以上版本即使没有JSTL（JSP标准标签库）也能使用EL
* EL表达式以 `${` 开头，并以 `}` 结束，如${x + y}，从左向右取值，返回结果类型为String
* EL表达式写在JSP的HTML代码中，而不能写在"<%%>"引起的JSP脚本中
* <font color='red'>**使用 `[]` 和 `.` 运算符来访问对象的属性，形式可以是${object.peopertyName}或${object["peopertyName"]}**</font>


### EL内置对象

我们知道jsp有九个内置对象，而EL表达式有11个对象，这些内置对象无需创建可直接使用

#### pageContext

pageContext对象表示当前JSP页面的javax.servlet.jsp.PageContext，包含了9大JSP内置对象


|对象名|						类型|								作用域| 
|------------- |------------- |------------- |
|request：请求对象| 	javax.servlet.ServletRequest的子类|		Request|
|response：响应对象| javax.servlet.ServletResponse的子类|     Page|
|pageContext：页面上下文对象，也是一个域对象，可以setAttribute，其作用范围只是本页面| javax.servlet.jsp.PageContext|  Page|
|session：会话对象，用于保存用户信息，跟踪用户行为| javax.servlet.http.HttpSession|			Session|
|application：应用程序对象，多个用户共享该对象，可以做计数器| javax.servlet.ServletContext|     Application|
|out：输出对象| javax.servlet.jsp.JspWriter|			        Page|
|config：配置对象| javax.servlet.ServletConfig|				Page|
|page：页面对象，代表JSP实例本身，使用较少|	 java.lang.Object|							Page|
|exception：异常对象| java.lang.Throwable|					Page|

例：获得客户端IP

	${pageContext.request.remoteAddr}

#### initParam

包含所有初始化参数的Map，可以获取初始化参数

例，设置初始化参数

	<context-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </context-param>

获得初始化参数

	${initParam.encoding}

#### param

包含所有参数的Map，可以获取参数，返回String

例：url

	http://localhost:8080/SpringMVC_study/?name=xxx

获得参数

	${param.name}	

#### paramValues

包含所有参数的Map，可获取参数数组，返回String[]

例：请求url

	http://localhost:8080/SpringMVC_study/?name=aaa&name=bbb

提交的参数name有多个值{"aaa","bbb"}，使用param只能获取第一个值，二使用paramValues能够获得其他的值

	${paramValues.name[0]}
    ${paramValues.name[1]}

#### header

包含所有头信息的Map，可以获取头信息

例：获得请求主机

	${header.host}

#### cookie

包含所有Cookie的Map，key为Cookie的name

	${cookie.JSESSIONID.value}

#### applicationScope，sessionScope，requestScope，pageScope

分别是包含application，session，request，page作用域变量的Map

以requestScope为例：

使用<jsp:useBean id="person" class="com.jsp.bean.Person"/>声明person对象后，${pageScope.person.age}将输出person的age属性，useBean域默认的作用域为request

若输出session域中的变量，声明使<jsp:useBean id="person" class="com.jsp.bean.Person" scope="session"/>

## JSTL

JSP标准标签库，用来解决遍历map或集合，格式化数字和日期等常见问题

### 依赖

	<dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>

### JSTL核心标签库

JSTL 核心标签库标签共有13个，功能上分为4类：
1. 表达式控制标签：out、set、remove、catch
2. 流程控制标签：if、choose、when、otherwise
3. 循环标签：forEach、forTokens
4. URL操作标签：import、url、redirect

### JSTL核心库

<p><font color='red'>***使用标签时，一定要在jsp文件头加入以下代码：***</font></p>

	<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

### 完整JSTL标签使用

http://www.cnblogs.com/lihuiyy/archive/2012/02/24/2366806.html

### 遍历行为：&lt;c:forEach>

语法：
&lt;c:forEach var="name" items="Collection" varStatus="statusName" begin="begin" end="end" step="step">&lt;/c:forEach>

该标签根据循环条件遍历集合 Collection 中的元素。 var 用于存储从集合中取出的元素；items 指定要遍历的集合；

***遍历list***

<pre>
&lt;%
	List a=new ArrayList();
	a.add("贝贝");
	a.add("晶晶");
	a.add("欢欢");
	a.add("莹莹");
	a.add("妮妮");
	request.setAttribute("a",a);
%>

<font color='red'>&lt;c:forEach var="fuwa" items="${a}">
	&nbsp;&lt;c:out value="${fuwa}"/>&lt;br>
&lt;/c:forEach></font>
</pre>

***遍历Map***

	<%
        Map<String, String> capitals = new HashMap<String,String>();
        capitals.put("Indonesia","Jakarta");
        capitals.put("Malaysia","Kuala Lumpur");
        capitals.put("Thailand","Bangkok");
        request.setAttribute("capitals",capitals);
    %>

    <c:forEach var="capital" items="${capitals}">
        ${capital.key} ${capital.value} <br/>
    </c:forEach>

***forEach嵌套***

	<%
        Map<String, String[]> bigCities = new HashMap<String,String[]>();
        bigCities.put("Australia",new String[]{"Sydney","Melbourne","Perth"});
        bigCities.put("New Zealand",new String[]{"Auckland","Christchurch","Wellington"});
        bigCities.put("Indonesia",new String[]{"Jakarta","Surabaya","Medan"});
        request.setAttribute("bigCities",bigCities);
    %>

    <c:forEach var="mapItem" items="${bigCities}">
        ${mapItem.key} :
            <c:forEach var="city" items="${mapItem.value}">
                ${city}
            </c:forEach>
        <br/>
    </c:forEach>

### 格式化日期和时间

	<b>格式化日期</b><br/>
	default：<fmt:formatDate value="${now}"/> <br/>
	short：<fmt:formatDate value="${now}" dateStyle="short"/> <br/>
	medium：<fmt:formatDate value="${now}" dateStyle="medium"/> <br/>
	long：<fmt:formatDate value="${now}" dateStyle="long"/> <br/>
	full：<fmt:formatDate value="${now}" dateStyle="full"/> <br/><br/>
	
	<b>格式化时间</b></br/>
	default：<fmt:formatDate value="${now}" type="time"/> <br/>
	short：<fmt:formatDate value="${now}" type="time" timeStyle="short"/> <br/>
	medium：<fmt:formatDate value="${now}" type="time" timeStyle="medium"/> <br/>
	long：<fmt:formatDate value="${now}" type="time" timeStyle="long"/> <br/>
	full：<fmt:formatDate value="${now}" type="time" timeStyle="full"/> <br/><br/>
	
	<b>格式化日期和时间</b></br/>
	default：<fmt:formatDate value="${now}" type="both"/> <br/>
	short：<fmt:formatDate value="${now}" type="both" timeStyle="short"/> <br/>
	medium：<fmt:formatDate value="${now}" type="both" timeStyle="medium"/> <br/>
	long：<fmt:formatDate value="${now}" type="both" timeStyle="long"/> <br/>
	full：<fmt:formatDate value="${now}" type="both" timeStyle="full"/> <br/><br/>
	
	<b>定制格式化日期和时间</b></br/>
	<fmt:formatDate value="${now}" type="both" pattern="yyyy-MM-dd HH:mm:ss"/> <br/>
	<fmt:formatDate value="${now}" type="both" pattern="yy/MM/dd HH:mm:ss"/> <br/>

浏览器显示：

![](http://i.imgur.com/4NLTcOv.png)
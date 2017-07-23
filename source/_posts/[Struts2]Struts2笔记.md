---
title: Struts2笔记

date: 2016-08-11 00:00:00

categories:
- Struts2

tags:
- Java
- Struts2

---
## 什么是Struts2

* Struts2是一个MVC框架
* Struts2起源于WebWork框架，Struts2就是WebWork2
* Struts2是对Struts1的一个补充，而不是替代品

## Struts2快速入门(Action、struts.xml、跳转结果)

### 登录功能Action

	package cn.apeius.action;
	
	import com.opensymphony.xwork2.ActionSupport;
	import com.sun.net.httpserver.Authenticator;
	
	/**
	 * Created by Asus on 2016/8/11.
	 */
	public class LoginAction extends ActionSupport {
	    private String account;
	    private String password;
	
	    public String execute(){
	        if("admin".equals(account) && "admin".equals(password)){
	            return SUCCESS;
	        }else{
	            return LOGIN;
	        }
	    }
	    public String getAccount() {
	        return account;
	    }
	
	    public void setAccount(String account) {
	        this.account = account;
	    }
	
	    public String getPassword() {
	        return password;
	    }
	
	    public void setPassword(String password) {
	        this.password = password;
	    }
	}

* 新建LoginAction，Struts2的Action需要继承com.opensymphony.xwork2.ActionSupport
* LoginAction中有两个属性account、password，代表JSP表单的两个输入框。Struts2会自动把输入框中的内容通过getter、setter方法设置进来
* 还有一个execute方法，提交数据后会自动调用该方法
* 返回值代表结果页面的跳转，具体文件见配置文件

### Struts2配置文件

	<?xml version="1.0" encoding="UTF-8"?>
	
	<!DOCTYPE struts PUBLIC
	        "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
	        "http://struts.apache.org/dtds/struts-2.3.dtd">
	
	<struts>
	    <!--定义一个package，所有的result和action等必须配置到package-->
	    <package name="main" extends="struts-default">
	        <!--所有的全局result-->
	        <global-results>
	            <!--名为login的result-->
	            <result name = "login">/login.jsp</result>
	        </global-results>
	        <!--LoginAction-->
	        <action name="LoginAction" class="cn.apeius.action.LoginAction">
	            <!--名为success的result-->
	            <result name="success">/success.jsp</result>
	        </action>
	    </package>
	</struts>

* Struts2所有的result、action等必须配置到package中，自定义的package一般继承struts-default
* JSP配置在result中。上述配置文件配置了两个result，一个是名为success，配置在action里面，当登录成功后跳转到success.jsp里面；另一个是全局的result，配置在global-results中，名为login，访问任何页面首页跳转到login.jsp

### JSP登录页面

	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<%@ taglib uri="/struts-tags" prefix="struts"%>
	<html>
	<head>
	    <title>登录</title>
	</head>
	<body>
	    <struts:form action="LoginAction">
	        <struts:label value="登录系统"></struts:label>
	        <struts:textfield name="account" label="帐号"/>
	        <struts:password name="password" label="密码"/>
	        <struts:submit value="登录"></struts:submit>
	    </struts:form>
	</body>
	</html>

### 配置web.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	         version="3.1">
	    <!--struts2的filter，所有请求都被映射到struts上-->
	    <filter>
	        <filter-name>struts2</filter-name>
	        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
	    </filter>
	    <!--匹配所有URL-->
	    <filter-mapping>
	        <filter-name>struts2</filter-name>
	        <url-pattern>/*</url-pattern>
	    </filter-mapping>
	
	    <welcome-file-list>
	        <welcome-file>login.jsp</welcome-file>
	    </welcome-file-list>
	</web-app>

* Struts2使用Filter作为分发器，配置ur-pattern最好配置为/*

## Struts2工作流程★★★★★

* 用户访问JSP页面 login.jsp，跳出登录页面
* 提交表单后数据提交给LoginAction
* Struts2***拦截所有请求***，包括*.action的请求
* ***查找struts.xml***，LoginAction对应LoginAction类，生成LoginAction实例，并将提交的***数据注入*** 该实例中（从request中获取参数），调用LoginAction实例的***execute方法***
* 根据返回结果，决定***跳转*** JSP页面

## Struts2线程安全

* Struts1所有Action都只有一个实例，不是线程安全
* Struts2对每个请求生成一个实例，处理完成即销毁，是线程安全的（Struts2是基于类设计的，参数绑定到类的属性，所有的方法都可以使用，Struts2必须是多例的，如果是单例会导致不同用户数据的冲突）

## Action详解

### 通配符配置Action

BookAction.java

	public class BookAction extends ActionSupport {
	
		public static List<Book> bookList = new ArrayList<Book>();
		private String title;
		private Book book;
	
		public String add() {
			bookList.add(book);
			title = "<br/><br/>添加书籍成功<br/><br/>";
			return "success";
		}
	
		// 书籍列表
		@SkipValidation
		public String list() {
			return "list";
		}
	
		// 清空书籍列表
		@SkipValidation
		public String clear() {
			bookList.clear();
			title = "<br/><br/>清空书籍列表成功<br/><br/>";
			return "list";
		}
	
	}

struts.xml

	<action name="*Book"
		class="com.helloweenvsfei.struts2.action.BookAction" method="{1}">
		<result>/successBook.jsp</result>
		<result name="{1}">/{1}Book.jsp</result>
		<result name="input">/initAddBook.jsp</result>
		<result name="list">/listBook.jsp</result>
	</action>
 
* `*`代表的内容可以在本Action配置内部使用{1},{2}等引用
* 举个例子，访问listBook.action时，执行list()方法


## 常见问题

### Struts2中关于"There is no Action mapped for namespace / and action name"的总结

http://www.cnblogs.com/gulvzhe/archive/2011/11/21/2256632.html


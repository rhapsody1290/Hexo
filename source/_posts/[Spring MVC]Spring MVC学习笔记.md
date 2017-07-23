---
title: Spring MVC学习笔记

date: 2016-09-01 00:00:00

categories:
- Spring MVC

tags:
- JavaEE
- Spring MVC

---

## Spring MVC简介

* ***SpringMVC是一个基于MVC设计理念的web框架***
* Spring MVC通过一套MVC注解，让POJO成为处理请求的控制器，无需实现任何接口，同时SpringMVC还支持REST风格的URL请求
* Spring MVC在框架设计、扩展性、灵活性等方面全面超越Struts、WebWork等MVC框架，成为MVC的领跑者
* Spring MVC框架围绕DispatcherServlet这个核心展开，DispatcherServlet是Spring MVC框架的总导演、总策划，他负责截获请求并将其分派给相应地处理器处理

## Spring MVC 整体架构

![](http://i.imgur.com/8Eb6auw.png)

1、用户发起请求到控制器 DispatcherServlet (请求分派)
2、DispatcherServlet 去 handlerMapping 查找 Handler 对应的 Handler（Handler可以理解成struts中的 action）
3、HandlerMapper 返回 HandlerExecutorChain 执行链（包含两部分内容：Handler,拦截器集合）
4、DispatcherServlet 调用 HandlerAdapter
5、HandlerAdapter 调用 Handler
6、Handler 处理具体的业务逻辑
7、Handler 处理完业务逻辑之后，返回 ModelAndView 给 HandlerAdapter，其中的 View 是视图名称，Modal 是数据模型
8、HandlerAdapter 将 ModelAndView 返回给 DispatcherServlet
9、DispatcherServlet 通过 ModelAndView 中的视图名称在视图解析器中查找视图
10、视图解析器返回真正的 View 视图对象
11、渲染视图
12、返回用户响应

## Spring MVC快速入门★★★★★★

### github

https://github.com/rhapsody1290/SpringMVC_study

### 导入依赖

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	
	    <groupId>cn.apeius</groupId>
	    <artifactId>Demo</artifactId>
	    <version>1.0-SNAPSHOT</version>
	    <packaging>war</packaging>
	
	    <!-- 全局属性配置,定义变量 -->
	    <properties>
	        <!--用来定义war包名称-->
	        <project.build.name>tools</project.build.name>
	        <!--用来定义资源文件的编码格式  -->
	        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	        <!-- spring版本号 -->
	        <spring.version>4.0.2.RELEASE</spring.version>
	        <!-- mybatis版本号 -->
	        <mybatis.version>3.2.8</mybatis.version>
	        <!-- log4j日志文件管理包版本 -->
	        <slf4j.version>1.7.7</slf4j.version>
	        <log4j.version>1.2.16</log4j.version>
	    </properties>
	
	    <dependencies>
	        <!--Spring MVC-->
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-webmvc</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <!--日志依赖-->
	        <dependency>
	            <groupId>org.slf4j</groupId>
	            <artifactId>slf4j-log4j12</artifactId>
	            <version>${slf4j.version}</version>
	        </dependency>
	        <!-- JSTL标签类 -->
	        <dependency>
	            <groupId>jstl</groupId>
	            <artifactId>jstl</artifactId>
	            <version>1.2</version>
	        </dependency>
	        <!--JSP相关-->
	        <dependency>
	            <groupId>javax.servlet</groupId>
	            <artifactId>javax.servlet-api</artifactId>
	            <version>3.0.1</version>
	            <scope>provided</scope>
	        </dependency>
	        <dependency>
	            <groupId>javax.servlet.jsp</groupId>
	            <artifactId>javax.servlet.jsp-api</artifactId>
	            <version>2.2.1</version>
	            <scope>provided</scope>
	        </dependency>
	    </dependencies>
	</project>

### web.xml

* Spring MVC自带一个<font color='red'>***Dispatcher Servlet***</font>，要是用这个servlet，需要把它配置到***部署描述符（web.xml文件）***
* load-on-startup 元素是可选的，如果它存在，则在***应用程序启动时*** 装载该servlet，并调用它的init方法；若它不存在，则在该servlet***第一次请求*** 时进行加载
* ★ DispatcherServlet将使用Spring MVC的诸多默认组件，在初始化时它会寻找WEB-INF目录下的配置文件，其命名规则为***servletName-servlet.xml***
* **url-pattern为/时，拦截所有请求，包括静态资源**[不拦截JSP]，此时需要设置annotation-driven和resources


	<?xml version="1.0" encoding="UTF-8"?>
	<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
	         version="3.1">
	
	    <servlet>
	        <servlet-name>springmvc</servlet-name>
	        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
			<load-on-startup>1</load-on-startup>
	    </servlet>
	    <servlet-mapping>
	        <servlet-name>springmvc</servlet-name>
	        <url-pattern>*.do</url-pattern>
	    </servlet-mapping>
	    <welcome-file-list>
	        <welcome-file>index.jsp</welcome-file>
	    </welcome-file-list>
	</web-app>

### spring-mvc的配置

#### 添加springmvc-servlet.xml

* spring-mvc 会默认去WEB-INF的目录下寻找${serlvet-name}-serlvet.xml的文件
* 所以我们把serlvetmvc的配置文件添加到web-inf的目录下，并且名字与 web.xml 中的servlet-name 相同

webapp/WEB-INF/springmvc-servlet.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xmlns:mvc="http://www.springframework.org/schema/mvc"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
	        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	
	</beans>

可以把配置文件放在应用程序的任意目录，用servlet定义的init-param元素，以便DispatcherServlet加载到该文件。在web.xml文件中，定义的DispatcherServlet中加入init-param节点，在里面写上系统中spring mvc配置文件的位置。

web.xml

	<servlet>
	    <servlet-name>springmvc</servlet-name>
	    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	    <init-param>
	        <param-name>contextConfigLocation</param-name>
	        <param-value>/WEB-INF/config/springmvc-servlet.xml</param-value>
	    </init-param>
	    <load-on-startup>1</load-on-startup>
	</servlet>

当然还是推荐放在WEB-INF的目录下的${serlvet-name}-serlvet.xml的文件

#### 自定义Handler（controller）★★★★★★

* 在Spring2.5版本前，开发一个控制器的唯一方法是***实现org.springframework.web.servlet.mvc.Controller接口***，这个接口公开了一个handleRequest方法
* 其实现类可以访问请求的HttpServletRequest和HttpServletResponse
* 还必须返回包含<font color='red'>***视图路径*** </font>或<font color='blue'>***视图路径和模的ModelAndView对象***</font>
* Controller接口的实现类只能处理一个单一动作（Action）（***一个Action对应一个Controller的实现类。调用其handleRequest方法***），而一个基于注解的控制器可以同时支持多个请求处理的动作，并且无需实现任何接口


	public class Hello implements Controller{
	
	    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
	        ModelAndView mv = new ModelAndView();
	        //设置试图名称
	        mv.setViewName("hello");
	        mv.addObject("msg","这是第一个springmvc程序");
	        return mv;

			/*也可以直接
			return new ModelAndView("hello","msg","这是第一个springmvc程序");
			*/
	    }
	}

#### 在springmvc-servlet.xml 中配置Handler

* 当DispatcherServlet收到/hello.do的请求时，自动构建控制器Hello，并调用其handleRequest方法，返回ModelAndView


	<!--配置自定义Handler，name表示对应的访问路径-->
    <bean name="/hello.do" class="cn.apeius.springmvc.controller.Hello"/>

#### 配置HandlerMapping（非必须，需要注释）

	<!--配置handlerMapping-->
    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

#### 配置handlerAdapter（非必须，需要注释）

	<!--配置handlerAdapter-->
    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>

#### 配置视图解析器

* 视图解析器<font color='red'>***可以不设置***</font>，但需填入完整路径，如 /WEB-INF/views/hello.jsp
* 视图解析器负责解析视图
* 可以通过在配置文件中定义一个ViewResolver来配置视图解析器
* 视图解析器配置有***前缀*** 和***后缀*** 两个属性，这样一来***view路径将缩短***，如只需填hello


	<!--配置试图解析器
    Example: prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" -> "/WEB-INF/jsp/test.jsp-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

#### 定义视图

*** 文件路径在/WEB-INF/views/hello.jsp***

![](http://i.imgur.com/1hiYmrm.png)

***hello.jsp中可以使用ModelAndView中的对象***

	<%@ page contentType="text/html;charset=UTF-8" language="java" %>
	<html>
	  <head>
	    <title>hello</title>
	  </head>
	  <body>
	      <font color="red">${msg}</font>
	  </body>
	</html>

## 分析第一个案例的执行过程

### 方法一：看日志

***导入日志***

log4j.properties

	log4j.rootLogger=DEBUG,A1
	log4j.logger.org.mybatis=DEBUG
	log4j.appender.A1=org.apache.log4j.ConsoleAppender
	log4j.appender.A1.layout=org.apache.log4j.PatternLayout
	log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n

***在web.xml中配置servlet 服务器开启的时候启动***

	<servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

### 方法二：看源码

在DispatcherServlet的doDispatch方法中打断点

## 精简之后的配置

springmvc-servlet.xml

	<!--配置自定义Handler，name表示对应的访问路径-->
    <bean name="/hello.do" class="cn.apeius.springmvc.controller.Hello"/>

    <!--配置试图解析器
    Example: prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" -> "/WEB-INF/jsp/test.jsp-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

## 第一个注解程序★★★★★★

基于注解的控制器有以下几个优点：

* <font color='red'>***一个控制器类可以处理多个动作***</font>，而一个实现了Controller接口的控制器***只能处理一个动作***。***这就允许将相关的操作都写在同一个控制器内***，从而减少应用程序中类的数量
* 基于注解的控制器的<font color='red'>***请求映射不需要存储在配置文件中*** </font>，使用RequestMapping注释类型，可以对一个方法进行请求处理

### 创建注解的步骤

***1、书写一个类，需要在类上面加上@Controller***

* 用于指示Spring类的实例是一个控制器
* Spring使用***扫描机制*** 来找到应用程序中所有***基于注解的控制器类***

***2、@RequestMapping映射一个请求与请求处理方法。一个采用@RequestMapping注释的方法将成为一个请求处理方法，访问相应的URL请求时调用。@RequestMapping使用后面有详解***

	@Controller
	public class AnnotationHello {
	
	    @RequestMapping(value = "/show1")//可以省略后缀
	    public ModelAndView test1(){
	        ModelAndView mv = new ModelAndView();
	        mv.setViewName("hello");
	        mv.addObject("msg","这是第一个注解程序");
	        return mv;
	    }
	}


***3、在springmvc的配置文件中，去配置扫描包，使@Controller生效，和spring配置扫描包的方式一样***

* 为了保证Spring能找到你的控制器，需要在Spring MVC的配置文件中声明spring-context
	  xmlns:context="http://www.springframework.org/schema/context"
* 在<component-scan/>元素中指定控制器类的基本包
	  <context:component-scan base-package="cn.apeius.springmvc.controller"/>

***4、测试***

	http://localhost:8080/SpringMVC_study/show1.do

### 推荐使用的HandlerMapper和HandlerAdapter（什么意思？？）

![](http://i.imgur.com/GRSWFil.png)

### 使用注解驱动替换推荐的配置（什么意思？？）

* url-pattern为/时，拦截所有请求，包括静态资源，此时需要在springmvc-config.xml中设置annotation-driven和resources元素
* annotation-driven 相当于注册了DefaultAnnotationHandlerMapping和AnnotationMethodHandlerAdapter两个bean，配置一些messageconverter，即解决了@Controller注解的使用前提配置
* resource 元素则指示Spring MVC哪些静态资源需要单独处理（不通过DispatcherServlet）；如果没有annotation-driven，resource元素会阻止任意控制器被调用；若不使用resources，则不需要annotation-driven


	<!--配置注解驱动，会默认加载HandlerMapping，HandlerAdapter-->
    <mvc:annotation-driven/>
	<!-- 确保/css目录下所有文件可见-->
    <mvc:resources mapping="/css/**" location="/css/"/>
	<!--允许显示所有的.html文件-->
    <mvc:resources mapping="/*.html" location="/"/>

* 或者使用mvc:default-servlet-handler，过滤掉所有的静态资源


	<!--过滤掉所有的静态资源，把静态资直接交给tomcat去处理-->
    <mvc:default-servlet-handler/>

## 使用RequestMapping映射请求★★★★★★

* 在SpringMVC中的众多Controller以及每个Controller的众多方法，请求是如何映射到具体的处理方法上？这个就是靠@RequestMapping完成的，***他完成了一个请求与请求处理方法的映射***
* <font color='red'>@RequestMapping既可以定义在类上也可以定义在方法上，请求映射的规则是：</font>
  	类上面的@RequestMapping.value + 方法上面的@RequestMapping.value

举个例子：
—— 控制器：

	@Controller
	@RequestMapping("test")
	public class AnnotationHello {
	
	    @RequestMapping("show1")//可以省略后缀
	    public ModelAndView test1(){
	        ModelAndView mv = new ModelAndView();
	        mv.setViewName("hello");
	        mv.addObject("msg","这是第一个注解程序");
	        return mv;
	    }
	}

—— 访问URL：

	http://localhost:8080/SpringMVC_study/test/show1.do

### 五种映射

#### 标准URL映射 

* 标准URL映射是最简单的一种映射，例如：

	@RequestMapping("hello")或	
	@RequestMapping（"/hello"）或
	@RequestMapping("value="/hello"")

* 请求URL：
		http://localhost:8080/web应用名/hello.do

* value是RequestMapping的注释的默认属性，因此若只有唯一的属性，则可以省略属性名字，即以下两个标注含义相同：
	  @RequestMapping("show1")
	  @RequestMapping(value = "show1")

* 但如果有超过一个属性时，就必须写入value属性名称

#### Ant风格的URL映射（通配符）

![](http://i.imgur.com/APZ3QF7.png)

举例1：

	@RequestMapping("/test/*/show2")
    public ModelAndView test2(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","/test/*/show2");
        return mv;
    }

能匹配：

	/test/a/show2.do
	/test/abc/show2.do
	/test/show2.do 匹配不到

举例2：

	@RequestMapping("/test/**/show3")
    public ModelAndView test3(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","/test/**/show3");
        return mv;
    }

能匹配：

	http://localhost:8080/web应用名/test/a/show3.do
	http://localhost:8080/web应用名/test/b/a/d/show3.do


#### 占位符映射 

* Url中可以通过一个或多个{xxxx}占位符映射,通过@PathVariable("xxx")绑定到方法的入参中
* 占位符映射让传递参数多了一种方式

例如： 

	@RequestMapping("/test/{itemId}/{itemName}")
    public ModelAndView test4(@PathVariable("itemId") String itemId, @PathVariable("itemName") String itemName){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","itemId:" + itemId + " " + "itemName:" + itemName);
        return mv;
    }

URL:

	http://localhost:8080/SpringMVC_study/test/2011/iphone6s.do

结果：

	itemId:2011 itemName:iphone6s


#### 限制请求方法映射 

***若无指定method属性，则请求处理方法可以处理任意HTTP方法***

***只允许get请求访问***

	@RequestMapping(value = "test5", method = RequestMethod.GET)
    public ModelAndView test5(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","限制只有get请求才能进入");
        return mv;
    }

***只允post和get方法***

	@RequestMapping(value = "test5", method = {RequestMethod.GET,RequestMethod.POST})
    public ModelAndView test5(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","限制只有get、post请求才能进入");
        return mv;
    }

#### 限制参数映射 

	@RequestMapping(value = "/test6",params = "userId")
    public ModelAndView test6(@RequestParam(value="userId") String userId){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","限定请求参数，必须要有Userid这个信息；userid：" + userId);
        return mv;
    }

* 请求中必须带userId参数
* 参数的规则如下
	* params="userId"请求参数中必须包含userId
	* params="!userId"请求参数中不能包含userId
	* params="userId!=1"请求参数必须包含userId，但其值不能为1
	* params="{"userId","name"}"必须包含userId和name参数


请求URL

	http://localhost:8080/SpringMVC_study/test6.do?userId=2

## 请求响应处理方法的数据绑定

* 一个请求URL与请求响应方法一一对应
* 数据绑定是将***用户输入*** 与***领域模型*** 相互绑定
	* 类型总为***String*** 的HTTP请求参数，可用于填充***不同类型***的对象属性
	* 数据绑定使得form bean变成多余

### 绑定servlet内置对象

非常简单，只需在参数中加入需要使用的内置对象，常用的有HttpServletRequest、HttpServletResponse、HttpSession

	@RequestMapping(value = "/test7")
    public ModelAndView test7(HttpServletRequest request, HttpServletResponse response, HttpSession session){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","Servlet中的对象");
        System.out.println(request);
        System.out.println(response);
        System.out.println(session);
        return mv;
    }

### @PathVariable获取占位符中的参数

* 通过@PathVariable可以绑定占位符参数到方法参数中
* 路径变量的类型可以不是字符串
* ***可以使用多个路径变量***


	@RequestMapping("/test/{itemId}/{itemName}")
    public ModelAndView test4(@PathVariable("itemId") String itemId, @PathVariable("itemName") String itemName){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","itemId:" + itemId + " " + "itemName:" + itemName);
        return mv;
    }

* 特殊说明：不要省略@PathVariable中的参数

![](http://i.imgur.com/LDhruoO.png)

### @RequestParam ★★★★★

* 将请求参数传入到方法中，如http://localhost:8080/SpringMVC_study/test8?userId=qm
* @RequestParam 注解的参数类型不一定是字符串，可以是整型等其他

<pre><font color='red'>/**
 * required：必须要有这个参数
 * defautlValue：默认值，如果defaultValue设置了值，那么required失效
 * @param userId
 * @return
 */</font>
@RequestMapping(value = "/test8")
public ModelAndView test8(@RequestParam(value = "userId",required = true,defaultValue = "10") String userId){
    ModelAndView mv = new ModelAndView();
    mv.setViewName("hello");
    mv.addObject("msg","userId：" + userId);
    return mv;
}
</pre>

### @CookieValue

在Spring MVC中通过@CookieValue可以轻松获取cookie的值

***Servlet中如何获取指定的cookie值***

	Cookie[] cookies = request.getCookies();
	//对数组进行遍历，根据cookie中的name 来找到对应的cookie
	for(Cookie cookie : cookies){
		if("abc".equals(cookie.getName()))
	    	cookie.getValue();//获取cookie中的值
	}

***使用@CookieValue***

	@RequestMapping(value = "/test9")
    public ModelAndView test9(@CookieValue("JSESSIONID") String JSESSIONID){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg","JESSIONID：" + JSESSIONID);
        return mv;
    }

### POJO对象绑定参数

SpringMVC会将请求过来的 ***参数名*** 和POJO实体中的 ***属性名*** 进行匹配，如果名称一致，将把值填充到对象中

	@RequestMapping(value = "/test10")
    public ModelAndView test10(User user){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg",user);
        return mv;
    }

User.java

	public class User {
	
	    private String userName;
	    private String password;
	
	    public String getPassword() {
	        return password;
	    }
	
	    public void setPassword(String password) {
	        this.password = password;
	    }
	
	    public String getUserName() {
	        return userName;
	    }
	
	    public void setUserName(String userName) {
	        this.userName = userName;
	    }
	
	    @Override
	    public String toString(){
	        return userName + " " + password;
	    }
	}

### 自定义复合对象类型

User对象中有ContactInfo属性，Controller中的代码和第3点说的一致，但是，在表单代码中，需要使用<font color='red'>***"属性名(对象类型的属性).属性名"***</font>来命名input的name

Model代码：

	public class ContactInfo {
	    private String tel;
	    private String address;
	
	    public String getTel() {
	        return tel;
	    }
	
	    public void setTel(String tel) {
	        this.tel = tel;
	    }
	
	    public String getAddress() {
	        return address;
	    }
	
	    public void setAddress(String address) {
	        this.address = address;
	    }
	
	}
	
	public class User {
	    private String firstName;
	    private String lastName;
	    private ContactInfo contactInfo;
	
	    public String getFirstName() {
	        return firstName;
	    }
	
	    public void setFirstName(String firstName) {
	        this.firstName = firstName;
	    }
	
	    public String getLastName() {
	        return lastName;
	    }
	
	    public void setLastName(String lastName) {
	        this.lastName = lastName;
	    }
	
	    public ContactInfo getContactInfo() {
	        return contactInfo;
	    }
	
	    public void setContactInfo(ContactInfo contactInfo) {
	        this.contactInfo = contactInfo;
	    }
	
	}

Controller代码：

<pre>
@RequestMapping("saysth.do")
public void test(User user) {
    System.out.println(user.getFirstName());
    System.out.println(user.getLastName());
    System.out.println(<font color='red'>user.getContactInfo().getTel()</font>);
    System.out.println(<font color='red'>user.getContactInfo().getAddress()</font>);
}
</pre>

表单代码：

<pre>
&lt;form action="saysth.do" method="post">
&lt;input name="firstName" value="张" />&lt;br>
&lt;input name="lastName" value="三" />&lt;br>
&lt;input name=<font color='red'>"contactInfo.tel"</font> value="13809908909" />&lt;br>
&lt;input name=<font color='red'>"contactInfo.address"</font> value="北京海淀" />&lt;br>
&lt;input type="submit" value="Save" />
&lt;/form>
</pre>


### Java的基本数据类型绑定

* 表单中input的name值和Controller的<font color='red'>***参数变量名保持一致***</font>，就能完成数据绑定，如果***不一致可以使用@RequestParam注解***
* 如果Controller方法参数中定义的是基本数据类型，但是从页面提交过来的***数据为null或者""的话，会出现数据转换的异常***。也就是必须保证表单传递过来的数据不能为null或""，所以，在开发过程中，对可能为空的数据，最好将参数数据类型定义成<font color='red'>***包装类型***</font>
* <font color='red'>***Java的基本数据类型数组可以自动转换***</font>，如String[] interests，但是用户自定义User[] users不行，需要将User[]包装到一个UserForm中，通List操作

表单代码

	<form action="/demos/demo1.action" method="post">
		<div>姓名:</div>
		<div><input name="name" value="张三"/></div>
		<div class="clear"></div>
		<div>年龄:</div>
		<div><input name="age" value="20"/></div>
		<div class="clear"></div>
		<div>收入:</div>
		<div><input name="income" value="100000"/></div>
		<div class="clear"></div>
		<div>结婚:</div>
		<div>
		<input type="radio" name="isMarried" value="true" checked="checked"/>是
		<input type="radio" name="isMarried" value="false"/>否</div>
		<div class="clear"></div>
		<div>兴趣:</div>
		<div>
		<input type="checkbox" name="interests" value="听歌" checked="checked"/>听歌
		<input type="checkbox" name="interests" value="书法" checked="checked"/>书法
		<input type="checkbox" name="interests" value="看电影" checked="checked"/>看电影
		</div>
		<div class="clear"></div>
		<div><input type="submit" value="提交表单"/></div>
	</form>

测试：

![](http://i.imgur.com/u6Gq5bU.png)

### 集合List绑定

* 如果方法需要接受的list集合，不能够直接在方法中书写List，List的绑定，需要 ***将List对象包装到一个类中*** 才能绑定<font color='red'>（***如果List中的类型是Java自带的，则可以自动转换***</font>，如List &lt;Object>，List<String>，而List &lt;User>不行）
* 与"自定义复合对象类型"数据绑定类似，但是UserForm对象的属性被定义成List，而不是普通自定义对象，所以在表单中需要指定List的下标
* Spring会创建一个以最大下标值为size的List对象，List中的对象，只有在表单中对应有下标的那些才会有值，否则会为null


表单

	<form action="/SpringMVC_study/test11.do">
			用户1：<input type="text" name="users[0].userName"/><br/>
			用户2：<input type="text" name="users[1].userName"/><br/>
			用户3：<input type="text" name="users[2].userName"/><br/>
			<input type="submit" value="测试"/>
	</form>

将List对象包装到一个类中

<pre>
public class UserForm {

    <font color='red'>private List<User> users;</font>

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }
}
</pre>

Action

<pre>
@RequestMapping(value = "/test11")
public ModelAndView test11(<font color='red'>UserForm userForm</font>){
    ModelAndView mv = new ModelAndView();
    mv.setViewName("hello");
    mv.addObject("msg","List集合");

    for(User user : userForm.getUsers()){
        System.out.println(user);
    }

    return mv;
}
</pre>

### 集合Set绑定

Set和List类似，也需要绑定在对象上，而不能直接写在Controller方法的参数中。但是，绑定Set数据时，***必须先在Set对象中add相应的数量的模型对象***

Model代码：

<pre>
public class User {
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

}

public class UserSetForm {

    <font color='red'>private Set<User> users = new HashSet<User>();</font>

    public UserSetForm() {
        <font color='red'>users.add(new User());
        users.add(new User());
        users.add(new User());</font>
    }

    public Set<User> getUsers() {
        return users;
    }

    public void setUsers(Set<User> users) {
        this.users = users;
    }

}
</pre>

Controller代码：

	@RequestMapping("saysth.do")
	public void test(UserSetForm userForm) {
	    for (User user : userForm.getUsers()) {
	        System.out.println(user.getFirstName() + " - " + user.getLastName());
	    }
	}

表单代码：

<pre>
&lt;form action="saysth.do" method="post">
	&lt;table>
		&lt;thead>
			&lt;tr>
			&lt;th>First Name&lt;/th>
			&lt;th>Last Name&lt;/th>
			&lt;/tr>
		&lt;/thead>
		&lt;tfoot>
			&lt;tr>
			&lt;td colspan="2">&lt;input type="submit" value="Save" />&lt;/td>
			&lt;/tr>
		&lt;/tfoot>
		&lt;tbody>
			&lt;tr>
			<font color='red'>&lt;td>&lt;input name="users[0].firstName" value="aaa" />&lt;/td>
			&lt;td>&lt;input name="users[0].lastName" value="bbb" />&lt;/td></font>
			&lt;/tr>
			&lt;tr>
			<font color='red'>&lt;td>&lt;input name="users[1].firstName" value="ccc" />&lt;/td>
			&lt;td>&lt;input name="users[1].lastName" value="ddd" />&lt;/td></font>
			&lt;/tr>
			&lt;tr>
			<font color='red'>&lt;td>&lt;input name="users[2].firstName" value="eee" />&lt;/td>
			&lt;td>&lt;input name="users[2].lastName" value="fff" />&lt;/td></font>
			&lt;/tr>
		&lt;/tbody>
	&lt;/table>
&lt;/form>
</pre>

### Map绑定

Map最为灵活，它也需要绑定在对象上，而不能直接写在Controller方法的参数中

Model代码：

<pre>
public class User {
    private String firstName;
    private String lastName;

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

}

public class UserMapForm {
    <font color='red'>private Map<String, User> users;</font>

    public Map<String, User> getUsers() {
        return users;
    }

    public void setUsers(Map<String, User> users) {
        this.users = users;
    }

}
</pre>

Controller代码：

	@RequestMapping("saysth.do")
	public void test(UserMapForm userForm) {
	    for (Map.Entry<String, User> entry : userForm.getUsers().entrySet()) {
	        System.out.println(entry.getKey() + ": " + entry.getValue().getFirstName() + " - " +
	        entry.getValue().getLastName());
	    }
	}

表单代码：

<pre>
&lt;form action="saysth.do" method="post">
	&lt;table>
	&lt;thead>
		&lt;tr>
			&lt;th>First Name&lt;/th>
			&lt;th>Last Name&lt;/th>
		&lt;/tr>
	&lt;/thead>
	&lt;tfoot>
		&lt;tr>
			&lt;td colspan="2">&lt;input type="submit" value="Save" />&lt;/td>
		&lt;/tr>
	&lt;/tfoot>
	&lt;tbody>
		&lt;tr><font color='red'>
			&lt;td>&lt;input name="users['x'].firstName" value="aaa" />&lt;/td>
			&lt;td>&lt;input name="users['x'].lastName" value="bbb" />&lt;/td></font>
		&lt;/tr>
		&lt;tr><font color='red'>
			&lt;td>&lt;input name="users['y'].firstName" value="ccc" />&lt;/td>
			&lt;td>&lt;input name="users['y'].lastName" value="ddd" />&lt;/td></font>
		&lt;/tr>
		&lt;tr><font color='red'>
			&lt;td>&lt;input name="users['z'].firstName" value="eee" />&lt;/td>
			&lt;td>&lt;input name="users['z'].lastName" value="fff" />&lt;/td></font>
		&lt;/tr>
	&lt;/tbody>
	&lt;/table>
&lt;/form>
</pre>

## 数据绑定与表单标签库结合（表单不清空）★★★★★

* 数据绑定可以将***用户表单输入*** 绑定到一个***领域模型***，可以自动将HTTP请求参数默认的***字符串*** 类型转化成***不同类型*** 的对象属性，<font color='red'>***因此不再需要FORM类***</font>
* 数据绑定还有另外一个好处：当输入验证失败时，它会重新生成一个HTML表单。手工编写HTML代码时，必须***记着用户之前输入的值***，重新填充输入字段。有了Spring的数据绑定和表单标签后，它们会替你完成这些工作


第一步： 访问displayCustomerForm.do时创建一个领域对象并初始化属性，***作为表单的默认显示***，并添加到Model中，然后跳转到显示表单页面SignUpForm.jsp

<pre>
@RequestMapping(value = "displayCustomerForm", method = RequestMethod.GET)
public String displayCustomerForm(ModelMap model) {
	Customer customer = new Customer();
	customer.setName("qm");
	customer.setAge(2333);
	<font color='red'>model.addAttribute("customer", customer);</font>
	return "SignUpForm";
}
</pre>

第二步：使用表单标签库

* commandName定义了模型的名称，其对象属性将用于填充所生成的表单
* input标签中个path属性，将这个输入字段绑定到commandName指定对象的一个属性
* errors标签可以显示一个特定的字段错误（path="name"）或所有字段错误（path="*"）

<pre>
&lt;%@ taglib prefix="form" uri="http://www.springframework.org/tags/form"%>
&lt;form:form <font color='red'>commandName="product"</font> action="product_save" method="post">
	&lt;fieldset>
		&lt;legend>Add a product&lt;/legend>
		&lt;p class="errorLine">
			<font color='red'>&lt;form:errors path="name" cssClass="error"/></font>
		&lt;/p>
		&lt;p>
			&lt;label for="name">*Product Name: &lt;/label>
			<font color='red'>&lt;form:input id="name" path="name" tabindex="1"/></font>
		&lt;/p>
		&lt;p>
			&lt;label for="description">Description: &lt;/label>
			&lt;form:input id="description" path="description" tabindex="2"/>
		&lt;/p>
		&lt;p class="errorLine">
			&lt;form:errors path="price" cssClass="error"/>
		&lt;/p>
		&lt;p>
			&lt;label for="price">*Price: &lt;/label>
			&lt;form:input id="price" path="price" tabindex="3"/>
		&lt;/p>
		&lt;p class="errorLine">
			&lt;form:errors path="productionDate" cssClass="error"/>
		&lt;/p>
		&lt;p>
			&lt;label for="productionDate">*Production Date: &lt;/label>
			&lt;form:input id="productionDate" path="productionDate" tabindex="4"/>
		&lt;/p>
		&lt;p id="buttons">
			&lt;input id="reset" type="reset" tabindex="5">
			&lt;input id="submit" type="submit" tabindex="6" 
				value="Add Product">
		&lt;/p>
	&lt;/fieldset>
&lt;/form:form>
</pre>

## 转换器和格式化（麻烦，建议使用joda-time，参考ssm整合部分）

1、实现一种对象类型转换成另一种对象类型
2、Converter是通用元件，可以在应用程序的任意层中使用
3、Formatter则是专门为Web层设计的

### Converter

通过转换器，实现将输入的日期字符串转换成Date类型

* 实现Converter接口，源数据类型和目标数据类型可以自由指定

<pre>
public class StringToDateConvert <font color='red'>implements Converter<String,Date> </font>{
	private String datePattern;
	
	public StringToDateConvert(String datePattern){
	    this.datePattern = datePattern;
	}
	
	<font color='red'>public Date convert(String s)</font> {
	    SimpleDateFormat dateFormat = new SimpleDateFormat(datePattern);
	    dateFormat.setLenient(false);
	    try {
	        return dateFormat.parse(s);
	    } catch (ParseException e) {
	        throw new RuntimeException("格式错误");
	    }
	}
}
</pre>

* Spring MVC配置文件中编写一个conversionService bean，Bean的类必须为org.springframework.context.support.ConversionServiceFactoryBean，同时配置converters属性，它将列出程序中所有订制的Converter


	<bean id = "conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="cn.apeius.convert.StringToDateConvert">
                    <constructor-arg name="datePattern" value="yyyy-MM-dd"/>
                </bean>
            </set>
        </property>
    </bean>

* 给annotation-driven元素的conversion-service赋值bean名称，本例中是conversionService


	<mvc:annotation-driven conversion-service="conversionService"/>

* 表单输入


	<form action="/SpringMVC_study/test22.do">
		<fieldset>
		    <legend>用户登录</legend>
		    <p>
		        <label>出生日期：</label>
		        <input type="text" name="birthday"/>
		    </p>
		    <p id = "buttons">
		        <input type="submit" value="测试"/>
		    </p>
		</fieldset>
	</form>

* 字符串日期自动转换


	@RequestMapping(value = "/test22")
    	public ModelAndView test22(@RequestParam("birthday") Date birthday){
        	return new ModelAndView("hello","msg",birthday);
    }

### Formatter

1、Formatter的源类型必须是一个String，适合在Web层转换表单中用户的输入
2、Converter源类型可以是任意类型，可以在任意层中使用

以下案例为将一个表单输入的String日期转换成Date：

* 编写一个实现org.springframework.format.Formatter接口的Java类

<pre>
public class DateFormatter <font color='red'>implements Formatter<Date></font>{
    private String datePattern;
    private SimpleDateFormat simpleDateFormat;

    public DateFormatter(String datePattern){
        this.datePattern = datePattern;
        simpleDateFormat = new SimpleDateFormat(datePattern);
        simpleDateFormat.setLenient(false);
    }

	/*利用Locale将String解析成目标类型（Date）*/
    public Date <font color='red'>parse(String text, Locale locale)</font> throws ParseException {
        return simpleDateFormat.parse(text);
    }

	/*利用Locale将目标类型转换成String*/
    public String <font color='red'>print(Date date, Locale locale)</font> {
        return simpleDateFormat.format(date);
    }

}
</pre>

* 配置conversionService bean，bean的类名必须为org.springframework.format.support.FormattingConversionServiceFactoryBean，注入属性formatters，它将列出程序中所有订制的Formatter


	<!--配置conversionService bean-->
    <bean id = "conversionService"
          class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="formatters">
            <set>
                <bean class="cn.apeius.formatter.Date，它将列出程序中所有订制的Converter">
                    <constructor-arg name="datePattern" value="yyyy-MM-dd"/>
                </bean>
            </set>
        </property>
    </bean>

* 给annotation-driven元素的conversion-service赋值bean名称，本例中是conversionService


	<mvc:annotation-driven
        conversion-service="conversionService"/>

* 测试同Converter

### 用Registrar注册Formatter

注册Formatter的另一种方法是使用Registrar：

* 创建一个实现org.springframework.format.FormatterRegistrar接口的Java类


	public class MyFormatterRegistrar implements FormatterRegistrar{
	
	    private  String datePattern;
	
	    public MyFormatterRegistrar(String datePattern){
	        this.datePattern = datePattern;
	    }
	    public void registerFormatters(FormatterRegistry registry) {
	        registry.addFormatter(new DateFormatter(datePattern));
	        //注册更多的Formatter
	    }
	}

* 在配置文件中注册Registrar


	<!--配置conversionService bean-->
    <bean id = "conversionService"
          class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="formatterRegistrars">
            <set>
                <bean class="cn.apeius.formatter.MyFormatterRegistrar">
                    <constructor-arg name="datePattern" value="yyyy-MM-dd"/>
                </bean>
            </set>
        </property>
    </bean>

* 给annotation-driven元素的conversion-service赋值bean名称，本例中是conversionService

### 选择Convert还是Formatter

* Converter是一般工具，***可以将一种类型转换成另一种类型***，Converter既可用在Web层，也可用在其他层中
* Formatter只能将String转换成另一种类型，Formatter适用于web层，将表单属性进行类型转换
* 因此在Spring MVC程序中，选择Formatter比选择Converter更合适

## 使用joda-time注解对日期格式转换

SpringMVC默认不支持字符串转换成Date格式，可以采用SpringMVC自带的转换器，还有一种简单的方法：

注解可以加载javaBean上:

	@DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;

注解接在方法参数上：

	public void test(@DateTimeFormat(pattern = "yyyy-MM-dd") @RequestParam("date") Date date){
        System.out.println(date);
    }

需要导入依赖：

	<!--时间操作组件-->
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>${joda-time.version}</version>
    </dependency>

## 应用@Autowired和@Service进行依赖注入

* 将依赖注入到Spring MVC控制器的最简单方法是通过***注解@Autowired到字段或方法***
	  @Autowired
	  private ProductService productService;

* 能被作为依赖注入的类必须***声明@Service***，
	  @Service
	  public class ProductServiceImpl implements ProductService {

* 在配置文件中还需添加一个<component-scan/>元素来***扫描依赖基本包***
	  <context:component-scan base-package="cn.apeius.product5.service"/>

## 验证器Validator（JSR spring5.0不通过）！！！

如果一个程序中既有Formatter，又有Validator（验证器），那么在调用Controller期间，将会有一个或者多个<font color='red'>***Formatter*** </font>试图将字符串转成domain对象中个属性值，一旦转换成功，<font color='red'>***验证器*** </font>就会介入

### Spring验证器

#### 验证的Product对象

	public class Product implements Serializable {
	    private static final long serialVersionUID = 748392348L;
		private String name;
	    private String description;
	    private Float price;
	    private Date productionDate;
	
	    public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
	    public String getDescription() {
	        return description;
	    }
	    public void setDescription(String description) {
	        this.description = description;
	    }
	    public Float getPrice() {
	        return price;
	    }
	    public void setPrice(Float price) {
	        this.price = price;
	    }
	    public Date getProductionDate() {
	        return productionDate;
	    }
	    public void setProductionDate(Date productionDate) {
	        this.productionDate = productionDate;
	    }
	    
	}

#### 实现Validator接口

<pre>
<font color='red'>/*需要实现接口中supports和validate两个方法*/</font>
public class ProductValidator implements Validator {

	<font color='red'>/*如果验证器可以处理指定的Class，supports方法将返回true*/</font>
    public boolean supports(Class<?> klass) {
        return Product.class.isAssignableFrom(klass);
    }

	<font color='red'>/*validate方法验证目标对象，并将错误填入Errors对象*/</font>
    public void validate(Object target, Errors errors) {
        Product product = (Product) target;
		<font color='blue'>/*
		1、给Errors对象添加错误最容易的方法是调用Errors对象的一个reject或rejectValue方法，
		大多时候只传入一个错误码，Spring会在属性文件中查找错误码，获得相应的错误信息
			void reject(String errorcode)
			void rejectValue(String field, String errorCode)
		2、使用ValidationUtils类
			if(firstName == null || firstName.isEmpty()) errors.rejectValue("price","xxx")
			等效于 ValidationUtils.rejectIfEmpty("price");
		*/</font>
        ValidationUtils.rejectIfEmpty(errors, "name", "productname.required");
        ValidationUtils.rejectIfEmpty(errors, "price", "price.required");
        ValidationUtils.rejectIfEmpty(errors, "productionDate", "productiondate.required");
        
        Float price = product.getPrice();
        if (price != null && price < 0) {
            errors.rejectValue("price", "price.negative");
        }

        Date productionDate = product.getProductionDate();
        if (productionDate != null) {
            // The hour,minute,second components of productionDate are 0
            if (productionDate.after(new Date())) {
                errors.rejectValue("productionDate", "productiondate.invalid");
            }
        }
    }
}
</pre>

#### 错误信息属性文件

若想要从某个属性文件获取错误消息，则要通过声明messageSource bean告诉spring去那里查找这个文件

***messageSource bean***

	<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
        <property name="basename" value="/WEB-INF/resource/messages"/>
    </bean>

***messages.properties***

	productname.required.product.name=Please enter a product name
	price.required=Please enter a price
	productiondate.required=Please enter a production date
	productiondate.invalid=Invalid production date. Please ensure the production date is not later than today.
	price.negative=price should be positive


#### Controller类

<pre>
@Controller
public class ProductController {

    private static final Log logger = LogFactory
            .getLog(ProductController.class);

    @RequestMapping(value = "/product_input")
    public String inputProduct(Model model) {
        model.addAttribute("product", new Product());
        return "ProductForm";
    }

    @RequestMapping(value = "/product_save")
    public String saveProduct(@ModelAttribute Product product,
            <font color='red'>BindingResult bindingResult</font>, Model model) {<font color='red'>//BindingResult继承Errors接口</font>		
        logger.info("product_save");
		<font color='red'>/*实例化Vaildator类，调用其validate方法*/</font>
        ProductValidator productValidator = new ProductValidator();
        productValidator.validate(product, bindingResult);

		<font color='red'>/*检验该验证器是否生成错误消息，需在BindingResult中调用hasErrors方法*/</font>
        if (bindingResult.hasErrors()) {
            FieldError fieldError = bindingResult.getFieldError();
            logger.info("Code:" + fieldError.getCode() + ", field:"
                    + fieldError.getField());

            return "ProductForm";
        }

        // save product here

        model.addAttribute("product", product);
        return "ProductDetails";
    }
}
</pre>

### JSR 303验证

Hibernate Validator 是 Bean Validation 的参考实现。 Hibernate Validator 提供了 JSR 303 规范中所有内置 constraint 的实现，除此之外还有一些附加的 constraint。

#### 依赖

	<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-validator -->
	<dependency>
	    <groupId>org.hibernate</groupId>
	    <artifactId>hibernate-validator</artifactId>
	    <version>5.2.4.Final</version>
	</dependency>


#### Product类

Prodcut类中的name和productionDate字段用JSR 303标注类型进行标注

	public class Product implements Serializable {
	    private static final long serialVersionUID = 78L;
	
	    @Size(min=1, max=10)
	    private String name;
	    
	    private String description;
	    private Float price;
	    
	    @Past
	    private Date productionDate;
	
	    public String getName() {
	        return name;
	    }
	    public void setName(String name) {
	        this.name = name;
	    }
	    public String getDescription() {
	        return description;
	    }
	    public void setDescription(String description) {
	        this.description = description;
	    }
	    public Float getPrice() {
	        return price;
	    }
	    public void setPrice(Float price) {
	        this.price = price;
	    }
	    public Date getProductionDate() {
	        return productionDate;
	    }
	    public void setProductionDate(Date productionDate) {
	        this.productionDate = productionDate;
	    }
	
	} 


## Spring MVC 和 Struts2的区别

* Struts2的入口是filter，Spring MVC的入口是Servlet，两者的实现机制不同
* Struts2是基于类设计的，参数绑定到类的属性，所有的方法都可以使用；Spring MVC是基于方法设计的，参数绑定到方法的形参，并且只有一个方法可以使用
* Struts2必须是多例的（Struts2传递到的参数绑定到类的属性，如果是单例会导致不同用户数据的冲突），Spring MVC默认是单例的，所以Spring MVC的性能比Struts2好

## jsp 和 jstl 视图解析器

userList.jsp

	<%@ page language="java" contentType="text/html; charset=UTF-8"
	         pageEncoding="UTF-8"%>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
	<html>
	<head>
	    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
	    <title>Insert title here</title>
	</head>
	<body>
	<table border="1px" width="100%" cellspacing="0">
	    <tr>
	        <td> 用户名</td>
	        <td> 密码</td>
	    </tr>
	    <c:forEach items="${userList}" var="user">
	        <tr>
	            <td> ${user.userName} </td>
	            <td> ${user.password} </td>
	        </tr>
	    </c:forEach>
	</table>
	</body>
	</html>

controller代码

    @RequestMapping(value="/test12")
    public ModelAndView test12(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("userList");
        List<User> userList = new ArrayList<User>();
        for(int i = 0 ; i < 3 ; i ++){
            User user = new User();
            user.setUserName("user_name"+i);
            user.setPassword("123456");
            userList.add(user);
        }
        mv.addObject("userList", userList);
        return mv;
    }

## 使用ResponseBody输出JSON

* 在实际开发过程中json是最为常用的一种方式，所以Spring MVC提供了一种更为简便的方式输出数据，即使用@ResponseBody注解
* 需引入Jackson:Json处理工具包


### 使用步骤

导入依赖

	<dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.4.4</version>
    </dependency>

返回json数组

	@RequestMapping(value="/test13")
    @ResponseBody
    public List<User> test13(){
        List<User> userList = new ArrayList<User>();

        for(int i = 0 ; i < 3 ; i ++){
            User user = new User();
            user.setUserName("user_name"+i);
            user.setPassword("123456");
            userList.add(user);
        }

        return userList;
    }

返回单个json对象

	@RequestMapping(value="/test14")
    @ResponseBody
    public User test14(){
        User user = new User();
        user.setUserName("user_name");
        user.setPassword("123456");
        return user;
    }

### 原理

![](http://i.imgur.com/BFUifSh.png)

### 忽略某个JaveBean的属性

    // 密码
    @JsonIgnore
    private String password;

## @RequestBody

使用@RequestBody可以将请求的json字符串转化POJO对象

***传递json对象***

	@RequestMapping(value="/test15")
    @ResponseStatus(value= HttpStatus.OK)
    public void test15(@RequestBody User user){
        System.out.println(user);
    }

***传递json数组***

	@RequestMapping(value="/test16")
    @ResponseStatus(value= HttpStatus.OK)
    public void test16(@RequestBody List<User> users){
        for(User user : users){
            System.out.println(user);
        }
    }

## 文件上传

### 添加依赖

	<dependency>
		<groupId>commons-fileupload</groupId>
		<artifactId>commons-fileupload</artifactId>
		<version>1.3.1</version>
	</dependency>

### 定义文件上传解析器

在springmvc的配置文件中，去定义文件上传的解析器

	<!-- 定义文件上传解析器 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 设定默认编码 -->
        <property name="defaultEncoding" value="UTF-8"></property>
        <!-- 设定文件上传的最大值5MB，5*1024*1024 -->
        <property name="maxUploadSize" value="5242880"></property>
    </bean>

### 客户端编程

	<form method="post" action="/SpringMVC_study/upload" enctype="multipart/form-data">
        <p><input type="file" name="file"/></p>
        <p><input type="submit" value="提交"/></p>
    </form>

如果想要上传多个文件，在input元素中加入multiple属性：

	<input type="file" name="file" multiple/>

### MultipartFile接口

上传到Spring MVC程序的文件会被包在一个MultipartFile对象中，具有以下方法：

	byte[] getBytes() #以字节数组形式返回文件的内容
	String getContentType() # 返回文件的内容类型
	InputStream getInputStream() # 返回一个InputStream，从中读取文件的内容
	String getName() # 以多部分的形式返回参数的名称
	String getOriginalFilename() # 返回客户端本地驱动器中个初始文件名
	String getSize() # 以字为单位，返回文件的大小
	boolean isEmpty() # 表示被上传的文件是否为空
	void transferTo(File destination) # 将上传的文件保存到目标目录下

### 利用注解上传单文件

	@RequestMapping(value="/upload")
    public String upload(@RequestParam("file") MultipartFile multipartFile)
            throws Exception {
        if (multipartFile != null) {
            // multipartFile.getOriginalFilename() 获取文件的原始名称
            multipartFile.transferTo(new File("d:\\tmp\\" + multipartFile.getOriginalFilename()));
        }
        return "redirect:/success.html";

    }

### 利用注解上传多文件

	@RequestMapping(value="/uploadMultipartFile")
    public String uploadMultipartFile(@RequestParam("files") MultipartFile[] multipartFiles)
            throws Exception {
        for(MultipartFile multipartFile : multipartFiles){
            if (multipartFile != null) {
                // multipartFile.getOriginalFilename() 获取文件的原始名称
                multipartFile.transferTo(new File("d:\\tmp\\" + multipartFile.getOriginalFilename()));
            }
        }

        return "redirect:/success.html";

    }

### 利用domain类上传文件★★★★★

Product类中加入了新的属性 List<MultipartFile> images：

<pre>
public class Product implements Serializable {
    private static final long serialVersionUID = 74458L;

    @NotNull
    @Size(min=1, max=10)
    private String name;

	private String description;
    private Float price;
    <font color='red'>private List&lt;MultipartFile> images;</font>

    public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    public Float getPrice() {
        return price;
    }
    public void setPrice(Float price) {
        this.price = price;
    }
    public List&lt;MultipartFile> getImages() {
        return images;
    }
    public void setImages(List&lt;MultipartFile> images) {
        this.images = images;
    }
}
</pre>

控制器：

	@RequestMapping(value = "/product_save")
    public String saveProduct(HttpServletRequest servletRequest,
            @ModelAttribute Product product, BindingResult bindingResult,
            Model model) {

        List<MultipartFile> files = product.getImages();

        List<String> fileNames = new ArrayList<String>();

        if (null != files && files.size() > 0) {
            for (MultipartFile multipartFile : files) {

                String fileName = multipartFile.getOriginalFilename();
                fileNames.add(fileName);

                File imageFile = new File(servletRequest.getServletContext()
                        .getRealPath("/image"), fileName);
                try {
                    multipartFile.transferTo(imageFile);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

        // save product here
        model.addAttribute("product", product);
        return "ProductDetails";
    }

### 进度条

* HTML5 input元素的change事件，当input元素的值发生改变时，就会被触发
* HTML5 在XMLHttpRequest对象中添加的progress事件，当异步使用XMLHttpRequest对象上传文件时，就会持续地触发progress对象，直到上传进度完成或取消。通过监听progress事件，可以监控文件上传操作的进度

#### UploadFile的domain类

UploadFile类中包含一个MultipartFile属性，代表一个文件

	public class UploadedFile implements Serializable {

	    private static final long serialVersionUID = 72348L;	
	    private MultipartFile multipartFile;
	
	    public MultipartFile getMultipartFile() {
	        return multipartFile;
	    }

	    public void setMultipartFile(MultipartFile multipartFile) {
	        this.multipartFile = multipartFile;
	    }
	}

#### Html5FileUploadController类

<pre>
@Controller
public class Html5FileUploadController {

    private static final Log logger = LogFactory
            .getLog(Html5FileUploadController.class);

    @RequestMapping(value = "/html5")
    public String inputProduct() {
        return "Html5";
    }

    @RequestMapping(value = "/file_upload")
    public void saveFile(HttpServletRequest servletRequest,
            <font color='red'>@ModelAttribute UploadedFile uploadedFile,</font>
            BindingResult bindingResult, Model model) {

        MultipartFile multipartFile = uploadedFile.getMultipartFile();
        String fileName = multipartFile.getOriginalFilename();
        try {
            File file = new File(servletRequest.getSession().getServletContext().getRealPath("/file"), fileName);
            multipartFile.transferTo(file);
            System.out.println(file.getAbsolutePath());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
</pre>

#### html5.jsp

html5.jsp的用户界面主要包含了一个名为progressBar的div元素，一个表单和另一个名为debug的div元素。progressBar用于展示上传进度，debug用于展示调试信息，表单中有一个类型为file的input元素的一个按钮，有一些注意点：

* 标识为files的input元素，它有一个multiple属性，用于支持多文件选择
* 这个按钮不是一个提交按钮，单击它不会提交表单，脚本是利用XMLHttpRequest对象来完成上传的

<pre>
&lt;!DOCTYPE HTML>
&lt;html>
&lt;head>
&lt;script>
	<font color='red'>/*①
	* JavaScript代码执行的第一个件事是分配四个变量：
	* totalFileLength，totalUploaded，fileCount，filesUploaded。
	* totalFileLength表示要上传的文件总长度，totalUploaded表示目前已经上传的字节数，
	* fileCount表示上传的文件数量，filesUploaded表示已经上传的文件数量
	*/</font>
    var totalFileLength, totalUploaded, fileCount, filesUploaded;

    function debug(s) {
        var debug = document.getElementById('debug');
        if (debug) {
            debug.innerHTML = debug.innerHTML + '&lt;br/>' + s;
        }
    }

	<font color='red'>/*②
	* 启动时将files input元素的change事件映射到onFileSelect函数，
	* 从本地目录选择了不同的文件，就会触发change事件；
	* 将按钮的click事件映射到startUpload函数，点击就执行上传操作
	*/</font>
    window.onload = function() {
        document.getElementById('files').addEventListener(
                'change', onFileSelect, false);
        document.getElementById('uploadButton').
                addEventListener('click', startUpload, false);
    }

	<font color='red'>/*③
	* 每当用户选择本地目录不同文件时就会调用该函数，计算fileCount和totalFileLength	
	*/</font>
    function onFileSelect(e) {
        var files = e.target.files; // FileList object
        var output = [];
        fileCount = files.length;
        totalFileLength = 0;
        for (var i=0; i&lt;fileCount; i++) {
            var file = files[i];
            output.push(file.name, ' (',
                  file.size, ' bytes, ',
                  file.lastModifiedDate.toLocaleDateString(), ')'
            );
            output.push('&lt;br/>');
            debug('add ' + file.size);
            totalFileLength += file.size;
        }
        document.getElementById('selectedFiles').innerHTML = 
            output.join('');
        debug('totalFileLength:' + totalFileLength);
    }

	<font color='red'>/*④
	* 当用户调用Upload按钮时，就会调用startUpload函数，初始化totalUploaded和filesUploaded，
	* 随之调用uploadNext函数，上传下一个文件
	*/</font>
    function startUpload() {
        totalUploaded = filesUploaded = 0;
        uploadNext();
    }

	<font color='red'>/*⑤★★★★★
	* 首先创建一个XMLHttpRequest和FormData对象，并将接下来要上传的文件添加到它的后面，
	* 随后，uploadNext函数将XMLHttpRequest对象的progress事件添加到onUploadProgress函数，
	* 并将load事件和error事件分别添加到onUploadComplete和onUploadFailed
	* 接下来打开一个服务器连接，并发出FormData
	* 在上传期间，会重复得调用onUploadProgress函数，让它有机会更新进度条
	*/</font>
    function uploadNext() {
        var xhr = new XMLHttpRequest();
        var fd = new FormData();
        var file = document.getElementById('files').
                files[filesUploaded];
        fd.append("multipartFile", file);
        xhr.upload.addEventListener(
                "progress", onUploadProgress, false);
        xhr.addEventListener("load", onUploadComplete, false);
        xhr.addEventListener("error", onUploadFailed, false);
        xhr.open("POST", "file_upload");
        debug('uploading ' + file.name);
        xhr.send(fd);
    }

	<font color='red'>/*⑥
	* 在上传期间，会重复得调用onUploadProgress函数，让它有机会更新进度条
	* 更新包括计算已经上传的总字节数比率，与选择文件的总字节数，得到上传比率
	* 更新div元素的宽度
	*/</font>
    function onUploadProgress(e) {
        if (e.lengthComputable) {
            var percentComplete = parseInt(
                    (e.loaded + totalUploaded) * 100 
                    / totalFileLength);
            var bar = document.getElementById('bar');
            bar.style.width = percentComplete + '%';
            bar.innerHTML = percentComplete + ' % complete';
        } else {
            debug('unable to compute');
        }
    }

	<font color='red'>/*⑦
	* 上传完成时，调用onUploadComplete函数，这个事件处理函数会增加totalUploaded，
	* 即已经上传的文件容量，并添加filesUploaded
	* 如果所有文件已经上传完毕，弹出文件已经成功完成的提示
	* 否则再次调用uploadNext
	*/</font>
    function onUploadComplete(e) {
        totalUploaded += document.getElementById('files').
                files[filesUploaded].size;
        filesUploaded++;
        debug('complete ' + filesUploaded + " of " + fileCount);
        debug('totalUploaded: ' + totalUploaded);        
        if (filesUploaded &lt; fileCount) {
            uploadNext();
        } else {
            var bar = document.getElementById('bar');
            bar.style.width = '100%';
            bar.innerHTML = '100% complete';
            alert('Finished uploading file(s)');
        }
    }
    
    function onUploadFailed(e) {
        alert("Error uploading file");
    }   

&lt;/script>
&lt;/head>
&lt;body>
&lt;h1>Multiple file uploads with progress bar&lt;/h1>
&lt;div id='progressBar' style='height:20px;border:2px solid green'>
    &lt;div id='bar' 
            style='height:100%;background:#33dd33;width:0%'>
    &lt;/div>
&lt;/div>
&lt;form>
    &lt;input type="file" id="files" multiple/>
    &lt;br/>
    &lt;output id="selectedFiles">&lt;/output>
    &lt;input id="uploadButton" type="button" value="Upload"/>
&lt;/form>
&lt;div id='debug' 
    style='height:100px;border:2px solid green;overflow:auto'>
&lt;/div>
&lt;/body>
&lt;/html>
</pre>

![](http://i.imgur.com/eJ5uWnl.png)

## 文件下载

* 只要把图片或者HTML这样的静态资源放在***应用程序的目录下，或者放在应用程序目录的子目录下，而不是放在WEB-INF下***，Servlet、JSP容器就会将该资源发送到浏览器，在浏览器中打开正确的URL即可下载
* 有时候静态资源是***保存在应用程序目录外***，或者***保存在某一个数据库***，或者***有时候需要控制它的访问权限，方式其他网站交叉引用它***，必须通过编程发送资源到浏览器


### 文件下载概览

1. 对请求处理方法使用void返回类型（如果不需要页面跳转的话），并在方法中添加HttpServletResponse参数
2. 将响应的内容类型设为文件的内容类型，例如 response.setContentType("application/pdf")，如果不清楚内容类型，希望浏览器始终显示Save as对话框，则将它设为application/octet-stream
3. 添加一个名为Content-Disposition的HTTP响应标题，并赋值attachment; filename=fileName，这里的fileName是默认文件名，应该出现在File Download对话框中
4. 将文件发送到浏览器

### 范例1：隐藏资源

<pre>
@RequestMapping(value="/resource_download")
public String downloadResource(HttpSession session, HttpServletRequest request,
        <font color='red'>HttpServletResponse response</font>) {

	<font color='red'>/*
	* 判断用户是否登录
	*/</font>
    if (session == null || 
            session.getAttribute("loggedIn") == null) {
        return "LoginForm";
    }

	<font color='red'>/*
	* 判断文件是否存在，并将文件发送到浏览器
	*/</font>
    String dataDirectory = request.getSession().
            getServletContext().getRealPath("/WEB-INF/data");
    File file = new File(dataDirectory, "secret.pdf");

    if (file.exists()) {
        <font color='red'>response.setContentType("application/pdf");
        response.addHeader("Content-Disposition", 
                "attachment; filename=secret.pdf");</font>        
		<font color='blue'>byte[] buffer = new byte[1024];
        int len = 0;
        FileInputStream fis = null;
        OutputStream os = null;
        // if using Java 7, use try-with-resources
        try {
            fis = new FileInputStream(file);
            os = response.getOutputStream();
            while((len = fis.read(buffer)) != -1){
                os.write(buffer, 0, len);
            }
        } catch (IOException ex) {
            // do something, 
            // probably forward to an Error page
        } finally {
            if (os != null) {
                try {
                    os.close();
                } catch (IOException e) {
                }
            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (IOException e) {
                }
            }
        }</font>
    }
    return null;
}
</pre>

### 范例2：防止交叉引用

* 通过编程控制，是的只有当***refer标题中包含你的域名时*** 才发出资源，这样可以防盗链
* 但还是有办法下载到这些资源，但是绝对不会像以前那么容易得到

ImageController.java

* 如果直接在浏览器访问http://localhost:8080/SpringMVC_study/image_get/1返回404 not found
* 若加入注解 @RequestHeader String referer，直接访问，会导致调用getImage函数失败，根本无法进入函数体，也不进行refer是否为空的判断
* 通过images.html的超链接跳转，能访问到图片

<pre>
@Controller
public class ImageController {
	
	private static final Log logger = LogFactory.getLog(ImageController.class);
	
	@RequestMapping(value="/image_get/{id}", method = RequestMethod.GET)
	public void getImage(@PathVariable String id,
	        HttpServletRequest request, 
	        HttpServletResponse response,
            <font color='red'>@RequestHeader String referer</font>) {
        <font color='red'>if (referer != null) {</font>
            String imageDirectory = request.getSession().getServletContext().
                    getRealPath("/WEB-INF/image");
            File file = new File(imageDirectory, 
                    id + ".jpg");
            if (file.exists()) {
                response.setContentType("image/jpg");
                byte[] buffer = new byte[1024];
                int len = -1;
                FileInputStream fis = null;
                OutputStream os = null;
                // if you're using Java 7, use try-with-resources
                try {
                    fis = new FileInputStream(file);
                    os = response.getOutputStream();
                    while ((len = fis.read(buffer)) != -1) {
                        os.write(buffer, 0, len);
                    }
                } catch (IOException ex) {
                    // do something here
                } finally {
                    if (os != null) {
                        try {
                            os.close();
                        } catch (IOException e) {
                            
                        }
                    }
                    if (fis != null) {
                        try {
                            fis.close();
                        } catch (IOException e) {
                            
                        }
                    }
                }
            }
        }
	}
}
</pre>

images.html

	<a href="image_get/1">图片1</a>


## 重定向与转发的区别  

* 重定向发生在浏览器，由浏览器重新发出http请求，转发发生web服务器，请求在web服务器转发  
* 重定向可以到外部网站，重定向只能访问WEB应用个资源  

## 方法的返回值为string

* 如果方法的返回值是string类型，那么此时表示返回值是视图名称viewname
* 当进行转发时，没有数据的传递，不需要去书写ModelAndView，直接书写返回值是String

## 重定向的实现

* 在转发地址前加上redirect：完成重定向，<font color='red'>***不需要加应用名***</font>，直接写URI
* 转发到外部网站，例 `return "redirect:http://www.baidu.com";`


不带参数传递：

	@RequestMapping(value = "test19")
    public String test19(){
        System.out.println("19");
        return "redirect:/success.html";
    }

参数传递：

<pre>
<font color='red'>/*
* 1、重定向发生在浏览器，不能直接访问WEB-INF中的文件，书写的URL以/开头
* 2、需要传递的参数会自动拼接在url，本例中http://localhost:8080/SpringMVC_study/login.html?name=zhangsan
*/</font>
@RequestMapping(value = "/test23")
public ModelAndView test23(){
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("redirect:/login.html");
    modelAndView.addObject("name","zhangsan");
    return modelAndView;
}
</pre>

### 数据传递

* 使用重定向的一个不便是***无法轻松传值给目标页面***，而采用转发可以将属性添加到Model，使得目标视图可以轻松访问。重定向经过客户端，***Model中的一切都会在重定向时丢失***
* Spring 3.1版本以及更高版本可以通过Flash属性提供一种***重定向传值的方法***
* 使用Flash属性，必须在Spring MVC配置中有一个`<mvc:annotation-driven/>`，并且在方法上添加一个参数类型org.springframework.web.servlet.mvc.support.RedirectAttributes
* 使用方法：redirectAttributes.addFlashAttribute("message", "The product was successfully added.");可以在转发页面访问到这个属性

## 转发的实现★★★★★★

* ***Spring MVC中默认是转发***，但返回的值默认是试图名称
* ***如需要实现请求方法间跳转、页面跳转***，在试图名称之前添加forward: 要访问的路径


    @RequestMapping(value = "/test24")
    public ModelAndView test24(){
        System.out.println(24);
        ModelAndView mv = new ModelAndView();
        mv.setViewName("forward:/test25");
        return mv;
    }

    @RequestMapping(value = "/test25")
    public ModelAndView test25(){
        System.out.println(25);
        ModelAndView mv = new ModelAndView();
        mv.setViewName("forward:/login.html");
        return mv;
    }

### 数据传递

* 在每次调用请求方法时，都会创建Model类型的一个实例
* 若打算使用该实例，在请求方法参数中加入org.springframework.ui.Model参数，Spring MVC会在每一个请求方法被调用时创建一个***Model***实例，用于增加需要显示在视图中的属性
* 调用model.addAttribute来添加属性

## JSP 页面、静态资源url地址（绝对路径和相对路径）

### Java Web容器中项目部署时的访问路径

一般网站部署后，**访问路径是不带项目名称的**(为什么？一台主机的80端口就运行一个web程序，就没必要写项目名了)，比如最代码的服务器部署目录：/data/www/zuidaima/,在tomcat的conf/server.xml中host的访问配置是：

	<Host name="localhost"  appBase="webapps"
	            unpackWARs="false" autoDeploy="false"
	            xmlValidation="false" xmlNamespaceAware="false">
	   <Context docBase="/data/www/zuidaima/" path="/">
	</Host>

这样http的访问地址就是http://www.zuidaima.com/，
而在eclipse jee集成tomcat版本本地开发时，eclipse的配置中path的配置是带有项目路径的，所以访问的时候除了要有端口外，还得带上项目路径，比如：http://localhost:8080/zuidaima/

<font color='red'>**建议Path设置为空，这样本地debug时，所有访问路径和线上是一致的，不会出现线上访问404的情况**</font>

### mvc开发中view层中访问路径的问题

比如jsp中配置静态页面的地址：

	<link href="/resource/css/bootstrap.min.css" rel="stylesheet" />

则该文件在项目的本地目录则是：/data/www/zuidaima/resource/css/bootstrap.min.css，则其通过http访问是 http://www.zuidaima.com/resource/css/bootstrap.min.css 

其中/resource/css/bootstrap.min.css，**以/开头则表示是相对于项目根目录而言**，则本地访问中，根目录配置是： /data/www/zuidaima/ ，**而web网页http访问中根路径是 http://www.zuidaima.com/**

**但是如果出现 resource/css/bootstrap.min.css 的不以/开头的配置，则其访问路径是相对于当前访问目录而言的**，比如在首页，这样配置，所有文件都是可以访问的，因为首页当前目录就是/根目录，但是如果访问比如: http://www.zuidaima.com/user/2318804493993984.html ，这样访问就404错误，**http真实访问目录是**： http://www.zuidaima.com/user/resource/css/bootstrap.min.css ，这样对照到服务器资源明显就是错误的路径，所以出现这样的配置：

	<link href="../resource/css/bootstrap.min.css" rel="stylesheet" />

★★★此时仍相对于页面 http://www.zuidaima.com/user/2318804493993984.html 相当于 
http://www.zuidaima.com/user/../resource/css/bootstrap.min.css ，这样和
http://www.zuidaima.com/resource/css/bootstrap.min.css 是一个作用，是否有点豁然贯通了

所以建议在web开发中，尽量是用<font color='red'>**相对路径的根目录配置法**</font>(通过相对路径，定位到根目录)，这样一目了然，http访问路径和服务器配置路径是一一对应的，当然在很多情况下，静态资源和动态请求是分开域名提供服务的，比如最代码的css是： 
http://static.zuidaima.com/resource/css/bootstrap.min.css， 这样如果不在同一个域名那只能通过绝对路径访问了。

### ★★★URL访问出现404时的思路

1、静态页面引入资源的时候建议使用**相对路径的根目录配置法**，相对于访问该JSP的控制器URL，**其中"/"表示主机：端口**。比如访问控制器/login/getPage，此时访问的资源路径为/css/bind.css

    <link href="../css/bind.css" rel="stylesheet">

2、配置静态资源的路径，用 $lt;mvc:resources location="",mapping=""/>，相对根路径配置，**其中 "/" 表示主机：端口/应用名**

    <mvc:resources location="/WEB-INF/resources/js/" mapping="/js/**"/>
    <mvc:resources location="/WEB-INF/resources/css/" mapping="/css/**"/>
    <mvc:resources location="/WEB-INF/resources/image/" mapping="/image/**"/>

3、若相对路径出现问题，可以使用绝对路径 ${pageContext.request.contextPath}

	 <!--使用绝对路径的方式引入CSS文件-->
	<link rel="stylesheet"href="${pageContext.request.contextPath}/css/ueditor.css" type="text/css"/>
	<!--使用绝对路径的方式引入JavaScript脚本-->
	<script type="text/javascript"src="${pageContext.request.contextPath}/js/ueditor.config.js"></script>

## Spring MVC静态资源处理

优雅REST风格的资源URL不希望带 .html 或 .do 等后缀.由于早期的Spring MVC不能很好地处理静态资源，所以在web.xml中配置DispatcherServlet的请求映射，往往使用 *.do 、 *.xhtml等方式。这就决定了请求URL必须是一个带后缀的URL，而无法采用真正的REST风格的URL。

**如果将DispatcherServlet请求映射配置为"/"，则Spring MVC将捕获Web容器所有的请求，包括静态资源的请求，Spring MVC会将它们当成一个普通请求处理，因此找不到对应处理器将导致错误。**

如何让Spring框架能够捕获所有URL的请求，同时又将静态资源的请求转由Web容器处理，是可将DispatcherServlet的请求映射配置为"/"的前提。由于REST是Spring3.0最重要的功能之一，所以Spring团队很看重静态资源处理这项任务，给出了堪称经典的两种解决方案。

先调整web.xml中的DispatcherServlet的配置，使其可以捕获所有的请求：

    <servlet>
        <servlet-name>springMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

通过上面url-pattern的配置，所有URL请求都将被Spring MVC的DispatcherServlet截获。

### 采用 &lt;mvc:default-servlet-handler />

在 springMVC-servlet.xml 中配置 &lt;mvc:default-servlet-handler />后，会在Spring MVC上下文中定义一个 
org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler ，它会像一个检查员，对进入DispatcherServlet的URL进行筛查，**如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由DispatcherServlet继续处理。**

一般Web应用服务器默认的Servlet名称是"default"，因此DefaultServletHttpRequestHandler可以找到它。如果你所有的Web应用服务器的默认Servlet名称不是"default"，则需要通过default-servlet-name属性显示指定：

	<mvc:default-servlet-handler default-servlet-name="所使用的Web服务器默认使用的Servlet名称" />

**此时静态资源放在Web根目录下可以访问到**

### 采用 &lt;mvc:resources /> 推荐！

&lt;mvc:default-servlet-handler />将静态资源的处理经由Spring MVC框架交回Web应用服务器处理。**而&lt;mvc:resources />更进一步，由Spring MVC框架自己处理静态资源，并添加一些有用的附加值功能。**

首先，**&lt;mvc:resources />允许静态资源放在任何地方**，如WEB-INF目录下、类路径下等，你甚至可以将JavaScript等静态文件打到JAR包中。通过location属性指定静态资源的位置，由于location属性是Resources类型，因此可以使用诸如"classpath:"等的资源前缀指定资源位置。传统Web容器的静态资源只能放在Web容器的根路径下，&lt;mvc:resources />完全打破了这个限制。

其次，**&lt;mvc:resources />依据当前著名的Page Speed、YSlow等浏览器优化原则对静态资源提供优化。**你可以通过cacheSeconds属性指定静态资源在浏览器端的缓存时间，一般可将该时间设置为一年，以充分利用浏览器端的缓存。在输出静态资源时，会根据配置设置好响应报文头的Expires 和 Cache-Control值。

在接收到静态资源的获取请求时，会检查请求头的Last-Modified值，如果静态资源没有发生变化，则直接返回303相应状态码，提示客户端使用浏览器缓存的数据，而非将静态资源的内容输出到客户端，以充分节省带宽，提高程序性能。

在springMVC-servlet中添加如下配置：

	<mvc:resources location="/,classpath:/META-INF/publicResources/" mapping="/resources/**"/>

以上配置将Web根路径"/"及类路径下 /META-INF/publicResources/ 的目录映射为/resources路径。假设Web根路径下拥有images、js这两个资源目录,在images下面有bg.gif图片，在js下面有test.js文件，则可以通过 /resources/images/bg.gif 和 /resources/js/test.js 访问这二个静态资源。

假设WebRoot还拥有images/bg1.gif 及 js/test1.js，则也可以在网页中通过 /resources/images/bg1.gif 及 /resources/js/test1.js 进行引用。

配置案例:

    <mvc:resources location="/WEB-INF/resources/js/" mapping="/js/**"/>
    <mvc:resources location="/WEB-INF/resources/css/" mapping="/css/**"/>
    <mvc:resources location="/WEB-INF/resources/image/" mapping="/image/**"/>

## 其他注解

### ModelAttribute

每次调用请求响应方法时都会***创建Model类的一个实例***，可以***在方法中添加一个Model类型的参数***，也可以在方法中***添加ModelAttribute注解类型***<font color='red'> **来访问Model实例**</font>

#### 不加ModelAttribute

先看一个没有使用@ModelAttribute的Controller方法：

	@RequestMapping("/save")  
	public String save(User user) {  
	    user.setUsername("U love me");  
	    userService.save(user);  
	    return "result";  
	}  

等价于：

	@RequestMapping("/save")  
	public String save(Model model,int id,String username) {  
	    User user=new User();  
	    //这里是通过反射从request里面拿值再set到user  
	    user.setId(id);  
	    user.setUsername(username);  
	    model.addAttribute("user",user);  
	      
	    user.setUsername("U love me");  
	    userService.save(user);  
	    return "result";  
	}  

* 其中User包含id和username两个私有属性,含有公共setter和getter方法
* 执行此方法时会将key为"user"(注意:这里即使参数名称是user1，key一样还是"user")，value为user的对象加入到model


#### 用途一：

* 带ModelAttribute注解的参数会将对象添加到Model中
* <font color='red'>***与不加ModelAttribute的区别：***</font>带ModelAttribute注解会先从model去获取key为"user"的对象，如果获取不到会通过反射实例化一个User对象，再从request里面拿值set到这个对象，然后把这个User对象添加到model(其中key为"user").
使用了@ModelAttribute可修改这个key，不一定是"user"，此情况下，***用与不用@ModelAttribute没有区别***

<pre>
<font color='red'>/*
本例中将用newOrder键值将Order实例添加到Model对象中；
如果为定义键值，则键值将使用对象类型的名字，即用键值order将Order实例添加到Model中*/</font>
public String summitOrder(@ModelAttribute("newOrder") Order order){
	
}
</pre>

#### 用途二：

* 标注一个<font color='red'>***非请求的处理方法***</font>，Spring MVC会每次在调用***请求处理方法之前*** 调用带@ModelAttribute注解的方法
* @ModelAttribute注解的方法可以***返回一个对象或一个void类型***，如果返回一个对象，则返回对象会自动加到Model中，如未指定键值，键值为对象类型的名字
* 若返回类型为void，若需要将对象放入Model中，则必须添加一个Model类型的参数，并自行将实例添加到Model中


	@ModelAttribute
    public String test23(){
        return new String("qm");
    }

    @RequestMapping
    public String test24(Model model){
        Map<String, Object> map = model.asMap();
        for(Map.Entry<String,Object> entry:map.entrySet()){
            System.out.println(entry.getKey() + " " + entry.getValue());
        }
        return "forward:/WEB-INF/views/hello.jsp";
    }

## 国际化

在这个全球化的时代，编写能够支持***不同语言的国家和地区的应用程序*** 越来越重要

### 国际化应用程序的方式

国际化应用程序的具体方式取决于有多少静态数据***需要以不同的语言显示出来***，这里有两种方式：

1. 如果大量数据是静态的，就要针对每一个语言区域单独创建一个资源版本
2. 如果静态数据有限，可以将文本元素，***如元件标签和错误消息隔离成为文本***，每个文本文件保存着一个语言区域的译文，随后应用程序会自动获取每一个元素

### 语言区域 Locale

java.util.Locale类表示一个语言区域，一个Locale对象包含3个主要原件：language、country、variant

* language是最主要的部分
* 语言本身不能区分一个语言区域，比如讲英语的国家很多，但不同国家讲的英语有区别
* variant是一个特定于供应商或特定于浏览器的代号，例如用WIN代表Windows

#### 构造器

	Locale(String language)
	Locale(String language, String country)
	Locale(String language, String country, String variant)

创建一个中国所用的中文Locale对象

	Locale locale = new Locale("zh","CN");

利用Locale类的静态方法来创建Local对象

	Locale locale = Locale.CHINA;

利用getDefault方法返回计算机的语言区域

	Locale locale = Locale.getDefault();

#### ResourceBundle读取属性文件

区域属性文件值

国际化和本地化应用程序时，需要具备以下条件：

	1. 将文本文件隔离成属性文件
	2. 选择和读取正确的属性文件

1、将文本文件隔离成属性文件，可以利用如下工具：http://javawind.net/tools/native2ascii.jsp?action=transform ，给出以下两个语言区域属性文件，文件的格式为：
	
	basename_languageCode_countryCode

MyResources_en_US.properties

	greetings=hello

MyResources_zh_CN.properties

	greetings=\u4f60\u597d


2、使用ResourceBundle类来读取属性文件中的值

	ResourceBundle resourceBundle = ResourceBundle.getBundle("MyResources", Locale.SIMPLIFIED_CHINESE);
    System.out.println(resourceBundle.getString("greetings"));//你好

    resourceBundle = ResourceBundle.getBundle("MyResources", Locale.US);
    System.out.println(resourceBundle.getString("greetings"));//hello

3、在Spring MVC中，不直接使用ResourceBundle，而是利用messageSourceBean告诉Spring MVC要将属性文件保存在哪里

### 国际化Spring MVC应用程序

在Spring MVC中，不直接使用ResourceBundle，而是利用messageSource bean告诉Spring MVC要将属性文件保存在哪里

#### 配置 messageSource bean

<font color='red'>***利用messageSource bean告诉Spring MVC要将属性文件保存在哪里***</font>

<pre>
&lt;bean id="messageSource"
	class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
	<font color='red'>//用两个基准名设置basenames的属性</font>
	&lt;property name="basenames" >
		&lt;list>
			&lt;value>/WEB-INF/resource/messages&lt;/value>
			&lt;value>/WEB-INF/resource/labels&lt;/value>
		&lt;/list>
	&lt;/property>
&lt;/bean>
</pre>

![](http://i.imgur.com/1T6Betq.png)

说明，上面定义的bean的类class有两种实现方式：

* 一种是ReloadableResourceBundleMessageSource，提供了定时刷新功能，***允许在不重启系统的情况下***，更新资源的信息；***它在应用程序目录下搜索这些属性文件***，即WEB目录下搜索
* 另一种是ResourceBundleMessageSource，它是不能重新加载的，如果在任意属性文件中修改了某一个属性的key或者value，那么要使修改生效，***就必须重启JVM***；属性文件必须放在***类路径*** 下，即src目录下

还有一个说明，如果只有一组属性文件，则可以用basename属性代替basenames，像下面这样：

	<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
        <property name="basename" value="resource/messages"/>
    </bean>

#### 配置 语言区域解析器bean

在Spring MVC中选择语言区域，可以使用语言区域解析器bean，它有几个实现，其中AcceptHeaderLocaleResolver是其中最容易使用的一个

	<bean id="localeResolver"
          class="org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver">
    </bean>

#### 使用message标签

使用message标签，要在使用该标签的JSP页面声明这个taglib指令：

	<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>

使用message标签：

	<label for="description"><spring:message code="label.description"/>: </label>

## 拦截器

* HandlerExecutionChain是一个执行链，从HandlerMapping返回给DispatcherServlet
* 其中包含Handler对象和Interceptor（拦截器）对象
* SpringMVC的拦截器定义了三个方法
	* preHandler：调用handler之前执行
	* postHandler：调用handler之后执行
	* afterCompletion：视图渲染完之后执行

### 拦截器执行过程

![](http://i.imgur.com/z7vVxv6.png)

* 在执行Handler前会经过多个拦截器
* 每个拦截器的前置方法会按照拦截器的顺序依次执行
* 每个拦截器的后置方法会从后向前执行
* 每个拦截器的afterCompletion方法会从后往前执行

### 编写自定义拦截器

<pre>
public class MyInterceptor implements HandlerInterceptor {

    <font color='red'>//前置方法，如果返回值是false，后面的拦截器不会执行；如果返回true，执行后面的拦截器</font>
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("前置方法");
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("后置方法");
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("完成方法");
    }
}
</pre>

### 配置拦截器

	<mvc:interceptors>
        <mvc:interceptor>
            <!--配置连接器的路径，/**表示表示拦截所有的请求-->
            <mvc:mapping path="/**"/>
            <!--拦截器的路径-->
            <bean class="cn.apeius.springmvc.interceptor.MyInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>

### 配置多个拦截器

	<mvc:interceptors>
        <mvc:interceptor>
            <!--配置连接器的路径，/**表示表示拦截所有的请求-->
            <mvc:mapping path="/**"/>
            <!--拦截器的路径-->
            <bean class="cn.apeius.springmvc.interceptor.MyInterceptor1"></bean>
        </mvc:interceptor>
        <mvc:interceptor>
            <!--配置连接器的路径，/**表示表示拦截所有的请求-->
            <mvc:mapping path="/**"/>
            <!--拦截器的路径-->
            <bean class="cn.apeius.springmvc.interceptor.MyInterceptor2"></bean>
        </mvc:interceptor>
    </mvc:interceptors>

若配置1，2，3，4多个拦截器，如果3的前置方法返回false，则：
* 拦截器4的所有方法都不会执行
* 拦截器1，2的完成方法仍会去执行
* 1，2，3的后置方法不会执行

## 总结

![](http://i.imgur.com/6ge6LQ1.png)
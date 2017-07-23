---
title: Tomcat笔记

date: 2016-07-06 10:14:00

categories:
- Tomcat

tags:
- Tomcat
- JavaEE

---
## Tomcat的目录结构文件

![](http://i.imgur.com/Dp2gPgS.png)

    bin: 启动和关闭tomcat的bat文件
    conf: 配置文件 
    	-->server.xml : 该文件用于配置和 server 相关的信息, 比如 tomcat启动端口后,配置Host,  配置Context 即web应用 
    	-->web.xml : 该文件配置与 web应用(web应用就相当于是一个 web站点)
    	-->tomcat-users.xml: 该文件用户配置tomcat 的用户密码 和 权限
    lib目录: 该目录放置运行tomcat 运行需要的jar包
    logs目录：存放日志, 当我们需要去查看日志的时候，很有用!,当我们启动tomcat错误时候，可以查询信息.
    webapps目录: 该目录下，放置我们的web应用(web 站点)
    work: 工作目录: 该目录用于存放jsp被访问后 生成的对应的 server文件 和.class文件

## 首页面设置及目录规范结构

![](http://i.imgur.com/UJt4wAM.png)

*配置访问首目录*

    ①在web文件夹下配置WEB-INF文件夹
    ②在 web.xml 文件中添加配置的代码:
      <welcome-file-list>
        <welcome-file>hello1.html</welcome-file>
      </welcome-file-list>
    ③通过http://localhost:8088/web1来访问hello1.html

*目录结构*

    web-inf目录下的 classes目录将来是存放 class文件
    lib 目录将来时存放 jar文件
    web.xml 配置当前这个web应用的信息.

## Tomcat体系

![](http://i.imgur.com/7cYZp67.png)

***理解服务、引擎、host（主机）、Web应用[Context]概念***

- 查看Tomcat中的server.xml文件，最外面节点为Server，Service下一个节点服务`<Service name = "Catalina">`，所以我们有时把Tomcat服务叫做Catalina服务。
- 服务下有引擎（engine）`<Engine name="Catalina" defaultHost="localhost">`，它可以管理多个主机host。
- 主机下有多个Web应用[Context]
- 与引擎并列的有连接器（connector），有支持不同类型协议（http、https等）的连接器。不同协议使用不同的连接器    

## 配置默认主机

　　当我们在浏览器中输入http://127.0.0.1:8080/web应用，引擎是如何知道用户要访问那个`主机`下的web应用？答：配置一个默认主机

	在tomcat/conf/server.xml 文件
	<Engine name="Catalina" defaultHost="主机名">
	
	默认是localhost，即在webapps目录下，查找web应用
	<Host name="localhost"  appBase="webapps"
	            unpackWARs="true" autoDeploy="true"
	            xmlValidation="false" xmlNamespaceAware="false">

## 配置域名[host]

*步骤：*

	(1) 在C:\WINDOWS\system32\drivers\etc 下的host文件 添加127.0.0.1 www.sina.com.cn[浏览器向Tomcat发送请求，但Tomcat认为自己不是www.sina.com，可以拒绝，所以需添加主机名]
	(2) 在tomcat 的server.xml文件添加主机名 
	<Host name="www.sina.com" appBase="d:\web3”>
		<Context path="/" docBase="d:\web3" />
	</Host>
	(3) 在d:\web3 加入了一个 /WEB-INF/web.xml 把 hello2.html设为首页面
	如果连端口都不希望带，则可以吧tomcat的启动端口设为80即可.
	(4) 重启生效

## CATALINA_BASE与CATALINA_HOME的区别

* CATALINA_HOME是Tomcat的***安装目录***，CATALINA_BASE是Tomcat的***工作目录***
* 如果我们想要运行Tomcat的多个实例，但是不想安装多个Tomcat软件副本。那么我们可以配置多个工作目录，每个运行实例独占一个工作目录，但是共享同一个安装目录
* Tomcat每个运行实例需要使用自己的conf、logs、temp、webapps、work和shared目录，因此CATALINA_BASE就指向这些目录。 而其他目录主要包括了Tomcat的二进制文件和脚本，CATALINA_HOME就指向这些目录

## Tomcat管理虚拟目录[context]

*需求*

当我们把 web 应用放到 webapps目录，tomcat会自动管理，如果我们希望tomcat可以管理其它目录下的web应用?　　-> 虚拟目录配置

假设我在d盘有一个web应用文件夹web2
	
*步骤*

    1、找到server.xml文件
    2、编辑host节点，在Host节点中添加Context
        在server.xml中添加：<Context path="/myweb2" docBase="d:\web2"/>
        myweb2：是访问时输入的web名,实际取出的是web2中的资源（★★★★url与文件夹有映射关系）
        "d:\web2"：web应用文件夹在计算机中绝对路径
    实际访问时输入的地址：http://localhost:8088/myweb2/hello2.html
    

## 定义上下文

### 显式定义

1、在Tomcat的conf/Catalina/localhost目录下创建一个XML文件。例如把一个commerce.xml文件放在conf/Catalina/localhost目录下，那么应用程序的上下文路径就是commerce，下面给出一个范例：

	<Context docBase="C://apps/commerce" reloadable="true"/>

docBase是必要的属性，用来定义应用程序的位置；reloadable属性是可选的，如果值为true，一旦应用程序中Java类文件或者其他资源有增加、减少或者更新，Tomcat都会侦测到，就会重新加载应用程序

2、在Tomcat的conf/server.xml文件中添加一个Context元素[同Tomcat管理虚拟目录]

	<Host name="localhost"  appBase="webapps"
	            unpackWARs="true" autoDeploy="true"
	            xmlValidation="false" xmlNamespaceAware="false">
		
		<Context path = "/commerce" docBase="C:/apps/commerce" reloadable="true" /> 
	
	</Host>

### 隐式定义

通过将一个war文件或者整个应用程序复制到Tomcat的webapps目录下，就可以隐式地部署应用程序
    






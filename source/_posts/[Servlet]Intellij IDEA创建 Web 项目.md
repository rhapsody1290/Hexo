---
title: Intellij IDEA创建 Web 项目

date: 2017-04-13 19:56:00

categories:
- Servlet

tags:
- Servlet
- IDEA

---

## 快速构建 Web 项目

打开IDEA，新建Project，左边菜单栏选择 Maven，直接点 Next

![](http://i.imgur.com/VKZroBL.png)

选择GroupId和ArtifactId

![](http://i.imgur.com/89qQ4Tx.png)

选择项目名称，默认会填上工程位置、模块姓名等，直接点Finsh

![](http://i.imgur.com/1B1iP1d.png)

进入到工程的主页面，注意到右上角跳出是否导入Maven，一般直接选择Auto_Import，这样在POM文件中添加依赖的时候，可以直接导入jar包

![4](http://i.imgur.com/nC0KdOK.png)

现在已经完成普通Maven项目的创建，接下来将要把他变成一个Web项目，只需要在工程名字上右键，选择Add Frame Support

![5](http://i.imgur.com/caLOyrH.png)

选择Web Application，并勾选创建Web.xml

![6](http://i.imgur.com/nhGocyN.png)

在工程目录上看到新添加了一个web的文件夹，而且注意到文件夹的图标和普通文件夹的图标不一样，这个文件夹是web的根目录

![7](http://i.imgur.com/gwXjx1M.png)

但是根据Maven规范，定义web根目录默认是在main文件夹下，而且以webapp命名。所以下图所示，将web文件夹重命名为webapp，而且移动到main文件夹下

![8](http://i.imgur.com/tMcsc6w.png)

打开pom.xml文件，指定工程的打包方式为war，当把该文件放到web容器下（可以是tomcat，也可以是jetty等）直接可以运行web程序

在build文件下，加入 Jetty 插件，个人觉得使用Jetty插件的好处是可以在不下载外部服务器的情况下，直接在IDEA中运行程序



	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	
	    <groupId>com.qianmingxs</groupId>
	    <artifactId>Demo</artifactId>
	    <version>1.0-SNAPSHOT</version>
	    <packaging>war</packaging>
	
	    <build>
	        <plugins>
	            <plugin>
	                <groupId>org.mortbay.jetty</groupId>
	                <artifactId>maven-jetty-plugin</artifactId>
	                <version>6.1.21</version>
	                <configuration>
	                    <!-- 多少秒进行一次热部署 -->
	                    <scanIntervalSeconds>10</scanIntervalSeconds>
	                    <!--端口-->
	                    <connectors>
	                        <connector implementation="org.mortbay.jetty.nio.SelectChannelConnector">
	                            <port>8080</port>
	                            <maxIdleTime>60000</maxIdleTime>
	                        </connector>
	                    </connectors>
	                    <scanIntervalSeconds>10</scanIntervalSeconds>
	                    <!--根目录，建议/-->
	                    <contextPath>/</contextPath>
	                </configuration>
	            </plugin>
	        </plugins>
	    </build>
	</project>


![9](http://i.imgur.com/5rB4b99.png)

打开IDEA中的Maven操作窗口，Plugins-jetty-jetty:run，直接双击就可以启动jetty容器，运行web程序，从控制台也可以看到应用程序在8080端口启动

如果想要进入**调试模式**，在jetty：run上右键选择 Debug 即可

![10](http://i.imgur.com/cMLRzrz.png)

在浏览器上输入localhost:8080，可以看到程序正常运行；如果在jetty文件中设置的contextPath不为path，运行的url应为localhost:8080/YourContextpath/

![11](http://i.imgur.com/ZtFol1O.png)

jetty：run-war和jetty：run功能差不多，在运行jetty：run-war的时候会先将web工程打包，然后执行war。

![12](http://i.imgur.com/pDTb6B7.png)

打包后的war在功能目录中target目录中

![13](http://i.imgur.com/cTzERfo.png)

## 使用Tomcat插件

在插件中加入tomcat插件即可

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	
	    <groupId>com.qianmingxs</groupId>
	    <artifactId>Demo</artifactId>
	    <version>1.0-SNAPSHOT</version>
	    <packaging>war</packaging>
	
	    <build>
	        <plugins>
	            <plugin>
	                <groupId>org.codehaus.mojo</groupId>
	                <artifactId>tomcat-maven-plugin</artifactId>
	                <version>1.1</version>
	                <configuration>
	                    <path>/</path>
	                    <port>8080</port>
	                    <uriEncoding>UTF-8</uriEncoding>
	                    <server>tomcat6</server>
	                </configuration>
	            </plugin>
	        </plugins>
	    </build>
	</project>

简要说明一下：

path是访问应用的路径
port是 tomcat 的端口号
uriEncoding URL按UTF-8进行编码，这样就解决了中文参数乱码。
Server指定tomcat名称

## IDEA 引入自定义jar包

之后可以在 Project Structure 的 Modules - Dependencies看到添加的lib文件，或者可以在Project Structure - Libraries看到添加的lib文件

移除lib文件需要在上述两个位置中进行移除

![](http://i.imgur.com/AUPqNZY.png)

## war打包工具

可以向用户自定义的文件打包到web目录下，下面给出了一个将用户自定义的lib文件打包到WEB-INF目录下

	<!--war打包-->
	<plugin>
	    <groupId>org.apache.maven.plugins</groupId>
	    <artifactId>maven-war-plugin</artifactId>
	    <version>2.1.1</version>
	    <configuration>
	        <webResources>
	            <resource>
	                <directory>${project.basedir}/lib</directory>
	                <targetPath>WEB-INF/lib</targetPath>
	                <filtering>false</filtering>
	                <includes>
	                    <include>**/*.jar</include>
	                </includes>
	            </resource>
	        </webResources>
	    </configuration>
	</plugin>
---
title: Maven基础POM配置讲解

date: 2016-08-18 00:00:00

categories:
- Maven

tags:
- Maven

---

## Maven坐标

### maven坐标的主要组成

    groupId：定义当前maven项目属于哪个项目
    artifactId：定义实际项目中的某一个模块
    version：定义当前项目的当前版本
    packaging：定义当前项目的打包方式 jar、war

    根据这些坐标，在maven库中可以找到唯一的jar包
	其中groupId、artifactId、version是必须定义的，packaging是可选的（默认是jar）

### Maven模块概念

1、Maven中，一个项目会被划分成很多模块，比如org.SpringFramework项目，对应的Maven模块有很多：spring-core、spring-context

2、groudId不应该只对应到公司（组织）的名称，而应定义到项目名

## POM.xml

	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
	         http://maven.apache.org/maven-v4_0_0.xsd">
	
	    <modelVersion>4.0.0</modelVersion> #这个是POM的版本号，现在都是4.0.0 的，必须得有，但不需要修改。
	
	    <groupId>com.smart</groupId> #定义当前maven项目属于哪个项目
	    <artifactId>smart-demo</artifactId> #定义实际项目中的某一个模块
	    <version>1.0</version> #定义当前项目的当前版本
	    <packaging>war</packaging> #定义当前项目的打包方式
	
	    <name>smart-demo Maven Webapp</name>
	    <url>http://maven.apache.org</url> #name、ur表示该项目的名称与 URL 地址，意义不大，可以省略。
	
	    <dependencies> #定义该项目的依赖关系
	        <dependency>
	            <groupId>junit</groupId>
	            <artifactId>junit</artifactId>
	            <version>3.8.1</version>
	            <scope>test</scope>
	        </dependency>
	    </dependencies>
	
	    <build> #表示与构建相关的配置，这里的 finalName 表示最终构建后的名称 smart-demo.war，这里的 finalName 还可以使用另一种方式来定义
	        <finalName>smart-demo</finalName>
	    </build>
	
	</project>

## 树形图表示POM.xml

![](http://i.imgur.com/4VMt6qa.png)

可见，除了***项目的基本信息***（***Maven 坐标***、***打包方式***等）以外，每个 pom.xml 都应该包括：

1. Lifecycle（生命周期）
2. Plugins（插件）
3. Dependencies（依赖）

Lifecycle 是项目构建的生命周期，它包括 9 个 Phase（阶段）。

大家知道，Maven 是一个核心加上多个插件的架构，而这些插件提供了一系列非常重要的功能，这些插件会在许多阶段里发挥重要作用。

![](http://i.imgur.com/8YUbAxw.png)

## 依赖Scope（作用域）★★★★★

我们可以在 pom.xml 中定义一些列的项目依赖（构件包），每个构件包都会有一个 Scope（作用域），它表示该构件包在什么时候起作用，包括以下五种：

1. compile：默认作用域，在编译、测试、运行时有效

2. test：对于测试时有效

3. runtime：对于测试、运行时有效

4. provided：对于编译、测试时有效，但在运行时无效（jar包在编译运行时有效，但在发布时由容器提供，不需要发布）

5. system：与 provided 类似，但依赖于系统资源

可以一张表格表示：

![](http://i.imgur.com/v8GTA8V.png)

## POM模版

	<dependencies>
        <!-- JUnit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <!-- MySQL -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.25</version>
            <scope>runtime</scope>
        </dependency>
        <!-- Servlet -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>
        <!-- Spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>4.0.3.RELEASE</version>
        </dependency>

        <!-- AspectJ -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.7.4</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.7.4</version>
        </dependency>

        <!-- Hibernate4 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>4.3.5.Final</version>
        </dependency>
        <!-- for JPA, use hibernate-entitymanager instead of hibernate-core -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>4.3.5.Final</version>
        </dependency>
        <!-- 以下可选 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-envers</artifactId>
            <version>4.3.5.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-c3p0</artifactId>
            <version>4.3.5.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-proxool</artifactId>
            <version>4.3.5.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-infinispan</artifactId>
            <version>4.3.5.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-ehcache</artifactId>
            <version>4.3.5.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-commons-annotations</artifactId>
            <version>3.2.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate.javax.persistence</groupId>
            <artifactId>hibernate-jpa-2.1-api</artifactId>
            <version>1.0.0.Final</version>
        </dependency>
        <dependency>
            <groupId>org.jboss</groupId>
            <artifactId>jandex</artifactId>
            <version>1.1.0.Final</version>
        </dependency>
        <!-- 为了让Hibernate使用代理模式，需要javassist -->
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.18.1-GA</version>
        </dependency>

        <dependency>
            <groupId>org.jboss.logging</groupId>
            <artifactId>jboss-logging</artifactId>
            <version>3.1.3.GA</version>
        </dependency>
        <dependency>
            <groupId>org.jboss.spec.javax.annotation</groupId>
            <artifactId>jboss-annotations-api_1.2_spec</artifactId>
            <version>1.0.0.Final</version>
        </dependency>


        <dependency>
            <groupId>antlr</groupId>
            <artifactId>antlr</artifactId>
            <version>2.7.7</version>
        </dependency>
        <dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
        </dependency>

        <dependency>
            <groupId>commons-collections</groupId>
            <artifactId>commons-collections</artifactId>
            <version>3.1</version>
        </dependency>
        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>commons-pool</groupId>
            <artifactId>commons-pool</artifactId>
            <version>1.4</version>
        </dependency>

        <dependency>
            <groupId>javax.transaction</groupId>
            <artifactId>jta</artifactId>
            <version>1.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.6.1</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate.javax.persistence</groupId>
            <artifactId>hibernate-jpa-2.0-api</artifactId>
            <version>1.0.0.Final</version>
        </dependency>

        <!-- tomcat7.0.35 数据库连接池 -->
        <dependency>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>tomcat-dbcp</artifactId>
            <version>7.0.35</version>
        </dependency>
    </dependencies>


## 参考文献

Maven 那点事儿
http://my.oschina.net/huangyong/blog/194583




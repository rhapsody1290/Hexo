---
title: Maven聚合与继承

date: 2017-02-09 22:13:00

categories:
- Maven

tags:
- Maven

---

## 聚合

什么是聚合？聚合就是**把多个模块或项目聚合到一起**，简单统一的完成编译等工作

为了能够使用一条命令就能构建 account-email 和 account-persist两个模块，我们需要建立一个额外的名为 account-aggregator 的模块，然后通过该模块构建整个项目的所有模块。 account-aggregator本身也是个 Maven 项目，它的 POM如下

<pre>
&lt;project>  
	&lt;modelVersion>4.0.0&lt;/modelVersion>  
	&lt;groupId>com.juvenxu.mvnbook.account&lt;/groupId>  
	&lt;artifactId>account-aggregator&lt;/artifactId>  
	&lt;version>1.0.0-SNAPSHOT&lt;/version>  
	<font color='red'>&lt;packaging> pom &lt;/packaging>  </font>
	&lt;name>Account Aggregator&lt;/name>  
	 &lt;modules>  
		&lt;module>account-email&lt;/module>  
		&lt;module>account-persist&lt;/module>  
	 &lt;/modules>  
&lt;/project> 
</pre>

注意：packaging的类型为pom ，module的值是一个以当前POM为主目录的相对路径。

## 继承

在POM中申明一些配置供子POM继承，以实现"一处申明，多处使用的"目的

在account-aggregator下创建一个account-parent的子目录，然后在该子目录下创建除account-aggregator模块之外的模块的父模块。为此在该子目录下创建一个pom.xml文件如下：

	<?xml version="1.0" encoding="UTF-8"?>  
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">  
			<modelVersion>4.0.0</modelVersion> 
 
	        <groupId>com.juvenxu.mvnbook.account</groupId>  
	        <artifactId>account-parent</artifactId>  
	        <version>1.0.0-SNAPSHOT</version>  

	        <packaging>pom</packaging>  

	        <properties>
	                <springframework.version>2.5.6</springframework.version>
	        </properties>
	
	        <dependencyManagement>
	                <dependencies>
	                       <dependency>
	                            <groupId>org.springframework </groupId>
	                            <artifactId>spring-core</artifactId>
	                            <version>${springframework.version}</version>
	                       </dependency>
	
	                       ......
	
	                </dependencies>
	        </dependencyManagement>
	</project> 

需要注意的是它的packaging的值必须为pom，这一点与模块聚合一样，作为父模块的POM，其打包类型也必须为pom。由于父模块只是为了消除配置的重复，因此也就**不需要src/main/java等目录了**

有了父模块就让其他子模块来继承它，修改account-email的pom文件如下：

	<?xml version="1.0" encoding="UTF-8"?>  
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">  
	<modelVersion>4.0.0</modelVersion> 
	        <parent>
	            <groupId>com.juvenxu.mvnbook.account</groupId>  
	            <artifactId>account-parent</artifactId>  
	            <version>1.0.0-SNAPSHOT</version>  
	            <relativePath>../account-parent/pom.xml</relativePath>  
	        </parent> 
	        <artifactId>account-email</artifactId> 
	        <name>Account Email</name> 
	
	        <dependencies>
	                <dependency>
	                    <groupId>org.springframework</groupId>
	                    <artifactId>spring-core</artifactId>
	                </dependency>
	                
	                ......
	                
	        </dependencies>
	
	        <build>
	                <plugins>
	                    ......
	                </plugins>
	        </build>
	
	</project>

**只配置了groupId和artifactId,省去了version**，如果父模块配置了依赖的scope也是可以省略的。这些信息可以省略是因为account-email继承了account-parent中的dependencyManagement配置，完整的依赖声明已经包含在父POM中了，子模块只需简单的配置groupId和artifactId。

使用这种以来管理机制虽然不能减少太多的配置项，但是经过别人实践后强烈推荐的方法。如果子模块不声明依赖的使用，即使该依赖已经在父POM文件dependencyManagement中声明了，也不会产生任何实际的效果。
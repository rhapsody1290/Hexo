---
title: 利用Maven下载Jar包

date: 2017-05-05 11:08:00

categories:
- Maven

tags:
- Maven

---

在任意目录下新建一个批处理文件 exec.bat

	call mvn -f pom.xml dependency:copy-dependencies
	@pause

然后再同目录下新建一个pom.xml文件

	<?xml version="1.0"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <groupId>temp.download</groupId>
	    <artifactId>temp-download</artifactId>
	    <version>1.0-SNAPSHOT</version> 
	    <dependencies>
			<dependency>
				<groupId>com.alibaba</groupId>
				<artifactId>fastjson</artifactId>
				<version>1.2.7</version>
			</dependency>
	    </dependencies>
	</project>

每次只要修改`<dependencies></dependencies>`中的内容就行了

点击运行后，会自动下载到Maven仓库中，而且在同级目录下的target文件也会有下载完成的Jar包

完整目录结构如下：

![](http://i.imgur.com/X6ZfaGj.png)

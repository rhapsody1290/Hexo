---
title: Maven安装及基础功能

date: 2016-08-18 00:00:00

categories:
- Maven

tags:
- Maven

---
## Maven作用★★★★★

* 统一开发规范与工具：项目目录结构，配置文件、单元测试代码位置；项目构建工具，自动完成那个编译、测试、打包等工作
* 统一管理 jar 包：Maven 中央仓库下载jar包

## 项目构建过程

Maven是一个跨平台的项目管理工具，主要用于基于java平台的项目构建，依赖管理。

如图为项目构建的过程

 ![这里写图片描述](http://img.blog.csdn.net/20160308115250226)


## Maven的安装与配置

### Maven的安装

***Jdk的情况***

    Jdk必须1.6以上的版本

***从官网下载maven***

    从http://maven.apache.org/官网上下载最新版本的maven

***设定path路径***

    1、配置M2_HOME的环境变量，新建一个系统变量：M2_HOME，路径是maven的安装目录
    2、配置path环境变量，在path值的末尾添加"%M2_HOME%\bin"

***建库***
    
    打开conf文件夹下的settings.xml文件，找到第53行，把注释去掉，修改成：<localRepository>D:\Maven\repo</localRepository>，当然了，前提是在某个路径下，手动建立了一个名为 repo的文件夹，然后把本地仓库指向该路径。
    
***利用命令行检查是否成功***

    mvn -v


### maven的约定

    src/main/java        存放项目的java文件
    src/main/resources   存放项目的资源文件，如spring，hibernate的配置文件
	src/test/java        存放所有的测试的java文件
	src/test/resources   存放测试用的资源文件
	target               项目输出位置
	pom.xml              项目配置文件

### maven的命令

```
mvn archetype:create ：创建 Maven 项目
mvn compile ：编译源代码
mvn test-compile ：编译测试代码
mvn test ： 运行应用程序中的单元测试
mvn site ： 生成项目相关信息的网站
mvn clean ：清除目标目录中的生成结果
mvn package ： 依据项目生成 jar 文件
mvn install ：在本地 Repository 中安装 jar
mvn eclipse:eclipse ：生成 Eclipse 项目文件
mvn -Dmaven.test.skip=true : 忽略测试文档编译
```

## maven项目快速入门

### hello项目

* 在myeclipse建立一个项目Hello，删除自动生成的src文件，建立4个Source Folder文件夹，名字如图所示：
　　　　![列表内容](http://img.blog.csdn.net/20160308191157113)
* 创建一个包cn.itcast.maven，并在该包下创建一个***类*** Hello

```
public class Hello{
    public void hello(){
        System.out.println("say hello");
    }
}
```

* 在src/test/java中创建一个包cn.itcast.maven，创建一个***测试类*** HelloTest，测试类中调用Hello类

```
public class HelloTest{
    public void testHello(){
        Hello hello = new Hello();
        hello.hello();
    }
}
```

* 编辑pom.xml文件

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>cn.itcast.maven</groupId>
	<artifactId>Hello</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>Hello</name>
	
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.9</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
</project>
```

* 用maven命令编译项目

```
mvn compile

命令行出现BUILD SUCCESS，表明编译成功
```

* target文件夹的变化，可以看到编译后的文件全部放入到了target里。	

![列表内容](http://img.blog.csdn.net/20160310104705856)

* clean，执行命令**mvn clean**,可以看到target的目录没有了。
* test，执行**mvn test**命令，自动生成测试报告

![这里写图片描述](http://img.blog.csdn.net/20160308192143954)

    说明：
    target/classes 存放编译后的类
    target/test-classes 存放编译后的测试类
    target/surefire-reports 存放测试报告

* package，执行mvn package，完成打包工作
	 
![这里写图片描述](http://img.blog.csdn.net/20160308192628080)

    说明：
    target/classes 编译后的类的路径
    target/test-classes 编译后的测试类的路径
    target/surefire-reports 测试报告
    target/maven-archiver 执行package的归档
    Hello-0.0.1-SNAPSHOT.jar 执行完package命令后打成的jar包

### Hellofriend项目

* 建立HelloFriend项目工程

![这里写图片描述](http://img.blog.csdn.net/20160310110111565)

* 建立cn.itcast.maven包及HelloFriend类

![这里写图片描述](http://img.blog.csdn.net/20160310111901071)

* 编辑HelloFriend类，引用之前编写的Hello类

```
public class HelloFriend {
	public void helloFriend(){
		Hello hello = new Hello();
		hello.hello();
	}
}
```

* 编写pom.xml文件

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>cn.itcast.maven</groupId>
	<artifactId>HelloFriend</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>HelloFriend</name>
	
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.9</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>cn.itcast.maven</groupId>
			<artifactId>Hello</artifactId>
			<version>0.0.1-SNAPSHOT</version>
			<scope>compile</scope>
		</dependency>
	</dependencies>
	
</project>
```

* 执行mvn compile命令

    执行这个命令的时候会出错，因为HelloFriend项目是建立在Hello项目基础之上的，但是现在工程中没有引入Hello.java这个类。所以会出错。

* 执行mvn clean install命令
    
    1、	打开命令行
    2、	把当前路径调节到Hello工程的根目录
    3、	执行mvn clean install命令，把Hello整个工程放入到仓库中

如果执行成功，则会在仓库中看到

![这里写图片描述](http://img.blog.csdn.net/20160310111201493)

* 执行mvn package命令打包HelloFriend工程
    
    可以看到成功以后，在target目录下多了一个jar包
    该jar包为当前工程的jar包。

* 建立cn.itcast.maven包和测试类HelloFriendTest类

![列表内容](http://img.blog.csdn.net/20160310112454683)

* 编辑HelloFriendTest类

```
public class testHeloFriend {

	public void testHelloFriend(){
		HelloFriend helloFriend = new HelloFriend();
		helloFriend.helloFriend();
	}
}

```

* 执行mvn package命令

    上图中的”say hello”就是输出的结果。

## maven的核心概念

### 项目对象模型

![这里写图片描述](http://img.blog.csdn.net/20160310173933529)
    
    说明：
    maven根据pom.xml文件，把它转化成项目对象模型(POM)，这个时候要解析依赖关系，然后去相对应的maven库中查找到依赖的jar包。
    在clean，compile，test，package等阶段都有相应的Plug-in来做这些事情。
    而这些plug-in会产生一些中间产物。

### 插件的位置

    在maven解压后的位置D:\Maven\apache-maven-3.0.5-bin\apache-maven-3.0.5有一个bin文件夹，里面有一个文件m2.config文件  

	set maven.home default ${user.home}/m2，其中该路径指明了仓库的存储位置。


其中settings.xml文件中,

	<localRepository>D:/Maven/repo</localRepository>

这个说明了仓库中的位置。

	D:\Maven\repo\org\apache\maven\plugins


这里的插件就是执行maven的各种命令所需要的插件。

### maven坐标

maven坐标的主要组成

    groupId：定义当前maven项目属于哪个项目
    artifactId：定义实际项目中的某一个模块
    version：定义当前项目的当前版本
    packaging：定义当前项目的打包方式

    根据这些坐标，在maven库中可以找到唯一的jar包

### 依赖管理

    具体案例在3 Maven项目。
    1、项目HelloSuperFriend依赖项目HelloFriend，项目HelloFriend依赖项目Hello。
    2、在HelloSuperFriend依赖项中加入
        <dependency>
            <groupId>cn.itcast.maven</groupId>
            <artifactId>HelloFriend</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    3、能够编译通过，体现了依赖的传递性

### 继承管理★★★★★
    
    现有一A项目，B、C项目同时依赖A项目，这是需要用到继承

1、创建一个项目ParentJunit

![这里写图片描述](http://img.blog.csdn.net/20160314203938379)
　　　　
2、ParentJunit添加一个Junit依赖

![这里写图片描述](http://img.blog.csdn.net/20160314204137520)

3、新建一个项目：HelloJunit，编写POM文件

```
<parent>
    <groupId>cn.itcast.maven</groupId>
    <artifactId>ParentJunit</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</parent>
```

4、只需要继承ParentJunit，所以junit就被引入到HelloJunit中了

### 仓库管理

    可以根据maven坐标定义每一个jar包在仓库中的存储位置。
    大致为：groupId/artifactId/version/
    仓库的分类：
    1、本地仓库
        ~/.m2/repository/，每一个用户也可以拥有一个本地仓库
    2、远程仓库
        2.1 中央仓库：Maven默认的远程仓库，http://repo1.maven.org/maven2
        2.2 私服：是一种特殊的远程仓库，它是架设在局域网内的仓库
        2.3 镜像：用来替代中央仓库，速度一般比中央仓库快

jar查找顺序
![这里写图片描述](http://img.blog.csdn.net/20160314211004957)

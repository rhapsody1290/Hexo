---
title: Maven 添加本地 jar 包

date: 2017-05-07 23:08:00

categories:
- Maven

tags:
- Maven

---

## 方法一：将jar包安装到本地repository中

```
mvn install:install-file -Dfile=my-jar.jar -DgroupId=org.richard -DartifactId=my-jar -Dversion=1.0 -Dpackaging=jar
```

## 方法二：

添加scope为system的依赖，解决编译、运行问题

	<dependency>
		<groupId>com.microsoft.sqlserver</groupId>
		<artifactId>sqljdbc</artifactId>
		<version>3.0</version>
		<scope>system</scope>
		<systemPath>${project.basedir}/lib/sqljdbc-3.0.jar</systemPath>
	</dependency>

打包时将自定义的jar包复制到web/lib下

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

方法三：建立自己的仓库

	http://blog.csdn.net/chagaostu/article/details/50056427
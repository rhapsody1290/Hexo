---
title: Maven插件

date: 2017-04-13 21:49:00

categories:
- Maven

tags:
- Maven

---

## war打包

war打包时添加第三方的jar包，如下图所示将lib目录下的jar包作为依赖

![](http://i.imgur.com/NMiYvVs.png)

配置如下：

    <build>
        <plugins>
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
        </plugins>
    </build>

引入本地依赖：

	<dependency>
	    <groupId>com.microsoft.sqlserver</groupId>
	    <artifactId>sqljdbc</artifactId>
	    <version>3.0</version>
	    <scope>system</scope>
	    <systemPath>${project.basedir}/lib/sqljdbc-3.0.jar</systemPath>
	</dependency>
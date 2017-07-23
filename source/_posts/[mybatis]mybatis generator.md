---
title: mybatis generator

date: 2017-04-15 12:39:00

categories:
- mybatis

tags:
- generator

---
详细说明
http://blog.csdn.net/isea533/article/details/42102297

## generatorConfig.xml
	
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE generatorConfiguration
	        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
	        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
	<generatorConfiguration>
	
	    <!-- 指定数据连接驱动jar地址 -->
	    <classPathEntry location="D:\IDEA\workspace\mk_wechat_app\app\lib\sqljdbc-3.0.jar" />
	
	    <!-- 一个数据库一个contex，使用flat，一个表对应一个实体类 -->
	    <context id="Tables" targetRuntime="MyBatis3" defaultModelType="flat">
	
	        <!-- 注释 -->
	        <commentGenerator>
	            <property name="suppressAllComments" value="true" /><!-- 是否取消注释 -->
	            <property name="suppressDate" value="true" /> <!-- 是否生成注释代时间戳 -->
	        </commentGenerator>
	
	        <!--定义如何连接目标数据库-->
	        <jdbcConnection driverClass="com.microsoft.sqlserver.jdbc.SQLServerDriver"
	                        connectionURL="jdbc:sqlserver://121.12.155.196:1433;databaseName=DBERPDG"
	                        userId="sa"
	                        password="1qaz@WSX">
	        </jdbcConnection>
	
	        <!-- 生成实体类地址 -->
	        <javaModelGenerator targetPackage="com.mk.domain" targetProject="src/main/java">
	            <!--是否使用子包-->
	            <property name="enableSubPackages" value="false" />
	            <!--任何字符串属性的setter方法将调用trim方法-->
	            <property name="trimStrings" value="true" />
	        </javaModelGenerator>
	
	        <!-- 生成xml文件 -->
	        <sqlMapGenerator targetPackage="mapper"  targetProject="src/main/resources">
	            <property name="enableSubPackages" value="false" />
	        </sqlMapGenerator>
	
	        <!--DAO-->
	        <javaClientGenerator type="XMLMAPPER" targetPackage="com.mk.mapper"
	                             targetProject="src/main/java">
	            <property name="enableSubPackages" value="false" />
	        </javaClientGenerator>
	
	        <!--tableName：指定要生成的表名，可以使用SQL通配符匹配多个表。
	        例如要生成全部的表，可以按如下配置：<table tableName="%" />-->
	        <table tableName="Partner_Info_Tab" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false">
	            <!--Cloudscape、DB2、DB2_MF、Derby、HSQLDB、Informix、MySql、SqlServer、SYBASE-->
	            <generatedKey column="Partner_No" sqlStatement="SqlServer"/>
	        </table>
	
	    </context>
	</generatorConfiguration>

## pom

	<build>
	    <plugins>
	        <!--Mybatis Generator插件-->
	        <plugin>
	            <groupId>org.mybatis.generator</groupId>
	            <artifactId>mybatis-generator-maven-plugin</artifactId>
	            <version>1.3.0</version>
	        </plugin>
	
	    </plugins>
	</build>



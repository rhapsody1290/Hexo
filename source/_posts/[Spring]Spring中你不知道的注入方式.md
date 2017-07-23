---
title: Spring中你不知道的注入方式

date: 2016-11-10 11:44:00

categories:
- Spring

tags:
- Spring

---

## 前言

在Spring配置文件中使用XML文件进行配置，实际上是让Spring执行了相应的代码，例如：

* 使用<bean>元素，实际上是让Spring执行无参或有参构造器
* 使用<property>元素，实际上是让Spring执行一次setter方法

但Java程序还可能有其他类型的语句：调用getter方法、调用普通方法、访问类或对象的Field等，而Spring也为这种语句提供了对应的配置语法：

* 调用getter方法：使用PropertyPathFactoryBean
* 调用类或对象的Filed值：使用FiledRetrievingFactoryBean
* 调用普通方法：使用MethodInvokingFactoryBean

https://my.oschina.net/itblog/blog/206481
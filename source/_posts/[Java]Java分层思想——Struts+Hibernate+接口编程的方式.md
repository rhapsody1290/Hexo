---
title: Java分层思想——Struts+Hibernate+接口编程的方式

date: 2016-08-22 00:00:00

categories:
- Java

tags:
- 接口编程
- Struts
- Hibernate
- Java

---
## 接口编程

* 通过接口(使用动态代理)，就达到web层和业务层的解耦（业务层代码改动，web层不需要重写，达到解耦）
* 数据库有两张表，分别是users表和message表，对应的在业务层有Users对象、UsersService对象；Message对象、MessageService对象
* 在web层定义UsersServiceInter接口，业务层实现类UsersServiceImp；同理定义MessageServiceInter解耦，业务层实现类MessageServiceImp
* 定义一个基础接口BaseServiceInter，把一些通用的方法直接定义到该接口内，子接口继承
* BaseService类实现BaseServiceInter接口，包含基础方法

## 留言板工程

### github
https://github.com/rhapsody1290/NoteBook

### 程序框架图

![](http://i.imgur.com/DP9tp0e.png)

### 工程目录

![](http://i.imgur.com/WCtAkwy.png)
---
title: Linux视频教程第19讲.补充Linux重要内容find alias

date: 2017-04-13 10:37:00

categories:
- Linux

tags:
- Linux

---

## 运行级别

init [0123456]，指定系统运行级别，类似windows的正常运行模式或安全模式

```
0：关机 
1：单用户
2：多用户状态没有网络服务 
3：多用户状态有网络服务 
4：系统未使用保留给用户 
5：图形界面 
6：系统重启
```

常用运行级别是3和5，要修改默认的运行级别可改文件

```
/etc/inittab的id:5:initdefault:
```
这一行中的数字

##常用命令

### **ln 建立符号连接(和windows的快捷方式)**
```
a) ln –s 源目标
b) ln –s /etc/inittab inittab #（inittab指向实际文件/etc/inittab）
```
### **搜索文件及目录——find**

在Linux中，因为文件系统是以级别式的结构组成的，所以要在整个系统中找到特定的文件和目录并不是件容易的事。而“find”命令可以解决上述问题。 

1、在特定的目录下搜索并显示指定名称的文件和目录

```
find /root/ -name Hello.java    #查找文件
```
2、搜索一段时间内被存取/变更的文件或目录

```
find /home –amin -10 十分钟内存取的文件和目录 
find /home -amin +10 十分钟前存取的文件和目录
find /home –atime -10 十小时内存取的文件和目录 
find /home –cmin -10 十分钟内更改过的文件和目录 
find /home –ctime -10 十小时前更改过的文件和目录 
find /home –size +10k 意思是说查找/home目录下大小为10k的文件
```

### **shell使用**

shell脚本文件：

<pre>
a) 是一个文本文件
b) 命令的集合 
c) 有执行的权限 
d) 执行方式（./文件名）
</pre>

**profile文件**

profile文件主要是用于配置环境变量

![这里写图片描述](http://img.blog.csdn.net/20151227194314714)

**用户登录后自动执行的shell脚本文件**

配置 .bashrc 文件可以指定某些程序在用户登录的时候就自动启动。

```
如：/home/tomcat/bin/startup.sh start
```

## 别名

a) 命令：alias显示系统当前定义的所有alias
b)去别名

```
alias llh='ls -l /home'
alias lm = '-al | more'
```

c)去别名

```
ualias lm
```



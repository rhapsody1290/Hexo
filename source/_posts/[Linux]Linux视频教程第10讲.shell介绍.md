---
title: Linux视频教程第10讲.shell介绍

date: 2017-04-12 22:21:00

categories:
- Linux

tags:
- Linux

---

## 概述

每个人在成功登陆linux后，系统会出现不同的提示符号，例如$、~、#等，然后你就可以开始输入需要的命令，若是命令正确，系统就会依据命令的要求来执行，直到注销系统为止；在登录到注销期间，**输入的每个命令都会经过解释及执行。而这个负责的机制就是shell**

## shell编程

其实作为命令语言互动式地解释和执行用户输入的命令只是shell功能的一个方面。shell还可以用来进行程序设计。它提供了定义变量和参数的手段以及丰富的程序控制结构。使用shell编程类似于DOS中批处理文件，称为shell script，又叫shell程序或shell命令文件

![这里写图片描述](http://img.blog.csdn.net/20151224102611018)

## 书籍推荐

《Linux命令、编辑器和shell编程》

## shell的分类

1. shell有很多种类，常用的有如下几种：
a) /bin/ash
b) /bin/bash 
c) /bin/tcsh-----csh 
d) /bin/ksh----欧洲常用
e) /bin/sh----中国常用 

2. 查看电脑有多少个shell:
ls -l /bin/*sh

3. **查看目前使用的是哪种SHELL**
a) env | grep "SHELL"
b) echo $SHELL
c) `echo $0`
4. 修改其它的SHELL
chsh -s 输入新的SHELL（/bin/csh） 
5. 注销下再重新登录，使用 env 
6. 不同的SHELL 可能有不同的命令 
7. SHELL命令补全功能（TAB）
输入MK，再按两下TAB，出现两头两个字母为MK的命令。cat p再按两个TAB ，会出现开头字母为p的文件或字母

## shell的使用

命令历史和互动：用上下箭头键可以重复以前所输入的命令

命令完成功能：用tab键能自动完成相关命令，再次按tab可得到清单

shell命令历史记录

```
history
```

## shell脚本文件

是一个文本文件，为一系列命令的集合，有执行的权限，**执行方式（./文件名）**

用户登录后自动执行的shell脚本文件 **.bashrc**  位于主目录下，

系统的脚本`/etc/bashrc`，里面是基本配置数据

**配置.bashrc文件可以指定某些程序在用户登录的时候就自动启动**

.bash_profile 位于主目录下，它为系统的每个用户设置环境信息
**/etc/ 中 profile 主要是配置环境变量**

用export可以临时加入一个系统路径，如

```
export PATH=$PATH:$HOME/bin:/root/test/t1，
```

输出环境PATH，引用原来的值`$PATH`，`$HOME`表示工作主目录，:是路径分隔符

总结：

	/etc/profile，/etc/bashrc 是系统全局环境变量设定
	~/.profile，~/.bashrc用户家目录下的私有环境变量设定

**已经定义好的环境变量**

SHELL：默认shell PATH：路径
USER：当前登录用户的用户名

**显示变量内容**

```
echo $SHELL echo $USER
echo $PATH
```

**shell通配符**

【案例】export NAME=Michael -

```
echo Welcome $NAME, the date is date
```

**单引号**：不处理任何变量和命令

【案例】echo ‘Welcome $NAME, the date is date ’ 

**双引号**：处理变量但不处理命令

【案例】echo “Welcome $NAME, the date is date “ 

**反引号**：把引号中的每个单词作为一个命令，如果是变量则先求值然后作为一个命令处理

【案例】echo “Welcome $NAME, the date is `date` “ 

**别名**

命令：alias显示系统当前定义的所有alias

【案例】alias cp=’cp -i’ 
【案例】alias li=’ls –l –color=tty’
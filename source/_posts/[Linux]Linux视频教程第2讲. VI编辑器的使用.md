---
title: Linux视频教程第2讲. VI编辑器的使用

date: 2017-04-12 20:59:00

categories:
- Linux

tags:
- Linux

---

## 什么是vi编辑器

vi编辑器是linux下最有名的编辑器，也是我们学习linux必须掌握的工具，在linux下也可使用vi进行程序的开发，如java程序，c程序。

ps：VI编辑器由Bill Joy 1976年在bsd unix 开发的（世界第一骇客，成为了自由软件协会）

vi分三种模式，分别是**一般模式、编辑模式与指令列命令模式**
        
一般模式：打开文档默认进入，可以移动光标，删除字符，删除整行，也可以复制粘帖。

编辑模式：按下i，I，o，O，a，A，r，R进入编辑模式，按Esc退出编辑模式。

指令命令模式：输入[:/?]三种任何一个按钮，就可以将光标移到最底下那一行，这个模式可以提供搜寻资料，读取、存盘，大量取代字符，离开vi，显示行号等动作。

## 如何使用vi进行开发？

**Java程序**

在linux下使用vi开发一个简单的java程序Hello.java，并且在linux下运行成功

1. vi Hello.java
2. 输入i，进入到插入模式 
3. 输入Esc键，进入命令模式
4. 输入冒号:[wq 表示保存退出，q!表示退出不保存] 
5. 编译javac Hello.java 
6. 运行java Hello

**c程序**
1. gcc Hello.cpp
2. gcc -o Hello Hello.cpp[参数o表示可自定义生成的out文件名，否则默认为a. out，重复写会覆盖以前的值][自己命名gcc -o my1 Hello.cpp]
3. ./Hello


---
title: 命令行编译Java程序

date: 2016-08-09 00:00:00

categories:
- Java

tags:
- Java
- 命令行

---
先说下.java和.class文件，.java文件可以放在任何位置，只要运行javac时，指定你要编译的源文件位置就好，然后javac还有另一个参数是指定.class文件输出到哪个目录。另外javac还有一个option叫 -d，表示编译出来的.class文件，按照包名放进相应层级的目录中去。
所以，假设你的.java文件在D:\code\src\com\company\Hello.java，（因为Java要求源文件中的public类与源文件名要相同，类名通常首字母大写，所以源文件通常是Hello.java这样的），并且假如你的Hello.java里写明package com.company;
假如你当前置身于D:\code\目录下，执行javac .\src\com\company\Hello.java -d .\bin\
表示编译.\src\com\company\Hello.java，将编译好的.class文件按包名层级结构放到.\bin\目录下。
于是在D:\code\bin\com\company\下会出现一个Hello.class文件。这时，运行Hello时，跟.java文件就没有关系了。
这时如果你置身于D:\code\bin\目录下，最好了，因为我们通常配置环境变量时，CLASSPATH会配置.，也就是当前路径，那你就在D:\code\bin\下执行java，那class也就在当前路径找了：
java com.company.Hello，这里java是启动虚拟机的命令，启动时运行哪个类呢？com.company.Hello，这是类名，带着包名，叫全限定名，也就是类的全名，这跟它在哪个目录下没关系，是类名，虚拟机会在CLASSPATH下，也就是当前目录下，去找com.company.Hello这个类，并加载运行。
至于它怎么找，你就不用管了。
如果你当前正置身于D:\code\下，那你执行java com.company.Hello就失败了，因为虚拟机在CLASSPATH下找不到这个类。那怎么办呢，就要告诉虚拟机，去哪找，也就是要指定CLASSPATH，可以执行：java -classpath .\bin com.company.Hello
不知道你能不能理解，你要运行的是com.company.Hello这个类，这是类名，不是目录名，去哪找这个类？用-classpath告诉虚拟机，所以，不能java .\bin\com.company.Hello，这就不伦不类了。

参考
http://bbs.csdn.net/topics/390865848
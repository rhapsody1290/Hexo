---
title: Linux视频教程第20讲.Linux启动过程分析

date: 2017-04-13 10:44:00

categories:
- Linux

tags:
- Linux

---

BIOS就是在开机的时候，计算机系统会主动执行的第一个程序了！

接下来 BIOS 会去分析计算机里面有哪些储存设备，我们以硬盘为例，BIOS 会依据使用者的设定去获得**能够开机的硬盘**，并且到该硬盘里面去读取**第一个扇区的 MBR 位置**。 MBR 这个仅有 446 bytes 的硬盘容量里面会放置最基本的**开机管理程序**，此时 BIOS就功成圆满，而接下来就是 MBR 内的开机管理程序的工作了。

这个开机管理程序的目的是在**加载(load)核心档案**，由于开机管理程序是操作系统在安装的时候所提供的，所以他会认识硬盘内的文件系统格式，因此就能够读取核心档案， 然后接下来就是核心档案的工作，开机管理程序也功成圆满，之后就是大家所知道的操作系统的任务啦！

简单的说，整个开机流程到操作系统之前的动作应该是这样的：

1、BIOS：开机主动执行的程序，会认识第一个可开机的装置；
2、MBR：第一个可开机装置的第一个扇区内的主要启动记录区块，内含开机管理程序；
3、开机管理程序(boot loader)：一支可读取核心档案来执行的软件；
4、核心档案：开始操作系统的功能...

## linux启动过程

linux系统的启动过程如下： 

```
a) BIOS自检 
b) 启动GRUB/LILO 
c) 运行linux内核并检测硬件
d) 运行系统的第一个进程init
e) init读取系统引导配置文件/etc/inittab中的信息进行初始化 
f) /etc/rc.d/rc.sysinit系统初始化脚本
g) /etc/rc.d/rcX.d/[KS] * -根据运行级别X配置服务
    a) 终止以"K"开头的服务 b) 启动以"S"开头的服务 
h) /etc/rc.d/rc.local执行本地特殊配置 
i) 其他特殊服务
```





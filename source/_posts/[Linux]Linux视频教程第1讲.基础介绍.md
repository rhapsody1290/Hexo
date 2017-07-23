---
title: Linux视频教程第1讲.基础介绍

date: 2017-04-12 20:42:00

categories:
- Linux

tags:
- Linux

---

## linux的初步介绍

吉祥物：小企鹅（想起小时侯被企鹅咬了一口），芬生学生创建，微软反LINUX广告（四个变形动物）

## linux的特点
优点

* 免费的/开源 
* 支持多线程/多用户 
* 安全性好
* 对内存和文件管理优越

缺点

* 操作相对困难

## linux的历史 

1960时期左右，MIT，即麻省理工学院有一台电脑，使用分时操作系统，只能同时允许30个人通过终端登录

1965年，MIT、GE、Bell实验室，决定将30 -> 300个人分时系统，multis计划，即火星计划

1969年，火星计划失败。但Bell的Ken Thompson（C语言设计者）开发了一个file server system[文件系统]，在Bell实验室很受欢迎

在Dennis Ritchie的加入下，1973年，unix诞生，开源，源码内核共享

各个大公司看到了商机，认为Unix未来会有很大的前景，在源代码基础上进行开发

* IBM：AIX 
* Sun：Solaris 
* HP： HP unix 
* 伯克利分校：BSD

minix系统出现（也是跟上述操作系统一样），麻雀虽小，五脏俱全

Linus Torvalds，芬兰读书，拥有PC 386，1991年计划把minix移植到pc上（商用的只能放在特定的机器上），1994发布linux 1.0版 [linux is not unix]，完全没有桌面

Linux内核代码也是开源的，一些公司看到些商机

* redhat红帽子
* s.u.s.e
* 红旗linux（中国）

## linux的第一次接触

###关机命令

```
shutdown-h now  #立即进行关机(管理员root才可以) 
shutdown -r now #现在重新启动计算机 
reboot          #现在重新启动计算机
```
###进入图形桌面 

```
startx
```

###用户登录

```
输入用户名密码

登录时尽量少用root账户登录，因为它是系统管理员，最大的权限，难免操作失误。
可以利用普通用户登录，登录后再用"su -"命令来切换成系统管理员身份
```

###用户注销

```
在提示符下输入logout即可，快捷键ctrl+D
```
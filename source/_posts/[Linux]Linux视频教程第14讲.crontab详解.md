---
title: Linux视频教程第14讲.crontab详解

date: 2017-04-12 23:40:00

categories:
- Linux

tags:
- Linux

---

## 概述

任务调度：是指系统在某个时间执行的特定的命令或程序 
系统工作：有些重要的工作必须周而复始地执行，如病毒扫描等 
个别用户工作：个别用户可能希望执行某些程序

### 命令

设置系统级任务调度：/etc/crontab 文件上编写调度任务
设置个人任务调度，执行crontab ‐e命令，接着输入任务到调度文件

### 调度文件的规则

![这里写图片描述](http://img.blog.csdn.net/20151226141249322)

[案例] 希望每天凌晨2：00去执行 date >> /home/mydate2，可以在crontab ‐e中加入：

```
0 2 * * * date >> /home/mydate2 
```

[案例]希望每分钟去执行：在crontab ‐e中加入：

```
* * * * * date >> /home/mydate2
```

### 调度多个任务

1、在crontab ‐e中直接写多个命令（不推荐）

```
* * * * * date >> /home/mydate2 
* * * * * * cp /home/mydate2 /root
```

缺点：所有任务按列表的方式进行写入，不能方便的管理

2、可以把所有的任务，写入到一个可执行文件（shell编程）

```
1. vi mytask.sh，写入
* * * * * date >> /home/mydate2 
* * * * * cp /home/mydate2 /root)
2. chmod 744 mytask.sh （默认不具有执行权限，只有rw-）
3. crontab中写入
* * * * /root/mytask.sh
开始执行命令

说明：.sh表示shell文件，chmod 修改权限，必须要有X的权限
```

##终止任务调度
```
crontab ‐r：终止任务调度(删除所有) ，若只需删除一个，则进入文件删除保存
crontab ‐l：列出当前有哪些任务调度
```
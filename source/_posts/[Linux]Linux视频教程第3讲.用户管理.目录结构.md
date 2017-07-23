---
title: Linux视频教程第3讲.用户管理.目录结构

date: 2017-04-12 21:01:00

categories:
- Linux

tags:
- Linux

---

## 简单介绍
详细介绍
http://blog.sina.com.cn/s/blog_620eb3b20101hzt3.html

linux的文件系统是采用层级式的树状目录结构，在此结构中的最上层是根目录"/"，然后在此目录下再创建其他的目录

深刻理解linux文件目录是非常重要的（9个）

![这里写图片描述](http://img.blog.csdn.net/20151220163024495)

```
root，存放root用户的相关文件 
home，存放普通用户的相关文件
bin，存放常用命令的目录，如vi，su
sbin，要具有一定权限才可以使用命令 mnt，默认挂载光驱和软驱的目录
etc，存放配置的相关文件
usr，安装一个软件的默认目录，相当于windows下的program files
var，存放经常变化的文件，如网络连接的sock文件
boot，存放引导系统启动的相关文件
```

## 装置文件名

* IDE 硬盘：/dev/hd[a-d]
* CDROM：/dev/cdrom
* 打印机：/dev/lp[0-2]
* 软盘驱劢器：/dev/fd[0-1]
* 网络卡：/dev/eth[0-n]

## Linux的用户管理

1. useradd 用户名   #添加用户，root权限，home会出现一个文件夹
2. passwd  用户名   #为新用户设密码
3. userdel 用户名   #删除用户
a) 【案例】userdel xiaoming #删除用户但保存用户主目录 
b) 【案例】userdel‐r xiaoming #删除用户以及用户主目录 
4. logout           #当前用户退出 
5. who am i         #当前用户是谁

提示："#"表示root用户，"$"表示普通用户。






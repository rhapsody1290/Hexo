---
title: Linux软件安装目录选择

date: 2017-05-16 22:00:00

categories:
- Linux

tags:
- Linux

---

/usr：系统级的目录，可以理解为C:/Windows/，/usr/lib理解为C:/Windows/System32。
/usr/local：用户级的程序目录，可以理解为C:/Progrem Files/。用户自己编译的软件默认会安装到这个目录下。
/opt：用户级的程序目录，可以理解为D:/Software，opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。

**个人习惯让软件包管理器来管理/usr目录，而把自定义的脚本(scripts)、软件放到/usr/local目录下面。/opt存放测试软件，用完就把目录删除即可**
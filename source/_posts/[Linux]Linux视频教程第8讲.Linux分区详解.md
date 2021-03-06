---
title: Linux视频教程第8讲.Linux分区详解

date: 2017-04-12 21:48:00

categories:
- Linux

tags:
- Linux

---

## 概述

硬盘的分区主要分为**d分区**（Primary Portion）和**扩展分区**（Extension Portion）两种。只是针对一个硬盘来讲，**主分区和扩展分区的数目之和不能大于4个**。

基本分区可以马上被使用但不能再分区，扩展分区必须再进行分区后才能使用，也就是说它必须还要进行二次分区。那么有扩展分区再分下去的是什么呢？它就是**逻辑分区**（Logical Portion），而且逻辑分区没有数量上限制

![](http://i.imgur.com/gke7XfC.png)

对windows用户来说，有几个分区就有几个驱动器，并且每个分区都会获得一个字母标识符，然后就可以选用这个字母来指定在这个分区上的文件和目录。它们的文件结构都是独立的，非常好理解。但对这些用户初上手Redhat Linux，可就有点恼人了。

因为对Redhat Linux用户来说无论有几个分区，分给哪一个目录使用，它归根结底就**只有一个根目录、一个独立且唯一的文件结构**。

**Redhat Linux中每个分区都是用来组成整个文件系统的一部分**。因为它采用了一种叫"载入"的处理方法，它的整个文件系统中包含了一整套的文件和目录，并将一个分区和一个目录联系起来。这时要载入的那个分区将使它的存储空间在这个目录下获得

## 硬盘
    
对于IDE硬盘，驱动器标识符为hdx~，其中"hd"表明分区所在设备的类型，这里是指IDE硬盘了。
    
"x"为盘号（a为基本盘，b为基本从属盘，c为辅助主盘，d为辅助从属盘），"~"代表分区，**前四个分区用数字1到4表示，它们是主分区或扩展分区，从5开始就是逻辑分区。逻辑分割的装置文件名号码，<font color='red'>一定由 5 号开始</font>**

例如：**hda3表示为第一个IDE硬盘上的第三个主分区或扩展分区**，hdb2表示为第二个IDE硬盘上的第二个主分区或扩展分区

对于SCSI硬盘则标识为"sdx~"，SCSI硬盘是用"sd"来表示分区所在设备的类型的，其余则和IDE硬盘的表示方法一样

## 文件系统

挂载点的意义：
每个filesystem都有独立的inode/block/superblock等信息，这个文件系统要能够链接到目录树才能被我们使用。将文件系统与目录树结合的动作我们成为挂载。挂载点一定是一个目录，该目录是进入该文件系统的入口。因此并不是任何文件系统都可以使用，必须要挂载到目录树的某个目录后，才能使用该文件系统

链接
1. Hard Link，实体链接，文件名链接到某个inode号码，两个文件名的所有相关信息都会一模一样（除了文件名），如果任意一个档名删除，其实inode与block都是存在的，仍可以通过另一个档名来读取到正确的档案数据。无论使用哪个档名来编辑，最终的结果都会写入到相同的inode与block中，均能对数据进行修改（ps:类似C++中的引用）
2. Sysbolic Link，符号链接，即快捷方式。符号连接建立一个独立的档案，而这个档案会让数据的读取指向他link的那个档案的档名。当来源档被删除之后，符号链接的档案会开不了。
        
## 内存置换空间（swap）之建置

    p289
    swap的功能就是在物理内存不足的情况下，内存中暂不使用的数据就会被挪到swap中，此时内存就会空出来给所需要执行的程序加载

## 几个重要命令

### 挂载命令

```
mount [-parameters] [设备名称] [挂载点]
mount /dev/sda1 /test/  #将sda1挂载到/test
```

特别说明：在挂载光驱时，可直接使用 mount /mnt/cdrom 

###卸载命令 

```
umount [挂载点] 
umount /test/   #卸载分区
```

### 查看Linux系统分区具体情况

```
df -h 
df [目录全路径]，查看某个目录是在哪个分区
```

###分区

```
fdisk ‐l    #查看系统有哪些分区
```

fdisk只有root可以执行，使用的装置名不要加上数字，因为partition是针对整个硬盘装置，而不是分区。例：

```
fdisk /dev/hdc
```

###磁盘格式化

mkfs:make filesystem

```
mkfs -t ext3 /dev/hdc6    #将/dev/hdc6格式化ext3文件系统
```
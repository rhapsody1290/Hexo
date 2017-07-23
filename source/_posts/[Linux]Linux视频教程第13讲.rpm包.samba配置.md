---
title: Linux视频教程第13讲.rpm包.samba配置

date: 2017-04-12 23:27:00

categories:
- Linux

tags:
- Linux

---

## RPM包

### 概述

**一种用于互联网下载包的打包及安装工具**，它包含在某些Linux分发版中。它生成具有.RPM扩展名的文件。RPM是Redhat Package Manager（Redhat软件包管理工具）的缩写。这一文件格式虽然打上了Redhat的标志，但是其原始设计理念是开放式的，现在包括OpenLinux、S.u.S.E.以及Turbo Linux等Linux的分发版本都有采用。可以算是公认的行业标准了

### RPM包的名称格式 

apache-1.3.23-11.i386.rpm

"1.3.23-11"：软件的版本号，主版本和此版本
"i386"：是软件所运行的硬件平台 
"rpm"：文件扩展名，代表RPM包

### RPM常用命令 

```
rpm ‐qa             #查询所安装的所有rpm软件包 
rpm ‐qa | more      #分页显示
rpm ‐qa | grep X "apache"   #软件名称
rpm ‐q 软件包名     #查询软件包是否安装★★★
rpm ‐qi 软件包名    #查询软件包信息 
rpm ‐ql 软件包名    #查询软件包中的文件 
rpm ‐qf 文件全路径名        #查询文件所属的软件包
    [例]rpm ‐qf /etc/passwd 
    [例]rpm ‐qf /root/install.log
rpm ‐qp 包文件名    #查询包的信息对这个软件包的介绍
[例]rpm ‐qp jdk-1_5_0-Linux-i586.rpm 
[例]rpm ‐qpi jdk-1_5_0-Linux-i586.rpm 
[例]rpm ‐qpl jdk-1_5_0-Linux-i586.rpm
```

## 安装RPM包

```
rpm ‐ivh RPM包全路径名称    #安装包到当前系统
i=install，安装
v=verbose，提示，即有提示信息 
h=hash，进度条
```

### 删除RPM包

```
rpm ‐e RPM包的名称 
【案例】rpm ‐e jdk
```

如果其它软件包依赖于您要卸载的软件包，卸载时则会产生错误信息，如：

```
【案例】rpm ‐e foo
removing these packages would break dependencies：foo is needed by bar-1.0-1 若让RPM忽略这个错误继续卸载，请使用‐‐nodeps命令行选项 
```

```    
强行删除，不建议
【案例】rpm ‐e ‐‐nodeps foo
```

### 升级RPM包

```
rpm ‐U RPM包全路径名
【案例】rpm ‐U cvs-1.11.2-10.i386.rpm
```

## SPRM
鸟哥p815

## 安装软件路径

    以apache为例，安装路径有
    · /etc/httped
    · /usr/lib
    · /usr/bin
    · /usr/share/man
    分别代表配置文件、函式库、执行档、联机帮助档
    如果放在预设的/usr/local，那么数据就会被放在
    · /usr/local/etc
    · /usr/local/bin
    · /usr/local/lib
    · /usr/local/man
    如果每个软件都选择在这个默认的路径下安装，那么所有的软件的档案都将放置在这四个目录中，未来想要升级或移除的时候都会比较难以追查档案的来源。所以如果在安装的时候选择的是单独的目录，例如apache放在/usr/local/apache，那么档案目录就会变成：
    · /usr/local/apache/etc
    · /usr/local/apache/bin
    · /usr/local/apache/lib
    · /usr/local/apache/man
    移除软件就简单的多，只要将目录移除即可
    
    为方便Tarball的管理，建议：
    1. 最好将tarball的原始数据解压到/usr/local/src中
    2. 安装时最好安装到/usr/local这个默认路径下
    3. 考虑未来的反安装步骤，最好可以将每个软件单独的安装在/usr/local底下
    4. 为安装到单独的软件之man page加入man path搜寻：如果你安装的软件放置在/usr/local/software，那么man page搜索的设定中，可将就得在/etc/man.config的40~50加入一行MANPATH /usr/local/software/man

##函式库管理

```
p794

静态函式库，扩展名.a
动态函式库，扩展名.so
绝大数的函式库都放置在：/usr/lib,/lib,Kernel的函式库在/lib/modules

将动态函式库加载高速缓存当中
1. 在/etc/ld.so.conf写下想要读入高速缓存当中的动态函式库所在的目录
2. 接在来利用ldconfig将/etc/ld.so.conf读入快取当中，画面上不会显示任何的信息
3. 同时也将数据记录一份在/etc/ld.so.cache这个档案当中
```

##samba配置

###什么是samba

这些年来，windows与Linux操作系统各自拥有自己的用户群和市场。然而在一般公司或学校里，可能同时有windows和Linux主机，windows主机彼此之间可以利用"网上邻居"来访问共享资源。NFS也能使Linux主机之间实现资源访问。**而samba服务软件能够使windows与Linux之间实现资源共享**.

SMB通信协议采用的是C/S结构，所以SAMBA软件可分阶段客户端及服务端两部分。通过执行samba客户端程序，Linux主机使可使用网络上的windows主机所共享的资源。而在Linux主机上安装samba服务器，则可以使windows主机访问samba服务器共享的资源

###samba安装 

samba的安装步骤
1、看看是否已经安装了samba
```
rpm ‐q samba 
```
2、如果有的话，就先卸载
```
rpm ‐e ‐‐nodeps samba（强制删除） 
```
3、把安装文件挂载到Linux下
```
a) samba-common-2.2.7a-7.9.0.i386.rpm 
b) samba-client-2.2.7a-7.9.0.i386.rpm 
c) samba-2.2.7a-7.9.0.i386.rpm
```
4、拷贝samba的rpm包到/home，准备安装
```
cp sam* /home
```
5、开始安装(顺序)
```
a) rpm ‐ivh samba-common-2.2.7a-7.9.0.i386.rpm 
b) rpm –ivh samba-client-2.2.7a-7.9.0.i386.rpm 
c) rpm –ivh samba-2.2.7a-7.9.0.i386.rpm
```
6、创建一个用户youyou
```
a) useradd youyou 
b) passwd youyou 
```
7、给youyou设置samba密码
```
将/etc/passwd中的用户都加到smbpasswd中
a) cat /etc/passwd | mksmbpasswd.sh > /etc/samba/smbpasswd

设置密码 
b) smbpasswd youyou 
```
8、启动samba服务器，测试
```
a) service smb start，启动
b) service smb stop，停止 
c) service smb restart,重启
d) 在windows运行窗口输入Linux的IP\\192.168.222.88 输入youyou的samba用户名，密码 
```
###samba配置

共享资源的基本配置/etc/samba/smb.conf

1、comment：针对共享资源所做的说明文字。默认值为空字符串
```
【案例】comment=dir for todayhero：共享这个目录是为了todayhero这个用户 
```
2、path：若共享的资源是目录，是指定该目录的位置
```
【案例】path=/tmp：共享tmp这个目录 
```
3、guestok：是否允许用户不使用账号和密码访问此资源
```
【案例】guest ok=yes：允许用户不使用账号和密码访问此资源
【案例】guest ok=no：不允许用户不使用账号和密码访问此资源 
```
4、hostsallow：设置连接主机的地址
```
【案例】hosts allow=192.168.2.1 server.abc.com：允许来自192.168.2.1或
server.abc.com
```
5、hostsdeny：设置禁止连接的主机地址
```
【案例】hosts deny=192.168.2.1：不允许192.168.2.1的主机访问samba服务器的资源
```
6、readonly：用于设置共享的资源是否为可读
```
【案例】read only=yes：允许只读
【案例】read only=no：不仅仅只读，也就是说可以写入
```

---
title: Linux常用命令

date: 2017-04-12 20:45:00

categories:
- Linux

tags:
- Linux

---

Linux命令缩写，方便记忆

http://blog.chinaunix.net/uid-27164517-id-3299073.html

## 开发者常用指令

<pre>

netstat -anp | grep 8080 #查询端口
route #路由表
tracert #路由追踪
ps -aux #进程列表
kill -9 pid #杀进程
top

env                     #查看环境变量
/etc/profile 和 ~/.bash_profile #系统整体配置文件（系统/个人用户）
ehoc $PATH              #PATH路径

ifconfig                #在linux/unix查看ip的命令

df - h                  #查看Linux系统分区具体情况
df -h /etc              #查看/etc底下可用的磁盘容量以易读的容量格式显示

crontab -e              #任务调度

ls -a                   #显示隐藏文件
ls -l                   #列表显示

du -h --max-depth=1     #查看文件夹大小

cat Hello.txt | grep "pattern" #搜索字符串
grep -n "pattern" file1 file2  #文件中查找，显示行数，可以在多个文件

tar -zxvf libpcap-1.3.0.tar.gz  #解压
zip aa.zip 文件名               #压缩

./configure，make，make install #Linux下编译安装源代码三个步骤
rpm ‐ivh RPM包全路径名称        #安装包到当前系统
rpm -ivh url --replacepkgs      #重新安装软件
rpm ‐e RPM包的名称              #删除

sudo 命令 / sudo su     #管理员权限

#文件操作

mkdir   folder          #建立文件夹
touch   filename        #新建文件
rmdir   folder          #删除空目录
rm filename             #删除文件
rm -rf 目录名字          #删除文件夹 -r下递归-f强行删除
cp file1 file2          #拷贝
cp -r dir1 dir2         #递归复制命令（复制子目录信息）
find /root/ -name Hello.java    #查找文件[不常用，速度慢，执行在硬盘中搜索，时间在文件权限那里]
</pre>

## 常用

```
cd ~                    #跳转到用户主目录
tab                     #文件名自动提示
pwd                     #查看当前目录
reset                   #清屏
clear                   #清屏
ctrl + alt + T          #终端，需在首选项-快捷键中设置

env                     #查看环境变量
/etc/profile            #系统整体配置文件
~/.bash_profile或~/.bash_login或~/.profile  #个人设定
ehoc $PATH              #PATH路径

ls                      #文件列表
ls -a                   #显示隐藏文件
ls -l                   #列表显示
ls -al <path>           #查看path路径下的文件列表
ls -ld /tmp             #查看目录的权限

|                       #管道命令
>                       #管道定向，双箭头>>追加到文件结尾
less                    #u，d，space，b
more                    #空格下一页，shift+PageUp上一页[more name]
cat xxx | more

grep xxx #搜索字符串
grep -n "System" file1 file2 #查找，显示行数

tar -zxvf libpcap-1.3.0.tar.gz  #解压
zip aa.zip 文件名

mount /mnt/cdrom        #挂载光驱 
unmount /mnt/cdrom      #卸载光驱

ifconfig                #在linux/unix查看ip的命令

df - h                  #查看Linux系统分区具体情况
df -h /etc              #查看/etc底下可用的磁盘容量以易读的容量格式显示

crontab -e              #任务调度
```

## 软件安装

```
# 安装二进制文件
vim README/INSTALL      #安装前参阅安装说明
./configure --help      #查看可用参数
./configure --prefix=/usr/local/xxx

#检查环境，创建makefile文件
make clean              #根据makefile清理一下环境
make                    #根据makefile进行编译
make insatll            #根据makefile进行安装

# rpm
rpm ‐q 软件包名             #查询软件包是否安装
rpm -qi rpm包               #列出该软件的详细信息
rpm -ql  rpm包  #列出该软件所有的文件与目录所在的完整文件名
rpm -qc rpm包               #查询配置文件
rpm -qf 配置文件             #查询配置文件是哪个软件的
rpm -qa                     #已安装的软件名称

rpm ‐ivh RPM包全路径名称     #安装包到当前系统
rpm -ivh url --replacepkgs  #重新安装软件
rpm ‐e RPM包的名称          #删除
rpm ‐e ‐‐nodeps samba      #强制删除 
rpm ‐Uvh RPM包全路径名     #升级，未安装的文件也会直接安装
rpm - Fvh                 #升级，未安装的不安装

rpm - V 软件              #该软件的档案被更动过，会列出来
rpm -Va                  #列出系统上所有可能被更动的档案
rpm -Vp 文件名            #列出该软件内可能被更动过的文件
rpm -Vf                  #某个文件是否被更动过

# yum
yun install 软件        #安装软件
yum update 软件         #升级软件
yum remove 软件         #移除软件
yum list    软件        #可用列表
```

##文件操作
```
mkdir   folder          #建立文件夹
touch   filename        #新建文件

rmdir   folder          #删除空目录
rm filename             #删除文件
rm -rf 目录名字          #删除文件夹 -r下递归-f强行删除
rm -i bashrc*           #利用通配符删除bashrc开头的文件   

mv test.log test1.txt   #文件改名
mv -i log1.txt log2.txt #将文件file1改名为file2，如果file2已经存在，则询问是否覆盖
mv test1.txt test3      #移动文件
mv log1.txt log2.txt log3.txt test3 #将文件log1.txt,log2.txt,log3.txt移动到目录test3中。 

cp file1 file2          #拷贝
cp -i file1 file2       #文件是否覆盖提醒
cp -a file1 file2       #相当于-dpR,保持文件的连接(d),保持原文件的属性(p)并作递归处理(R)
cp -p file1 file2       #连同文件的属性一起复制过去，而非使用默认属性
cp -r dir1 dir2         #递归复制命令（复制子目录信息）
cp -rf dir1 dir2        #拷贝文件夹，不提示是否覆盖

whereis <文件或目录>    #查找文件
locate <文件或目录>     #查找包含该字符的文件或目录，updatedb更新数据库
find /root/ -name Hello.java    #查找文件[不常用，速度慢，执行在硬盘中搜索，时间在文件权限那里]

ln –s /etc/inittab inittab #（inittab指向实际文件/etc/inittab） #ln 建立符号连接(和windows的快捷方式)
```
##系统管理
```
shutdown -h now         #立即进行关机(管理员root才可以) 
shutdown -r now         #现在重新启动计算机 
rebot                   #重启
halt                    #关机
logout                  #注销

chkconfig               #系统服务
netstat -anp | grep 8080 #查询端口
route #路由表
tracert #路由追踪
ps -aux #进程列表
kill -9 pid #杀进程

sudo 命令 / sudo su     #管理员权限
exit                    #退出当前用户
su - test               #切换普通用户
su或su -                #切换回root

//python
sudo pip install -r requirements.txt

#系统服务
service mysqld start    #启动mysql数据库
service mysqld stop     #停止
service mysqld status   #查看状态

chkconfig --list        #查看系统服务
chkconfig mysqld on      #添加到开机启动
chkconfig mydql off     #关闭开机启动

#防火墙
service   iptables status   #查询防火墙状态
service   iptables stop     #停止防火墙
service   iptables start    #启动防火墙
service   iptables restart  #重启防火墙
chkconfig   iptables off    #永久关闭防火墙
chkconfig   iptables on     #永久关闭后启用
更改过滤规则 1、vim /etc/sysconfig/iptables 2、添加过滤规则 3、service iptables restart 重启
```
##vim操作
p336
一般模式：
```
- 移动光标
hjkl                    #左下上右
ctrl + f                #屏幕向下一页，相当与Page Down★
ctrl + b                #屏幕向上一页，相当于Page Up★
ctrl + d                #屏幕向下半页
ctrl + u                #屏幕向上移动半页
0或[home]               #移动到这一行的最前面字符处★
$或[end]                #移动到这一行的最后面字符★
nG                      #n为数字，移动到文档第n行★

- 搜索
/pattern               #向下寻找和pattern匹配的内容★
?pattern               #向上寻找和pattern匹配的内容
:noh                    #取消高亮
字母n或N                #向前或向后继续寻找

- 替换
:n1,n2s/word1/word2/g   #n1与n2行之间用word2替换word1★
:1,$s/word1/word2/g     #从第一行到最后一行寻找word1，替换为word2★
:1,$s/word1/word2/gc    #第一行到最后一行寻找word1，替换为word2，替换前提示用户确认★

- 删除 
x,X                     #x向后删一个字符，X向前删★
dd                      #删除一行★

- 复制粘帖      
yy                      #复制光标那一行★
p，P                    #p下一行粘帖，P上一行
u                       #复原前一个动作
ctrl+r                  #重做上一个动作
.                       #重复前一个动作
```

编辑模式：
```
i,I     #i光标处插入，I所在行第一个非空字符插入
a,A     #a光标所下一个字符插入，行最后一个字符插入
o,O     #光标所在下一行插入新的一行，O上一行
```
指令列模式：
```
:w      #将编辑的数据写入硬盘文件
:w!     #强行写入
:q      #离开vi
:q!     #强制离开，不保存
:wq     #储存离开
:w filename #另存为
！<command>            #执行shell命令
```
分页器命令：
```
字母u和d                #分页器向上、向下翻动半页
空格、b                 #分页器向下翻一页，向上翻一页
字母j和k                #向下翻一行，上翻一行
左箭头、右箭头           #行截断，向左向右滚动
```
其他：
```
:1,$d                   #全部删除
v,V,ctrl+v              #字符选择,行选择，区块选择
y                       #反白的地方复制
d                       #反白的地方删除
```
##linux的用户、权限管理
```
# 用户
useradd 用户名   #添加用户，root权限，home会出现一个文件夹
passwd  用户名   #为新用户设密码
userdel 用户名   #删除用户
a) 【案例】userdel xiaoming #删除用户但保存用户主目录 
b) 【案例】userdel ‐r xiaoming #删除用户以及用户主目录

cat /etc/passwd     #查看Linux下所有用户信息
用户：密码：用户ID：组ID：用户主目录：shell解释器

logout           #当前用户退出 
who am i         #当前用户是谁

# 用户组
groupadd policeman   #添加组，只能在root用户下操作
cat /etc/group  #查看所有组
useradd –g 组名 用户名  #创建用户，并添加到指定组
usermod -g 组名 用户名 #把一个用户移值到另一个组

# 操作
chmod 777 文件名     #用户所在组、其他组都能进行读、写、执行
chown 用户名 文件名   #修改文件所有者
chown ‐R root ./abc   #改变abc这个目录及其下面所有的文件和目录的所有者是root
chgrp 组名 文件名   #修改文件所有组

umask  #文件预设权限，为文件或文件夹建立时默认的权限，umask -S以符号类型的方式显示
```

##压缩
```
#gzip常用用法
gzip    文件名             #压缩文件，并删除源文件，后缀gz
gzip -d     xxx.gz                  #解压

#bzip2常用用法
bzip2压缩比比gzip还要好，后缀bz2
bzip2   文件名                      #压缩
bzip2 -d 压缩名                     #解压
bzcat   压缩名                      #查看压缩包

#zip常用用法
zip aa.zip 文件名1 文件名2          #压缩文件
zip –r aa.zip 文件夹路径            #压缩文件夹
zip file.zip * -x file             #将不需要压缩的文件排除在外  

unzip file.zip                     #直接解压缩文件
unzip file.zip –x file2            #除了file2文件外，其他的文件都解压缩
unzip –Z file.zip                  #查看file.zip压缩包的内容。也可以使用“-l”、“-v”来查看压缩包的内容

zip -d test1.zip test.MYI #删除压缩文件test1.zip中test.MYI文件
zip -m test1.zip test. MYI #向压缩文件中test1.zip中添加test. MYI文件

#打包指令tar
p307
-j 透过bzip2压缩，档名取为*.tar.bz2
-z 透过gzip压缩，档名趣味*.tar.gz

tar -jcvf filename.tar.bz2 要被压缩的档案或目录 #压缩
tar -jtvf filename.tar.bz2  #查询
tar -jxvf filename.tar.bz2  #解压缩

tar -jxvf filename.tar.bz2 -C /tmp #-C解压目录
tar -jzxf filename.tar.bz2 待解开档名   #解开单一档案
tar -jcvf filename.tar.bz2 --exluce=etc* #不包含默写档案

tar -zpcvf /root/etc.tar.gz /etc    
tar -jpcvf /root/etc.tar.bz2 /etc
#备份etc,p保留原本档案的权限与属性
```

## 环境变量

	.bash_profile 位于主目录下，它为系统的每个用户设置环境信息
	**/etc/ 中 profile 主要是配置环境变量**
	
	配置 .bashrc 文件可以指定某些程序在用户登录的时候就自动启动。

##命令集

```
date        #日期
cal         #日历
bc          #计算器
ctrl + D    #文字输入结束
```

##命令专题
命令init [0123456]
```
指定系统运行级别，类似windows的正常运行模式或安全模式 

0：关机 
1：单用户
2：多用户状态没有网络服务 
3：多用户状态有网络服务[常用]
4：系统未使用保留给用户 
5：图形界面 
6：系统重启

常用运行级别是3和5，要修改默认的运行级别可改文件 /etc/inittab的id:5:initdefault:这一行中的数字

切换用户：输入su切换用户或者logout

FAQ：不小心设置了6，导致系统启动-重启-启动循环，怎么办？ 

1. 在进入grub引导界面时，在数秒的时候，请输入 e 
2. 然后选中第二行(kernel)，输入e
3. 在出现的界面里，输入1【1表示单用户级别】(6.5中测试是输入single，最终显示的是quiet single)，1的前面需要加一个空格，单用户模式既可以修改模式，又可以修改密码，Enter 
4. 返回后，按b，重新启动

注意：用运行级别1可以绕过ROOT密码，不需要密码就可以用，用passwd就OK
```
---
配置文件被更改

如：![此处输入图片的描述](http://www.th7.cn/d/file/p/2014/11/04/74a34c1bdf612bbcc05bd30c3a5647ed.png)
```
最前面的八个信息是：
    S ：(file Size differs) 档案的容量大小是否被改变
    M ：(Mode differs) 档案的类型戒档案的属性 (rwx) 是否被改变？如是否可执行等参数已被改变
    5 ：(MD5 sum differs) MD5 这一种纹码的内容已经不同
    D ：(Device major/minor number mis-match) 装置的主/次代码已经改变
    L ：(readLink(2) path mis-match) Link 路径已被改变
    U ：(User ownership differs) 档案的所属人已被改变
    G ：(Group ownership differs) 档案的所属群组已被改变
    T ：(mTime differs) 档案的建立时间已被改变

第二排的意思是：
    c ：配置文件 (config file)
    d ：文件数据文件 (documentation)
    g ：鬼档案～通常是该档案丌被某个软件所包吨，较少发生！(ghost file)
    l ：许可证文件 (license file)
    r ：自述文件 (read me)
```
---

##shell使用

shell脚本文件：
<pre>
a) 是一个文本文件
b) 命令的集合 
c) 有执行的权限 
d) 执行方式（./文件名）
</pre>

**profile文件**
profile文件主要是用于配置环境变量
![这里写图片描述](http://img.blog.csdn.net/20151227194314714)

**用户登录后自动执行的shell脚本文件**
配置.bashrc文件可以指定某些程序在用户登录的时候就自动启动。
```
如：/home/tomcat/bin/startup.sh start
```


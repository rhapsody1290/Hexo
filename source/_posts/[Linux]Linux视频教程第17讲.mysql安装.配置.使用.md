---
title: Linux视频教程第17讲.mysql安装.配置.使用

date: 2017-04-13 10:32:00

categories:
- Linux

tags:
- Linux

---

## 概述
http://www.greensoftcode.net/techntxt/20112911325655007025
安装教程
http://m.sogou.com/web/uID=NsZTT-9VbVa3FHjJ/v=5/type=1/sp=3/ct=160216001911/keyword=linux+mysql%E5%85%8D%E5%AE%89%E8%A3%85/id=14027a77-47bf-467c-bdf4-f1f60458878f/sec=Yw8xhdD_JJCJYpZI3fALVg../tc?pg=webz&clk=24&url=http%3A%2F%2Fwww.faceye.net%2Fsearch%2F117071.html&f=0&id=14027a77-47bf-467c-bdf4-f1f60458878f&pid=sogou-apps-4ff6fa96179cdc28&dp=1&lxg=NGRhZm9wYWtuMDxqMT86bWs7bWk4bWkla2dlJmlmbHpnYWwmanpnf3tteiU%2FJT8mMSVrfTUlbmk1&key=linux+mysql%E5%85%8D%E5%AE%89%E8%A3%85&pno=3&g_ut=3&wml=0&w=1347&mcv=361&pcl=288,4187&sed=0&ml=28&sct=38
mysql数据库在linux下可以充分发挥威力，mysql数据库越来越受到软件公司的青睐，为什么呢？

免费、跨平台、轻、支持多并发

在北京很多软件公司属于创业型的中、小公司，从节约成本的角度考虑，mysql特别适合中、小项目

## mysql安装


ps:安装之前查看是否已经安装mysql，

```
rpm –qa mysql
```
如果有就删除之

```
rpm –e mysql
rpm –e -–nodeps mysql 强制删除

yum remove  mysql mysql-server mysql-libs mysql-server;
find / -name mysql 将找到的相关东西delete掉；
rpm -qa|grep mysql(查询出来的东东yum remove掉)
```

1、准备安装文件，copy到/usr/local下 
2、把安装文件解压

```
tar -zxvf mysql-5.5.47-linux2.6-x86_64.tar.gz
```

3、重命名

```
mv mysql-5.5.47-linux2.6-x86_64 mysql 
```

4、创建mysql组

```
useradd mysql
```

5、创建mysql用户，并放入到mysql组中

```
useradd -g mysql mysql
```

6、关键步骤

```
a) 进入mysql文件夹，执行如下命令来初始化数据库
scripts/mysql_install_db --user=mysql
b) 修改文件的所有者(数据库是重要文件，不希望拥有者是普通用户)
chown ‐R root .
c)修改date文件夹的所有者(数据文件夹需要普通用户能够操作)
chown ‐R mysql data
d) 改变用户组
chgrp ‐R mysql .
说明：“.”点号代表当前目录及文件
```

7、启动mysql 

```
bin/mysqld_safe –-user=mysql & 
&表示以后台的方式启动
```

ctrl+c退出，检查一下进程，netstat ‐anp，查看监听端口是3306的是不是打开了

**出现问题pid file /var/run/mysqld/mysqld.pid ended**
http://www.csdn123.com/html/exception/646/646216_646212_646218.htm

8、如何进入mysql

```
cd bin
./mysql ‐u root ‐p回车
```

9、mysql修改密码

http://www.cnblogs.com/daizhuacai/archive/2013/01/17/2865138.html

10、添加环境变量

如果希望在任何一个目录下都可以进入mysql，则需在用户变量/root/.bash_profile中添加路径

```
a) env | more
查看path中是否指定mysql 路径
b) 进入/root下，查看到.bash_profile 存放用户变量 
c) vi进入.bash_profile，在path后面加上 /home/mysql/bin/
d) logout一下，再登录输入
mysql ‐uroot ‐proot
此时root用户可以在任意目录输入命令就可以进入mysql了

如果需要所有用户都能在任意位置可以输入命令，则需要修改/etc/profile的PATH
```

11、设置mysql开机自动启动

参考http://blog.sina.com.cn/s/blog_6ccd0a11010175ri.html
http://blog.163.com/longsu2010@yeah/blog/static/173612348201111710850381/

```
cp support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld    #将mysqld加到启动服务列表里
运行命令“chkconfig --list”查看系统服务
chkconfig mysqld on     #mysql.server开机启动
chkconfig mysqld off    #该命令关闭了mysql开机启动
> 
```

12、多种启动方式

```
一、启动
1、使用 service 启动：service mysqld start
2、使用 mysqld 脚本启动：/etc/inint.d/mysqld start
3、使用 safe_mysqld 启动：safe_mysqld&
二、停止
1、使用 service 启动：service mysqld stop
2、使用 mysqld 脚本启动：/etc/inint.d/mysqld stop
3、mysqladmin shutdown
三、重启
1、使用 service 启动：service mysqld restart
2、使用 mysqld 脚本启动：/etc/inint.d/mysqld restart
```

## Mysql测试

测试mysql数据库是否可以在linux下正确使用

a) 建立数据库和表 

b) 加入部分数据

c) 编写一个ShowUser.java文件，在控制台显示用户

```
import java.sql.*;
public class MysqlTest{
    public static void main(String[] args){
        try{
            Class.forName("com.mysql.jdbc.Driver");
            Connection con = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/Demo?user=root&password=root");
            Statement sm = con.createStatement();
            ResultSet rs = sm.executeQuery("select * from users");
            while(rs.next()){
                System.out.println(rs.getString(2));
            }
        }catch(Exception e){
            e.printStackTrace();
        }

    }
}
```

Note：特别注意mysql的驱动要存放的位置，放在jdk的主目录下的/jre/lib/ext/ 

## 备份与恢复

备份：

```
./mysqldump ‐u root ‐p密码 数据库名 > /home/data.bak 
```

恢复：

```
mysql ‐u root ‐p密码 数据库名 < /home/data.bak
```
 
Note：‐p和密码之间没有空格,在恢复数据库的时候，你需要建立一个空数据库！
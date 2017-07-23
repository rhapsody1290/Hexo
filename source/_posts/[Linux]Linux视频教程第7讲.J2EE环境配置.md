---
title: Linux视频教程第7讲.J2EE环境配置

date: 2017-04-12 21:45:00

categories:
- Linux

tags:
- Linux

---

**jdk安装步骤**

1、把mypackage.iso挂载到linux操作系统上
2、在vm做好配置
3、mount /mnt/cdrom，挂载光驱 unmount /mnt/cdrom，卸载光驱
4、把安装文件拷贝到/home -

```
cp 文件 /home
```

5、安装

```
./ j2sdk-1_4_2_19-linux-i586.bin
```

6、查看一个文件vi /etc/profile [环境配置文件] #注释配置先前安装的jdk

7、jdk1.5.0_06配置完毕需要注销一下 

**eclipse安装步骤**

1、挂载共享文件
2、把安装文件拷贝到/home
3、安装

```
tar ‐zxvf eclipse-SDK-3.2.1-linux-gtk.tar.gz 
cp 文件 /home
```

4、进入图形界面，运行eclipse需要桌面支持

```
startx
```

5、启动eclipse 

```
./eclipse
```

**MyEclipse安装步骤**

1、挂载共享文件
2、把安装文件拷贝到/home
3、安装

```
./ MyEclipseEnterpriseWorkbenchInstaller_5_1_0GA_E3_2_1.bin 
cp 文件 /home
```

注意点:进入图形界面安装支持，否则报错 选择已安装的eclipse的主目录

重新启动eclipse 

```
./eclipse &     #后台运行eclipse
```

这时会发现，菜单栏上多了一个MyEclipse选项

**tomcat安装步骤**

我们知道java ee的服务器有tomcat、jboss、weblogic、websphere、resin…这些都可以安装到linux下，我们给人家安装tomcat，安装步骤如下：

1、挂载共享文件
2、把安装文件拷贝到/home
3、安装
4、测试

编写一个简单的jsp页面 配置tomcat和jdk

```
tar ‐zxvf jakarta-tomcat-5.0.30.tar.gz 
cp 文件 /home
```


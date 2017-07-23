---
title: Linux下安装Nexus

date: 2017-05-16 10:06:00

categories:
- Maven

tags:
- Maven
- Nexus

---

## Linux安装目录选择

/usr：系统级的目录，可以理解为C:/Windows/，/usr/lib理解为C:/Windows/System32。
/usr/local：用户级的程序目录，可以理解为C:/Progrem Files/。用户自己编译的软件默认会安装到这个目录下。
/opt：用户级的程序目录，可以理解为D:/Software，opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。

**个人习惯让软件包管理器来管理/usr目录，而把自定义的脚本(scripts)、软件放到/usr/local目录下面。/opt存放测试软件，用完就把目录删除即可**

## Nexus安装

**注意Nexus对JDK版本有要求**
博主环境：JDK1.6.45 ，nexus-2.4-bundle.tar.gz
Nexus所有版本下载地址：http://www.sonatype.org/nexus/archived

1、下载Nexus后，在/usr/local/下创建nexus目录
2、进入nexus目录，将下载文件nexus-2.4-bundle.tar.gz移动到该目录下，使用mv命令
3、tar -zxvf nexus-2.4-bundle.tar.gz节解压，会生成nexus-2.4.0-09、sonatype-work两个目录
4、编辑系统配置文件：vim /etc/profile，在文件的尾巴增加下面内容：

	# Nexus
	NEXUS_HOME=/usr/local/nexus-2.4.0-09
	export NEXUS_HOME
	RUN_AS_USER=root
	export RUN_AS_USER

5、刷新配置：source /etc/profile
6、开放防火墙端口：

* 添加规则：sudo iptables -I INPUT -p tcp -m tcp --dport 8081 -j ACCEPT
* 保存规则：sudo /etc/rc.d/init.d/iptables save
* 重启 iptables：sudo service iptables restart

7、启动nexus，进入/usr/local/nexus/nexus-2.4.0-09/bin目录，执行./nexus start
8、访问http://115.159.84.37:8081/nexus

参考：

https://tianweili.github.io/2015/03/17/Linux%E4%B8%8B%E4%BD%BF%E7%94%A8nexus%E6%90%AD%E5%BB%BAmaven%E7%A7%81%E6%9C%8D/
https://github.com/judasn/Linux-Tutorial/blob/master/Nexus-Install-And-Settings.md

## 配置nexus、添加自定义jar包

仓库地址：

	<!-- 仓库地址 -->
	<repositories>
		<repository>
			<id>nexus</id>
			<name>Team Nexus Repository</name>
			<url>http://yourhostname:8081/nexus/content/groups/public</url>
		</repository>
	</repositories>

jar包依赖：

	<!-- jar -->
	<dependencies>
		<dependency>
			<groupId>de.innosystec</groupId>
			<artifactId>java-unrar</artifactId>
			<version>0.5</version>
		</dependency>
	</dependencies>

https://tianweili.github.io/2015/03/17/Linux%E4%B8%8B%E4%BD%BF%E7%94%A8nexus%E6%90%AD%E5%BB%BAmaven%E7%A7%81%E6%9C%8D/

## 我搭建的maven仓库

http://maven.qianmingxs.com:8081/nexus

仓库地址：http://maven.qianmingxs.com:8081/nexus/content/groups/public

## Nexus配置、部署

http://aijezdm915.iteye.com/blog/1335025
---
title: 邮件服务器postfix安装

date: 2017-12-04 12:28:00

categories:
- Linux

tags:
- postfix
- 邮件

---

## 腾讯云服务器不能用的原因

腾讯为提高腾讯IP质量，将25端口禁止了，详见

https://cloud.tencent.com/developer/ask/23866

## 调试邮件服务器

配置文件

	vim /etc/postfix/main.cf

重启
	
	service postfix restart

服务器端口

	telnet 127.0.0.1 25

日志

	tail -f /var/log/maillog

测试

	echo "Mail Content" | mail -s "Mail Subject" qianmingxs@126.com

## postfix各个目录的含义

/etc/postfix:该目录中包括Postfix服务的主配置文件、各类脚本、查询表等。
/usr/libexec/postfix/:该目录中包括Postfix服务的各个服务器程序文件。
/var/spool/postfix/:该目录中包括Postfix服务的邮件队列相关的子目录，其中每个队列子目录用于保存不同的邮件，比如说：

	1>.Incoming(传入)：刚接收到的邮件。
	2>.Active(活动)：正在投递的邮件。
	3>.Deferred(推迟)：以前投递失败的邮件。
	4>.Hold(约束)：被阻止发送的邮件。
	5>.Corrupt（错误）：不可读或不可分析的邮件。

/usr/sbin/:该目录中包括Postfix服务的管理工具程序，这些程序文件名以post开头。其中，主要的几个程序文件及其作用如下。

	1>.Postalias:用于构造、修改和查询别名表。
	2>.Postalias:用于显示和编辑main.cf配置文件。
	3>.Postfix:用于启动、停止postfix,要求有root用户权限。
	4>.Postmap:用于构造、修改或者查询查询表。
	5>.Postqueue:用于管理邮件队列，一般用户使用。
	6>.Postsuper:用于管理邮件队列，要求有root用户权限。

参考：http://www.linuxidc.com/Linux/2013-08/88977.htm
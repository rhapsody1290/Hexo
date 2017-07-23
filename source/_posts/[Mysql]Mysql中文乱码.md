---
title: Mysql中文乱码

date: 2017-02-25 23:42:00

categories:
- Mysql

---

## JDBC?

JDBC获得数据库连接时写在URL上的?useUnicode=true&characterEncoding=utf-8，作用是指定字符的编码和解码格式

例如：mysql数据库用的是gbk编码，而项目数据库用的是utf-8编码。这时候如果添加了useUnicode=true&characterEncoding=UTF-8 ，那么作用有如下两个方面：

1、存数据时：

数据库在存放项目数据的时候会先用UTF-8格式将数据解码成字节码，然后再将解码后的字节码重新使用GBK编码存放到数据库中。

2、取数据时：

在从数据库中取数据的时候，数据库会先将数据库中的数据按GBK格式解码成字节码，然后再将解码后的字节码重新按UTF-8格式编码数据，最后再将数据返回给客户端。

## 设置数据库字符集

有时候程序操作Mysql数据库时会出现 "?????" 或者是其他中文乱码，在命令行中输入：

	show variables like 'character%';

可看到如下字符：

	character_set_client		latin1
	character_set_connection	latin1
	character_set_database	  utf8
	character_set_results	   latin1
	character_set_server	    utf8
	character_set_system		utf8

解决办法是，在连接数据库之后，读取数据之前，先执行一项查询"SET NAMES UTF8"

* 在PHP里为 mysql_query("SET NAMES UTF8"); 
* 在JDBC或者Mybatis中使用查询SQL语句为 "SET NAMES UTF8"

到MySQL命令行输入"SET NAMES UTF8;"，然后执行

	show variables like 'character%';

发现原来为latin1的那些量"character_set_client"、“character_set_connection”、
"character_set_results"的值全部变为utf8了，原来是这3个变量在捣蛋。

查阅手册，上面那句等于：

	SET character_set_client = utf8;     
	SET character_set_connection = utf8; 
	SET character_set_results = utf8;    

看看这3个变量的作用：

* 信息输入路径：client→connection→server
* 信息输出路径：server→connection→results

换句话说，每个路径要**经过3次改变字符集编码**。以出现乱码的输出为例，server里utf8的数据，传入connection转为latin1，传入results转为latin1，utf-8页面又把results转过来。**如果两种字符集不兼容，比如latin1和utf8，转化过程就为不可逆的，破坏性的。**

但这里要声明一点，"SET NAMES UTF8"作用只是临时的，MySQL重启后就恢复默认了。

总结：为了让你的网页能在更多的服务器上正常地显示，还是加上"SET NAMES UTF8"吧，即使你现在没有加上这句也能正常访问。

从执行命令前后可知，set names gbk只可以修改character_set_client、character_set_connection、 character_set_results的编码方式，并且这种修改是窗口级别的，只针对本窗口有效，打开另外一个窗口修改无效。也可发现数据库底层的编码方式没有改变，插入数据后还是以utf8编码方式保持。

http://blog.csdn.net/zsmj_2011/article/details/7943734

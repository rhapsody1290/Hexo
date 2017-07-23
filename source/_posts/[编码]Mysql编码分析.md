---
title: Mysql编码分析

date: 2017-02-27 19:18:00

categories:
- 编码

tags:
- 编码

---

http://www.jb51.net/article/30864.htm

查看Mysql的编码设置，在命令行中输入：

	show variables like "char%";

MySQL字符集设置 

* character_set_server：默认的内部操作字符集 
* character_set_database：当前选中数据库的默认字符集 
* character_set_client：客户端来源数据使用的字符集 
* character_set_connection：连接层字符集 
* character_set_results：查询结果字符集 
* character_set_system：系统元数据(字段名等)字符集

MySQL中的字符集转换过程 
1、MySQL Server收到请求时将请求数据从character_set_client转换为character_set_connection； （qm解析：发送端将字符用A进行编码，Mysql接收到字节用character_set_client解码）
2、进行内部操作前将请求数据从character_set_connection转换为内部操作字符集，其确定方法如下： 
• 使用每个数据字段的CHARACTER SET设定值； 
• 若上述值不存在，则使用对应数据表的DEFAULT CHARACTER SET设定值(MySQL扩展，非SQL标准)； 
• 若上述值不存在，则使用对应数据库的DEFAULT CHARACTER SET设定值； 
• 若上述值不存在，则使用character_set_server设定值。 
3. 将操作结果从内部操作字符集转换为character_set_results。 

![](http://i.imgur.com/6gWJtFs.jpg)

## Mysql的编码转换

在Tomcat中接收的


## 使用 useUnicode=true&characterEncoding=utf-8 作用

useUnicode：是否使用Unicode字符集，如果参数characterEncoding设置为gb2312或gbk，本参数值必须设置为true，默认false
characterEncoding：当useUnicode设置为true时，指定字符编码。比如可设置为gb2312或gbk，默认false
## Java 连接 Mysql 数据库

1、引入mysql-connector-java

	<dependency>
	    <groupId>mysql</groupId>
	    <artifactId>mysql-connector-java</artifactId>
	    <version>5.1.38</version>
	</dependency>

2、Java连接数据库代码

	//1、加载驱动
	Class.forName("com.mysql.jdbc.Driver");
	//2、获得连接
	Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8", "root", "root");
	//3、创建Statement
	String sql = "select * from tb_user";
	PreparedStatement statement = connection.prepareStatement(sql);
	//4、执行sql语句
	ResultSet resultSet = statement.executeQuery();
	//5、处理结果集
	while (resultSet.next()) {
	    System.out.println(resultSet.getString(1) + " " + resultSet.getString(2));
	}
	//6、关闭资源
	if (connection != null) {
	    connection.close();
	}
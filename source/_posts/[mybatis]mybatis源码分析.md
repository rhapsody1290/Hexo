---
title: mybatis源码分析

date: 2017-06-20 14:09:00

categories:
- mybatis

tags:
- mybatis

---

摘自：http://blog.csdn.net/luanlouis/article/details/40422941

## mybatis优势

* 面向接口编程，定义接口通过动态代理的方式生成一个mapper实例
* 参数映射和动态SQL语句生成： 动态语句生成是 MyBatis 通过传入的参数值，使用 Ognl 来动态地构造SQL语句；参数映射指的是对于Java 数据类型和jdbc数据类型之间的转换
* SQL语句的执行以及封装查询结果集成List
* 事务管理、连接池管理机制、缓存机制、QL语句的配置方式

## 动态代理生成mapper

MyBatis和数据库的交互有两种方式：

1、使用传统的MyBatis提供的API；

![](http://i.imgur.com/UjHifnz.png)

创建一个和数据库打交道的SqlSession对象，然后根据Statement Id 和参数来操作数据库

2、使用Mapper接口

![](http://i.imgur.com/k6RCXIh.png)

根据 MyBatis 的配置规范配置好后，通过SqlSession.getMapper(XXXMapper.class) 方法，MyBatis 会根据相应的接口声明的方法信息，**通过动态代理机制生成一个 Mapper 实例**，我们使用Mapper 接口的某一个方法时，MyBatis 会根据这个方法的**方法名和参数类型，确定Statement Id**，底层还是通过SqlSession.select("statementId",parameterObject);或者SqlSession.update("statementId",parameterObject); 等 **SqlSession 的增删改查方法来实现对数据库的操作**

## SqlSession 的工作过程分析

1、开启一个数据库访问会话---创建SqlSession对象：

	SqlSession sqlSession = factory.openSession();  

MyBatis封装了对数据库的访问，把对数据库的会话和事务控制放到了SqlSession对象中。

2、为SqlSession传递一个配置的Sql语句 的Statement Id和参数，然后返回结果：

	List<Employee> result = sqlSession.selectList("com.louis.mybatis.dao.EmployeesMapper.selectByMinSalary",params);  

**MyBatis在初始化的时候，会将MyBatis的配置信息全部加载到内存中，使用 org.apache.ibatis.session.Configuration 实例来维护**。使用者可以使用sqlSession.getConfiguration() 方法来获取

	<select id="selectByMinSalary" resultMap="BaseResultMap" parameterType="java.util.Map" >  
	  select   
	    EMPLOYEE_ID, FIRST_NAME, LAST_NAME, EMAIL, SALARY  
	    from LOUIS.EMPLOYEES  
	    <if test="min_salary != null">  
	        where SALARY < #{min_salary,jdbcType=DECIMAL}  
	    </if>  
	</select>  

加载到内存中会生成一个对应的 **MappedStatement** 对象，然后会以key="com.louis.mybatis.dao.EmployeesMapper.selectByMinSalary" ，value为MappedStatement对象的形式维护到Configuration的一个Map中。当以后需要使用的时候，只需要**通过Id值来获取就可以了。**

从上述的代码中我们可以看到SqlSession的职能是：

SqlSession根据Statement ID, 在mybatis配置对象Configuration中获取到对应的MappedStatement对象，然后调用mybatis执行器来执行具体的操作。

**SqlSession的作用：**

作为MyBatis工作的主要顶层API，表示和**数据库交互的会话**，完成必要数据库**增删改查**功能

3、MyBatis执行器Executor根据SqlSession传递的参数执行query()方法

Executor.query()方法几经转折，最后会创建一个StatementHandler对象，然后将必要的参数传递给StatementHandler，使用StatementHandler来完成对数据库的查询，最终返回List结果集。

**Executor的功能和作用是：**

(1、根据传递的参数，**完成SQL语句的动态解析**，生成BoundSql对象，供StatementHandler使用；
(2、为查询创建**缓存**，以提高性能
(3、**创建JDBC的Statement连接对象，传递给StatementHandler对象，返回List查询结果。**

4、StatementHandler对象负责设置Statement对象中的查询参数、处理JDBC返回的resultSet，将resultSet加工为List 集合返回：

**StatementHandler对象主要完成两个工作：**

(1、对于JDBC的PreparedStatement类型的对象，创建的过程中，我们使用的是SQL语句字符串会包含若干个 ? 占位符，我们其后再**对占位符进行设值**。StatementHandler通过parameterize(statement)方法对Statement进行设值；    
(2、StatementHandler通过List<E> query(Statement statement, ResultHandler resultHandler)方法来完成执行Statement，和**将Statement对象返回的resultSet封装成List；**

对StatementHandler的query分析：

StatementHandler 的 List<E> query(Statement statement, ResultHandler resultHandler)方法的实现，是调用了ResultSetHandler的handleResultSets(Statement) 方法。**ResultSetHandler的handleResultSets(Statement) 方法会将Statement语句执行后生成的resultSet 结果集转换成List<E> 结果集：**

## 参考

http://www.cnblogs.com/luoxn28/p/5932648.html
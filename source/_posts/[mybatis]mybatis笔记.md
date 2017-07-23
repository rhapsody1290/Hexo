---
title: mybatis笔记

date: 2016-08-11 00:00:00

categories:
- mybatis

tags:
- Java
- JavaEE
- mybatis

---

## JDBC回顾

### JDBC连接数据库分为六步

	1. 加载驱动
	2. 创建连接
	3. 获取statement对象
	4. 执行sql语句
	5. 处理结果集
	6. 关闭资源

![](http://i.imgur.com/I20xxTm.png)

### 存在的问题及解决方案

	1、加载驱动问题：
	　　● 每次执行都加载驱动
	　　● 驱动名称，硬编码到java代码中，如果需要修改驱动，需要修改java文件
			—— 解决方案：将驱动名称放入到外部的配置文件
	2、数据库的连接信息，硬编码到java代码中
			—— 解决方案：外部配置文件
	3、sql语句设置参数的问题： 参数下标硬编码了，需要人为的去判断参数的位置
	4、遍历结果集：需要人工的判断字段名，以及个位置参数类型，不方便
			—— 是否可以将结果集直接映射到一个pojo对象中
	5、频繁的创建连接，关闭连接，导致资源浪费，影响性能
			—— 解决：连接池。

## mybatis介绍★★

![xxxx](http://i.imgur.com/rupfMQb.png)

* mybatis的前身是ibatis，改名后将版本升级为3.X
* Mybatis是类似Hiberate的 ***ORM框架***，位于***持久层***
* Mybatis是直接***基于JDBC做了简单的封装***，性能角度来看JDBC > mybatis > Hiberate

## mybatis整体架构

![xxxx](http://i.imgur.com/pHNaWys.png)

* mybatis-config.xml全局配置文件，包括数据库的连接参数、全局配置项
* mapper.xml有多个，配置sql语句
* SqlSessionFactory创建SqlSession
* SqlSession执行crud操作
* executor底层的执行器
* Mappered Statement为一个个的sql语句

## mybatis快速入门(参考官方文档)★★★★★

### github
https://github.com/rhapsody1290/MybatisStudy

### 安装

* 想要使用 MyBatis 只需将 mybatis-x.x.x.jar 文件置于 classpath 中
* 如果使用 Maven 构建项目，则需将下面的 dependency 置于 pom.xml 中  


	<dependency>
	  <groupId>org.mybatis</groupId>
	  <artifactId>mybatis</artifactId>
	  <version>3.2.8</version>
	</dependency>

### 从 XML 中构建 SqlSessionFactory

* 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的
* **SqlSessionFactory** 的实例可以通过 **SqlSessionFactoryBuilder** 获得
* SqlSessionFactoryBuilder 可以从 **XML 配置文件**或**一个预先定制的 Configuration 的实例**构建出 SqlSessionFactory 的实例（直接从Java程序中创建配置）

***从XML配置文件创建SqlSessionFactory实例***

从 XML 文件中构建 SqlSessionFactory 的实例非常简单，建议使用类路径下的资源文件进行配置。但是也可以使用任意的 InputStream 实例，包括字符串形式或 URL 形式的文件路径来配置。MyBatis 包含一个叫 Resources 的工具类，它包含一些静态方法，可使从 classpath 或其他位置加载资源文件更容易。

	String resource = "mybatis-config.xml";//指定配置文件的路径
	InputStream inputStream = Resources.getResourceAsStream(resource);//默认会从classes目录下寻找配置文件
	SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

***mybatis-config.xml——XML配置文件核心设置***

XML 配置文件（configuration XML）中包含了对 MyBatis 系统的***核心设置*** ，包含***获取数据库连接实例的数据源***（DataSource）和***决定事务范围和控制方式的事务管理器***（TransactionManager）。XML 配置文件的详细内容后面再探讨，这里先给出一个简单的示例：

	<!-- mybatis-config.xml -->
	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE configuration
	  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	  "http://mybatis.org/dtd/mybatis-3-config.dtd">
	<configuration>
	  <environments default="development">
	    <environment id="development">
	      <transactionManager type="JDBC"/>
	      <dataSource type="POOLED">
	        <property name="driver" value="${driver}"/>
	        <property name="url" value="${url}"/>
	        <property name="username" value="${username}"/>
	        <property name="password" value="${password}"/>
	      </dataSource>
	    </environment>
	  </environments>
	  <mappers>
	    <mapper resource="mapper.xml"/>
	  </mappers>
	</configuration>

<font color = "red">注意：上述配置文件采用了Property文件注入的方式，这里为了方便，我们可以直接写死连接数据库的参数</font>

![](http://img.blog.csdn.net/20160802171100530)

当然，XML 配置文件中还有很多可以配置的，上面的示例指出的则是***最关键的部分***。要注意 XML 头部的声明，需要用来验证 XML 文档正确性。

* environment 元素体中包含了事务管理和连接池的环境配置
* mappers 元素是包含一组 mapper 映射器（这些 mapper的 XML 文件包含了 SQL 代码和映射定义信息）

### mapper.xml——映射文件

Mapper.xml映射文件中定义了操作数据库的sql，每个sql是一个statement，映射文件是mybatis的核心

	<!-- mapper.xml -->	
	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE mapper
	        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<!--namespace：名字空间，唯一标识，不需要特别的起名称，只要保证所有的mapper中唯一即可-->
	<mapper namespace="cn.apeius.User">
	    <!--sql语句映射，也称作mapperedStatement
	        resultType：结果集对应的java类型，需要书写类的全路径
	        #{id}：相当于？，表示参数的占位-->
	    <select id="selectUser" resultType="User">
	        select * from tb_user where id = #{id}
	    </select>
	</mapper>

### 从 SqlSessionFactory 中获取 SqlSession

SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句

	//获取sqlsession
	SqlSession session = sqlSessionFactory.openSession();
    try {
		//sqlSession.selectOne("sql的id","sql中需要的参数")
        User user = (User) session.selectOne("cn.apeius.User.selectUser", 1);
        System.out.println(user);
    } finally {
		//关闭session
        session.close();
    }

### 可能的错误

![](http://i.imgur.com/X0igqa5.png)

问题原因：mybatis没有引入 mapper ，需要在 mybatis-config.xml 中使用mapper引入外部的mapper.xml

### mybatis使用步骤总结

![](http://i.imgur.com/9XiuQMK.png)

***java代码***
![](http://i.imgur.com/hURwgDx.png)

## 添加日志支持

### 导入依赖

	<dependency>
	    <groupId>org.slf4j</groupId>
	    <artifactId>slf4j-log4j12</artifactId>
	    <version>1.6.1</version>
	</dependency>

### 编写配置文件

***log4j.properties***

	log4j.rootLogger=DEBUG,A1
	log4j.logger.org.mybatis=DEBUG
	log4j.appender.A1=org.apache.log4j.ConsoleAppender
	log4j.appender.A1.layout=org.apache.log4j.PatternLayout
	log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n

## 完整的CRUD操作(未使用动态代理类)★★★★★

### 创建UserDao接口
		
	public interface UserDao {
	    /**
	     * 根据id查询用户信息
	     * @param id
	     * @return
	     */
	    public User queryUserById(Long id);
	    /**
	     * 查询所有用户
	     * @return
	     */
	    public List<User> queryAllUser();
	    /**
	     * 添加用户信息
	     * @param user
	     */
	    public void addUser(User user);
	    /**
	     * 修改用户信息
	     * @param user
	     */
	    public void updateUser(User user);
	    /**
	     * 根据id删除用户信息
	     * @param id
	     */
	    public void deleteUserById(Long id);
	}

### 创建UserDao的实现类

	public class UserDaoImpl implements UserDao {
	    private SqlSession sqlSession;
	    public UserDaoImpl(SqlSession sqlSession){
	        this.sqlSession = sqlSession;
	    }
	    public User queryUserById(Long id) {
	        return sqlSession.selectOne("user.queryUserById", id);
	    }
	
	    public List<User> queryAllUser() {
	        return sqlSession.selectList("user.queryAllUser");
	    }
	
	    public void addUser(User user) {
	        sqlSession.insert("user.addUser", user);
	    }
	
	    public void updateUser(User user) {
	        sqlSession.update("user.updateUser", user);
	    }
	
	    public void deleteUserById(Long id) {
	        sqlSession.delete("user.deleteUserById", id);
	    }
	}

### 编写User对应的Mapper.xml

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE mapper
	        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<!--
		1、namespace：名字空间，唯一标识，不需要特别的起名称，只要保证所有的mapper中唯一即可
		2、但为了与Mapper接口进行映射，一般为Mapper接口的完全限定名
		-->
	<mapper namespace="cn.apeius.mybatis.mapper.UserMapper">
	    <select id="queryUserById" resultType="cn.apeius.mybatis.domain.User">
	        select * from tb_user where id = #{id}
	    </select>

	    <!-- resultType为结果集映射的java的类型 -->
	    <select id="queryAllUser" resultType="cn.apeius.mybatis.domain.User">
	        select * from tb_user
	    </select>
	
	    <!-- #{}中的值为domain类中的属性名 -->
	    <insert id="addUser">
	        INSERT INTO tb_user VALUES (NULL,#{user_name},#{password},#{name},#{age},#{sex},#{birthday},NOW(),NOW())
	    </insert>
	    
	    <update id="updateUser">
	        UPDATE tb_user SET user_name = #{user_name},password = #{password},name = #{name}, age = #{age}, sex = #{sex}, birthday = #{birthday}, updated = NOW() WHERE id=#{id}
	    </update>
	
	    <delete id="deleteUserById">
	        delete from tb_user where id = #{id}
	    </delete>
	</mapper>

### 全部配置文件mybatis-config.xml加入UserMapper.xml

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE configuration
	        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	        "http://mybatis.org/dtd/mybatis-3-config.dtd">
	<configuration>
	    <environments default="development">
	        <environment id="development">
	            <transactionManager type="JDBC"/>
	            <dataSource type="POOLED">
	                <property name="driver" value="com.mysql.jdbc.Driver"/>
	                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
	                <property name="username" value="root"/>
	                <property name="password" value="root"/>
	            </dataSource>
	        </environment>
	    </environments>
	    <mappers>
	        <mapper resource="UserMapper.xml"/>
	    </mappers>
	</configuration>

### 编写测试用例JUNIT★★★★★★★

* 在UserDao接口中按快捷键Ctrl + Shift + T，快速创建UserDaoTest测试类
* add/update/delete需要commit


	public class UserDaoTest {
	    UserDao userDao;
	    SqlSession sqlSession;
		
		//初始操作	    
		@Before
	    public void setUp() throws Exception {
	        String resource = "mybatis-config.xml";
	        InputStream is = Resources.getResourceAsStream(resource);
	        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	        sqlSession = sqlSessionFactory.openSession();
	        this.userDao = new UserDaoImpl(sqlSession);
	    }

		@After
	    public void tearDown() throws Exception{
	        sqlSession.close();
	    }

	    @Test
	    public void testQueryUserById() throws Exception {
	        User user = userDao.queryUserById(1l);
	        System.out.println(user);
	    }
	
	    @Test
	    public void testQueryAllUser() throws Exception {
	        List<User> users = userDao.queryAllUser();
	        for(User user : users){
	            System.out.println(user);
	        }
	    }
	
	    @Test
	    public void testAddUser() throws Exception {
	        User user = new User();
	        user.setName("xxx");
	        user.setAge(18);
	        user.setSex(1);
	        user.setPassword("123456");
	        user.setUser_name("xxxx");
	        user.setBirthday(new Date());
	        userDao.addUser(user);
			//需要事务进行提交
	        sqlSession.commit();
	    }
	
	    @Test
	    public void testUpdateUser() throws Exception {
	        //先查询再更新
	        User user = userDao.queryUserById(15l);
	        user.setUser_name("二舅");
	        userDao.updateUser(user);
	        sqlSession.commit();
	    }
	
	    @Test
	    public void testDeleteUserById() throws Exception {
	        userDao.deleteUserById(17l);
	        sqlSession.commit();
	    }
	}

### 解决数据库字段名和实体类属性名不一致的问题
查询数据的时候，查不到userName的信息，原因：数据库的字段名是***user_name***，POJO中的属性名字是***username***，两端不一致，造成mybatis无法填充对应的字段信息

* 解决方案1：在sql语句中使用别名

![](http://i.imgur.com/lGbkLOR.png)

![](http://i.imgur.com/Ch9AR6z.png)

* 解决方案2： 参考后面的resultMap –mapper具体的配置的时候
* 解决方案3：参考驼峰匹配 --- mybatis-config.xml 的时候


## 动态代理mapper实现类★★★★★★★

### 思考CURD的dao中的问题
![](http://i.imgur.com/dokD9Lz.png)

1、上述开发过程为：接口->实现类->mapper.xml
2、可以发现在实现类中，使用mybatis的方式非常类似
　　●  查询 —— selectOne/selectList
　　●  查询 —— insert
　　●  修改 —— update
　　●  删除 —— delete
3、sql statement 硬编码到java代码中（举例，函数参数user.queryUserById与mapper.xml中id="queryUserById对应"）

**思考：能否只写接口，不书写实现类，只编写Mapper.xml即可**


答：因为在dao（mapper）的实现类中对sqlsession的使用方式很类似，mybatis提供了接口的动态代理

### 名称空间

* mapper.xml 根标签的 namespace 属性 称为名称空间。namespace的定义本身是没有限制的，只要不重复就行
* 如果希望使用mybatis通过的动态代理的接口，就需要namespace中的值，和需要对应的Mapper(dao)接口的全路径一致

![](http://i.imgur.com/1CDRzor.png)

### 通过sqlSession.getMapper(Class)

* 在mybatis中，dao层的命名规则，以后都修改成*mapper
![](http://i.imgur.com/vUHzBKf.png)

* 要求mapper中的statement,sql中的id 和方法中的名字一模一样才可以去调用
![](http://i.imgur.com/StHBgCi.png)

### 使用步骤

#### 定义接口

	package cn.apeius.mybatis.mapper;
	
	public interface UserMapper {
	    /**
	     * 根据id查询用户信息
	     * @param id
	     * @return
	     */
	    public User queryUserById(Long id);
	}

#### 编写mapper.xml文件

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE mapper
	        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
	        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
	<!--namespace：名字空间，唯一标识，不需要特别的起名称，只要保证所有的mapper中唯一即可-->
	<mapper namespace="cn.apeius.mybatis.mapper.UserMapper">
	    <select id="queryUserById" resultType="cn.apeius.mybatis.domain.User">
	        select * from tb_user where id = #{id}
	    </select>
	</mapper>

#### 调用

	public class UserMapperTest {
	    UserMapper userMapper;
	    SqlSession sqlSession;
	
	    @Before
	    public void setUp() throws Exception {
	        String resource = "mybatis-config.xml";
	        InputStream is = Resources.getResourceAsStream(resource);
	        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
	        this.sqlSession = sqlSessionFactory.openSession();
	        //this.userMapper = new UserDaoImpl(sqlSession);
	        this.userMapper = sqlSession.getMapper(UserMapper.class);
	    }
	
	    @Test
	    public void testQueryUserById() throws Exception {
	        User user = userMapper.queryUserById(1l);
	        System.out.println(user);
	    }
	}

### 使用动态代理总结★★★★★★★★★

![](http://i.imgur.com/k8rtmcd.png)

* 在名字空间"cn.apeius.mybatis.mapper.UserMapper"定义了一个名为"queryUserById"的映射语句，这样它就允许使用指定的完全限定名"cn.apeius.mybatis.mapper.UserMapper.queryUserById"来调用映射语句

		User user = sqlSession.selectOne("cn.apeius.mybatis.mapper.UserMapper.queryUserById",1L);
		System.out.println(user);

* 这和使用完全限定名调用Java对象的方法是相似的，这样命名可以直接映射到命名空间同名的Mapper类，并将select语句中的名字、参数和返回类型映射成Mapper类的方法，这样可以调用对应Mapper接口的方法
		
		UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
		User user = userMapper.queryUserById(1L);
		System.out.println(user);

* ***综上所述：namespace与Mapper接口映射，sql语句与mapper接口中的方法映射***
* ***一张表对应一个Doamin对象，一个Mapper接口（DAO），一个mapper.xml***。例如user表对应User.java、UserMapper.java、UserMapper.xml

## mybatis-config.xml 配置

子标签出现的顺序不能改变

![](http://i.imgur.com/q6iBImX.png)

### properties(读取外部的资源文件)

![](http://i.imgur.com/So9Ptdk.png)

***编写jdbc.properties文件***

	driver=com.mysql.jdbc.Driver
	url=jdbc:mysql://localhost:3306/mybatis
	username=root
	password=root

***mybatis-config.xml属性值注入***

	<environments default="development">
	    <environment id="development">
	        <transactionManager type="JDBC"/>
	        <dataSource type="POOLED">
	            <property name="driver" value="${driver}"/>
	            <property name="url" value="${url}"/>
	            <property name="username" value="${username}"/>
	            <property name="password" value="${password}"/>
	        </dataSource>
	    </environment>
	</environments>

### settings（设置）

***mapUnderscoreToCamelCase用法：***

* 开启驼峰匹配：从经典数据库的命名规则，到经典java命名规则的映射
* 经典数据库的命名规则： 
* 
* 
* 个单词之间使用下划线分割，例如：user_name
* java经典命名规则：驼峰样式，多个单词，后续的单词首字母大写 例如：userName
* 开启驼峰匹配：相当于去掉数据库名字中的下划线，然后在与java中的属性名进行对应

![](http://i.imgur.com/GKLEVn7.png)

### typeAliases
类型别名是为 Java 类型命名的一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余

***typeAliases的使用1***
* 使用这个配置，"User"可以用在任何使用"cn.apeius.mybatis.domain.User"的地方
* ***大小写不敏感***
* 缺点：每个一个类都去书写别名，麻烦

![](http://i.imgur.com/PLq7JS8.png)

***typeAliases的使用2——扫描包的方式***

每一个在包 cn.apeius.mybatis.domain 中的 Java Bean，在没有注解的情况下，会使用 Bean 的***首字母小写的非限定类名*** 来作为它的别名。比如 
cn.apeius.mybatis.domain.User 的别名为 User，且***大小写不敏感***

![](http://i.imgur.com/CTiAuKz.png)

若有注解，则别名为其注解值。

	@Alias("xxx")
	public class User {
		...
	}

### plugins（插件，又名拦截器）

MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis 允许使用插件来拦截的方法调用包括：

* Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
* ParameterHandler (getParameterObject, setParameters)
* ResultSetHandler (handleResultSets, handleOutputParameters)
* StatementHandler (prepare, parameterize, batch, update, query)

图：利用拦截器进行分页
![](http://i.imgur.com/DlB41sl.png)

### environments（环境）

#### 为什么要适应多种环境

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中，现实情况下有多种理由需要这么做

* 开发环境：开发人员日常开发的时候使用的环境
* 测试环境：测试人员测试的时候使用环境
* 预发布环境：几乎和线上环境一模一样，在上线之前在进行一次测试
* 生成环境：线上环境。 正式的java程序运行的环境

不过要记住：尽管可以配置多个环境，***每个 SqlSessionFactory 实例只能选择其一***

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

* ***每个数据库对应一个 SqlSessionFactory 实例***

#### 指定创建哪种环境

***配置多个环境***

<pre>
&lt;environments <font color='red'> default="development"</font>>
	&lt;environment <font color='blue'>id="development"</font>>
		&lt;transactionManager type="JDBC"/>
		&lt;dataSource type="POOLED">
			&lt;property name="driver" value="${driver}"/>
			&lt;property name="url" value="${url}"/>
			&lt;property name="username" value="${username}"/>
			&lt;property name="password" value="${password}"/>
		&lt;/dataSource>
	&lt;/environment>
	&lt;environment <font color='blue'>id="online"</font>>
		&lt;transactionManager type="JDBC"/>
		&lt;dataSource type="POOLED">
			&lt;property name="driver" value="${driver}"/>
			&lt;property name="url" value="${url}"/>
			&lt;property name="username" value="${username}"/>
			&lt;property name="password" value="${password}"/>
		&lt;/dataSource>
	&lt;/environment>
&lt;/environments>
</pre>

***代码指定创建那种环境***

	SqlSessionFactory factory = sqlSessionFactoryBuilder.build(reader, environment);

### mappers映射器

mappers映射器将 mapper.xml 文件配置到 mybatis 的环境中

#### 相对于类路径的资源引用

<pre>
&lt;!-- Using classpath relative resources -->
&lt;mappers>
	&lt;mapper resource="UserMapper.xml"/>
&lt;/mappers>
</pre>

#### 使用mapper接口路径

* 这里所谓的 ***mapper接口路径***，实际上就是 ***dao的接口路径***
* 在mybatis中，***通常把dao的包叫做mapper类名，也叫做mapper***
* 使用 class 方式去加载的时候，要求

	* 要求 mapper.xml 文件的名字和 mapper 接口的名字一致
	* 要求 mapper.xml 文件和 mapper 接口类在一个目录

![](http://i.imgur.com/T8Ghcku.png)
	
	<mapper class='cn.itcast.mybatis.mapper.UserMapper'/>

**问题:**
1、mapper.xml 和 java文件没有分离,明天和 spring 整合之后解决
2、需要一个一个的去加载 mapper

#### 使用mapper接口扫描包(常用)

扫描指定包下的所有接口，要求mapper.xml和接口在***同一个包下***，并且***名字相同***

<pre>
&lt;!-- Register all interfaces in a package as mappers -->
&lt;mappers>
	&lt;package name="cn.itcast.mybatis.mapper"/>
&lt;/mappers>
</pre>


<font color='red'>***缺点:***</font>

1、如果包的路径有很多？ 明天spring整合的时候解决
2、mapper.xml和mapper.java没有分离

## Mapper.xml

![](http://i.imgur.com/MYBT4po.png)

### CURD操作

#### select——书写select语句

![](http://i.imgur.com/6fIzsFk.png)

* id属性（必须）：当前名称空间下的statement的***唯一标识***；要求id和mapper接口中的方法的***名字一致***
* resultType：将结果集映射为java的对象类型，和 resultMap 二选一（必须）
* ***parameterType：传入参数类型。可以省略***

#### insert——书写insert语句

![](http://i.imgur.com/Vzy1hda.png)

* id属性：必须的，当前名称空间下的statement的唯一标识(必须属性)
* parameterType：传入的参数类型，可以省略
* 标签内部：具体的sql语句
* 使用#{} 去替换一个变量
* 如果需要数据库影响的行数，可以***直接在接口上定义返回值Integer*** 即可

#### 获取自增的id的值

插入数据后获得自增长的id号

![](http://i.imgur.com/8URkb5u.png)

* useGenerateKeys：true
* keyColumn:主键列的名字
* keyProperty：实体类对应的属性

测试

![](http://i.imgur.com/i6Rwlo0.png)

#### update

![](http://i.imgur.com/dh6mcwe.png)

* id属性：当前名称空间下的statement的唯一标识(必须属性)
* parameterType：传入的参数类型，可以省略
* 标签内部：具体的sql语句
* 使用#{} 去替换一个变量

#### delete

![](http://i.imgur.com/bdcNIic.png)

* id属性：当前名称空间下的statement的唯一标识(必须属性)
* parameterType：传入的参数类型，可以省略
* 标签内部：具体的sql语句
* 使用#{} 去替换一个变量

### #{}的用法及传入多个参数

* `#{}` 存在mapper.xml中的sql语句部分，标识该位置可以接受参数信息，***相当于?占位符***
* 传入的参数和参数名无关，问题：如果有多个参数该怎么办？

![](http://i.imgur.com/T5wdiqe.png)

方法一：使用0，1……自然数取出对应的数据，0表示第一个参数

	select * from tb_user where user_name = #{0} and password = #{1}

方法二：使用param1，param2……param1表示第一个参数

	select * from tb_user where user_name = #{param1} and password = #{param2}

***方法三（常用）：在方法的定义上使用@param为传入的参数定义一个名字***

* 在UserMapper接口中声明函数login，使用@param为每个参数定义一个名字


	public User login(@Param("username") String username, @Param("password") String password);

* sql语句在#{}中填入定义的名字
	

	select * from tb_user where user_name = #{username} and password = #{password}

* 测试


	@Test
	public void testLogin(){
	    User user = userMapper.login("zhangsan","123456");
	    System.out.println(user);
	}

### ${}的用法及传入多个参数

${}的用法：仍然是接受传递的参数，但是${}是sql语句的拼接

***需求***

查询表中的信息，有时候从历史表中去查询数据，有时候需要去新的表去查询数据，希望使用1个方法来完成操作

* 方法一：$在取值时，使用value表示传入的参数值，不建议写

	* mapper接口
			public List<User> queryUserByTableName(String tableName);
	* mapper.xml
			select * from ${value}
	* 测试
			@Test
		    public void testQueryUsersByTableName(){
		        List<User> users = userMapper.queryUserByTableName("tb_user");
		        for(User u : users){
		            System.out.println(u);
		        }
		    }


* 方法二（常用）：$在取值时，可以在mapper接口的参数之前使用@Param主键为当前参数指定一个名字

	* mapper接口
			public List<User> queryUserByTableName(@Param("tableName") String tableName);
	* mapper.xml
			select * from ${tableName}
	* 测试
			@Test
			public void testQueryUsersByTableName(){
			    List<User> users = userMapper.queryUserByTableName("tb_user");
			    for(User u : users){
			        System.out.println(u);
			    }
			}

### 面试题：#{}与${}的区别

![](http://i.imgur.com/h9QffWS.png)

### resultMap与resultType用法★★★★★★★★

#### 从SQL查询结果到领域模型实体　

***从SQL查询结果集到JavaBean或POJO实体的过程：***

1. 通过JDBC查询得到ResultSet对象
2. 遍历ResultSet对象并将每行数据暂存到HashMap实例中，以结果集的字段名或字段别名为键，以字段值为值
3. 根据ResultMap标签的type属性***通过反射实例化领域模型***
4. 根据ResultMap标签的type属性和id、result等标签信息将HashMap中的键值对，填充到领域模型实例中并返回

#### resultType

* 在MyBatis进行查询映射的时候，其实查询出来的每一个属性都是放在一个对应的Map里面的，其中键是属性名，值则是其对应的值
* 当提供的返回类型属性是resultType的时候，MyBatis会将Map里面的键值对取出赋给resultType所指定的对象对应的属性

#### resultMap

返回类型直接是一个ResultMap主要用在进行复杂联合查询上

***简单 resultMap 配置***

	<resultMap id="BlogResult" type="com.liulanghan.Blog" >    
	    <id column="id" property="id"/>    
	    <result column="title" property="title"/>    
	    <result column="content" property="content"/>    
	    <result column="owner" property="owner"/>    
	</resultMap>   
	   
	<select id="selectBlog" parameterType="int" resultMap="BlogResult">    
	      select * from t_blog where id = #{id}    
	</select>  

* 结果集的列比resultMap多会报错么？ —— 不会,只映射resultMap中有的列
* 结果集的列比resultMap少会报错么？ —— 不会,只映射结果集中有的列


***高级结果映射***
见高级查询

## Mybatis学习路线总结★★★★★★★★★★

![](http://i.imgur.com/EbzMZer.png)

## 一级缓存

在mybatis中，一级缓存默认是开启的，并且无法关闭

![](http://i.imgur.com/BQQ7iX7.png)

***一级缓存满足条件：***
1. 同一个session中
2. 相同的SQL和参数

### 测试

1、测试代码
 
![](http://i.imgur.com/ymQQ6nN.png)

2、日志输出

![](http://i.imgur.com/o1OxJMa.png)

### 使用session.clearCache()强制查询不缓存

1、代码

![](http://i.imgur.com/2m1FkUi.png)

2、日志输出：
 
![](http://i.imgur.com/IrbSbCH.png)

向数据库发送两次查询语句

### 执行update,delete,insert 语句的时候，会刷新缓存

1、代码

![](http://i.imgur.com/TxLA7bj.png)

2、日志

![](http://i.imgur.com/CBcMlvg.png)


## 二级缓存

mybatis 的二级缓存的 ***作用域*** 是一个mapper的***namespace*** ，同一个namespace中查询sql可以从缓存中命中

### 开启二级缓存

需要在mapper.xml 中加入如下：

	<cache />

### 测试二级缓存

* 查询一次
* 关闭sqlSession
* 重新打开sqlSession
* 继续查询，观察是否发送sql语句

1、代码

![](http://i.imgur.com/Ny0NUin.png)

2、日志
　　
![](http://i.imgur.com/e489uAs.png)

### 二级缓存的高级配置

![](http://i.imgur.com/K1YSI2j.png)

### 关闭二级缓存

在全局的mybatis-config.xml 中去关闭二级缓存

![](http://i.imgur.com/RW4Q6F3.png)

![](http://i.imgur.com/UcpX9OF.png)

***测试***

![](http://i.imgur.com/Ny0NUin.png)

* 日志会有两次sql语句查询

## 高级查询★★★★★★

### 官方文档

http://blog.csdn.net/tayanxunhua/article/details/19194607

![](http://i.imgur.com/uvS6OXb.png)

### 数据库设计

1、用户表
2、订单表
3、产品表

思路：

* 一个用户可以有多个订单，某个订单只属于一个用户：需在订单表中设置指向用户表的外键
* 一个订单中有多个产品，一个产品可以属于多个订单，关系是多对多：建立中间表，指向订单表和产品表的外键

### 表之间的关系，以订单表作为出发点

![](http://i.imgur.com/rnuPgvZ.png)

订单表与用户表：1：1

* 一个订单只能属于一个人 1：1
* 一个用户可以有多个订单 1：N

订单表与商品表：N:N

* 一个订单可以有多个商品 1：N
* 一个商品可以属于多个订单表 1：N

订单表与订单详情：1：N

* 一个订单有多个订单详情
* 一个订单详情只属于一个订单

<font color='red'>***总结，以订单的角度看，他们的关系是：***</font>

* 订单和人是一对一关系
* 订单和订单详情是一对多的关系
* 订单和商品是多对多的关系

### 自动映射配置：autoMapping详解

http://www.cnblogs.com/TheViper/p/4480765.html
http://www.zhihu.com/question/41983738/answer/98829976

### 一对一查询

* 查询订单，并且查询出下单人的信息
* 核心思想：面向对象的思想，在Order对象中添加User对象
* **association关联元素**处理一对一关联，比如：一个博客只有一个作者，一张订单只属于一个用户

**方法一：关联的嵌套查询 （select联合查询，懒加载）**

Domain

<pre>
public class Order {
    private int id;
    <font color='red'>private User user;//private int user_id;</font>
    private String order_number;
}
</pre>

定义OrderMapper接口

	public interface OrderMapper {
		public Order queryOrderAndUserLazy(String orderNumber);
	}

Ordermapper.xml

    <!--1、查询订单，属性中包含对象必须使用resultMap-->
    <select id="queryOrderAndUserLazy" resultMap="lazyOrderUserResultMap">
        SELECT * FROM tb_order WHERE order_number = #{order_number}
    </select>

    <!--2、指定Order中user成员变量的详细装配方式，即根据user_id查询User对象-->
    <resultMap id="lazyOrderUserResultMap" type="Order">
		<!-- 一个 ID 结果，标记结果作为 ID 可以帮助提高整体效能 -->
        <id property="id" column="id"></id>
		<!--resultMap必须制定javaType，如果你映射到一个JavaBean，MyBatis通常可以断定类型-->
        <association property="user" javaType="User" column="user_id" select="queryUserById"/>
    </resultMap>

    <!--3、查询用户-->
    <select id="queryUserById" resultType="User">
        SELECT * FROM tb_user where id = #{id}
    </select>

测试

	@Test
    public void testQueryOrderAndUserByOrderNumber() throws Exception {
        Order order = orderMapper.queryOrderAndUserLazy("20140921001");
        System.out.println(order);
        System.out.println(order.getUser());
    }

**有两个查询语句**：一个来加载订单，另外一个来加载用户, 而且订单的结果映射描述了"queryUserById"语句应该被用来加载它的 User 属性。  

**其他所有的属性将会被自动加载，假设它们的列和属性名相匹配**。这种方式很简单, 但是对于大型数据集合和列表将不会表现很好，就会出现"N+1"的问题。

"N+1"的问题可以是这样引起的：     
你执行了一个单独的 SQL 语句来获取结果列表(就是"+1")——查询Order语句
对返回的每条记录,你执行了一个查询语句来为每个加载细节(就是"N")——查询User语句
这个问题会导致成百上千的 SQL 语句被执行，这通常不是期望的

MyBatis 能**延迟加载**这样的查询就是一个好处,因此你可以分散这些语句同时运行的消耗。然而,如果你加载一个列表,之后迅速迭代来访问嵌套的数据,你会调用所有的延迟加载,这样的行为可能是很糟糕的。所以还有另外一种方法

**方法二：关联查询**

方式一：属性全部重命名，手动映射

* 代替了执行一个分离的语句,我们联合订单表和用户表在一起
* 所有结果被唯一而且清晰的名字来重命名，这使得映射非常简单
* 缺点是所有字段都要手动映射，操作简单，但工作量较大，可以使用自动映射


    <!--联合查询订单和用户，一对一关系-->
    <select id="queryOrderAndUserByOrderNumber" resultMap="orderUserResultMap">
        SELECT
            o.id as order_id,
            o.user_id as order_user_id,
            o.order_number as order_order_number,
            u.id as user_id,
            u.user_name as user_user_name,
            u.password as user_password,
            u.name as user_name,
            u.age as user_age,
            u.sex as user_sex,
            u.birthday as user_birthday,
            u.created as user_created,
            u.updated as user_updated
        FROM
            tb_order o
        LEFT JOIN tb_user u ON u.id = o.user_id
        WHERE
          o.order_number = #{order_number}
    </select>
    <resultMap id="orderUserResultMap" type="Order" autoMapping="true">
        <id property="id" column="order_id"/>
        <result property="order_number" column="order_order_number"/><!-- 有顺序要求，必须放在id后-->
        <association property="user" javaType="User">
            <id property="id" column="user_id"/>
            <result property="user_name" column="user_name"/>
            <result property="password" column="user_password"/>
            <result property="name" column="user_name"/>
            <result property="age" column="user_age"/>
            <result property="sex" column="user_sex"/>
            <result property="birthday" column="user_birthday"/>
            <result property="created" column="user_created"/>
            <result property="updated" column="user_updated"/>
        </association>
    </resultMap>

方式二：自动映射，冲突字段自动命名，手动映射，其余元素自动匹配（推荐）★★★★★

* 当自动匹配结果的时候，Mybatis会获取列名，并且查找一个相同的属性（忽略大小写）。这意味着命名为ID的列和命名为id的属性被查找到的时候，Mybatis将会把列ID的值赋给属性id
* 通常数据库列名命名的时候使用大写和下划线并且Java属性常常依据驼峰命名法。为了保证在他们之间的自动匹配要设置属性 mapUnderscoreToCamelCase 为 true
* 对于每一个result map，所有的在ResultSet中有的并且**没有被手动匹配的列将会被自动的匹配**。默认不开启，**设置 autoMapping="true"开启**
* <font color='red'>***冲突字段重命名，自动赋值，其余元素自动匹配***</font>


	<!--联合查询订单和用户，一对一关系-->
    <select id="queryOrderAndUserByOrderNumber" resultMap="orderUserResultMap">
        SELECT
          *,u.id as user_id,o.id as order_id
        FROM
          tb_order o
        LEFT JOIN tb_user u ON u.id = o.user_id
        WHERE
          o.order_number = #{order_number}
    </select>
    <resultMap id="orderUserResultMap" type="Order" autoMapping="true">
        <id property="id" column="order_id"/>
        <association property="user" javaType="User" autoMapping="true">
            <id property="id" column="user_id"/>
        </association>
    </resultMap>

方式三：外部结果映射

使用外部的结果映射元素来映射关联，使得User结果映射可以重用。如果你不需要重用它的话，或者你仅仅引用你所有的结果映射合到一个单独描述的结果映射中，即方式二

    <!--联合查询订单和用户，一对一关系-->
    <select id="queryOrderAndUserByOrderNumber" resultMap="orderUserResultMap">
        SELECT
        *,u.id as user_id,o.id as order_id
        FROM
        tb_order o
        LEFT JOIN tb_user u ON u.id = o.user_id
        WHERE
        o.order_number = #{order_number}
    </select>

    <resultMap id="orderUserResultMap" type="Order" autoMapping="true">
        <id property="id" column="order_id"/>
        <association property="user" javaType="User" resultMap="user"/>
    </resultMap>

    <resultMap id="user" type="User" autoMapping="true">
        <id property="id" column="user_id"/>
    </resultMap>

### 一对多查询

查询订单，查询出下单人信息并且查询出订单详情

* 订单表、用户表、订单详情三表联合查询
* 映射结果为Order类，填充user和orderDetails属性对象

Domain

	public class Order {
	    private int id;
	    private User user;//private int user_id;
	    private String order_number;
	    private List<OrderDetail> orderDetails;
	}

定义OrderMapper接口

	public interface OrderMapper {
		public Order queryOrderAndUserAndDetailByOrderNumber(String orderNumber);
	}

OrderMapper.xml

	<!--collection：1：N-->
    <select id="queryOrderAndUserAndDetailByOrderNumber" resultMap="orderUserDetailResultMap">
        SELECT
            *,o.id as order_id,d.id as detail_id
        FROM
            tb_order o
        LEFT JOIN tb_user u ON u.id = o.user_id
        LEFT JOIN tb_orderdetail d ON o.id = d.order_id
        WHERE o.order_number = #{order_number}
    </select>
    <resultMap id="orderUserDetailResultMap" type="Order" autoMapping="true">
        <id property="id" column="order_id"/>
        <association property="user" javaType="User" autoMapping="true">
            <id property="id" column="user_id"/>
        </association>
        <!--collection 定义集合
            property：集合的属性名
            javaType：集合的类型
            ofType：集合中成员类型
            子标签内：填id和result
        -->
        <collection property="orderDetails" javaType="List" ofType="OrderDetail" autoMapping="true">
            <id property="id" column="detail_id"/>
        </collection>
    </resultMap>

测试

	Order order = orderMapper.queryOrderAndUserAndDetailByOrderNumber("20140921001");
    System.out.println(order);
    List<OrderDetail> orderDetails = order.getOrderDetails();
    for(OrderDetail d : orderDetails)
        System.out.println(d);

### 多对多查询

查询订单，查询出下单人信息并且查询出订单详情中的商品数据

doamin

	public class OrderDetail {
	    private int id;
	    private Order order;//private int order_id;
	    private Item item;//private int item_id;
	    private double total_price;
	    private int status;
	}

定义OrderMapper

	public interface OrderMapper {
	    public Order queryOrdreAndUserAndDetailAndItemByOrderNumber(String orderNumber);
	}

OrderMapper.xml

	<!--查询订单，查询出下单人信息并且查询出订单详情中的商品数据 N:N-->
    <select id="queryOrdreAndUserAndDetailAndItemByOrderNumber" resultMap="orderUserDetailItemResultMap">
        SELECT
            *
        FROM
            tb_order o
        LEFT JOIN tb_user u ON u.id = o.user_id
        LEFT JOIN tb_orderdetail d ON o.id = d.order_id
        LEFT JOIN tb_item i ON d.item_id = i.id
        WHERE o.order_number = #{order_number}
    </select>
    <resultMap id="orderUserDetailItemResultMap" type="Order" autoMapping="true">
        <id property="id" column="order_id"/>
        <association property="user" javaType="User" autoMapping="true">
            <id property="id" column="user_id"/>
        </association>
        <collection property="orderDetails" javaType="List" ofType="OrderDetail" autoMapping="true">
            <id property="id" column="detail_id"/>
            <association property="item" javaType="Item" autoMapping="true">
                <id property="id" column="item_id"></id>
            </association>
        </collection>
    </resultMap>

测试

	@Test
    public void testQueryOrdreAndUserAndDetailAndItemByOrderNumber(){
        Order order = orderMapper.queryOrdreAndUserAndDetailAndItemByOrderNumber("20140921001");
        System.out.println(order);
    }

### resultMap的继承

![](http://i.imgur.com/R9JdYy3.png)

## 延迟加载(未成功)

### 开启延迟加载

	<settings>
        <!-- 打开延迟加载的开关 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <!-- 将积极加载改为消极加载，即延迟加载 -->
        <setting name="aggressiveLazyLoading" value="false"/>
    </settings>

* lazyLoadingEnabled：true使用延迟加载，false禁用延迟加载，默认为false
* aggressiveLazyLoading：
	* 默认true，当有一个进行加载的时候，就会把所有没有加载的属性加载进来
	* false按需加载，只有调用对应的getter的才会去加载

### 添加cglib

	<dependency>
			<groupId>cglib</groupId>
			<artifactId>cglib</artifactId>
			<version>3.1</version>
	</dependency>

### 接口定义

	public interface OrderMapper {
	    public Order queryOrderAndUserLazy(String orderNumber);
	}

### OrderMapper.xml

<pre>
&lt;mapper namespace="cn.apeius.mybatis.mapper.OrderMapper">
	&lt;select id="queryOrderAndUserLazy" resultMap="<font color = ' red'>lazyOrderUserResultMap</font>">
		SELECT * FROM tb_order WHERE order_number = #{order_number}
	&lt;/select>
	&lt;resultMap id="<font color = 'red'>lazyOrderUserResultMap</font>" type="Order" autoMapping="true">
		&lt;id property="id" column="id">&lt;/id>
			&lt;-- columnw中的user_id作为参数传入queryUserById-->
		&lt;association property="user" javaType="User" column="user_id" select="<font color = 'blue'>queryUserById</font>"/>
	&lt;/resultMap>
	&lt;select id="<font color = 'blue'>queryUserById</font>" resultType="User">
		SELECT * FROM tb_user where id = #{id}
	&lt;/select>
&lt;/mapper>
</pre>

### 测试

    @Test
    public void testQueryOrderAndUserLazy(){
        OrderMapper orderMapper = sqlSession.getMapper(OrderMapper.class);
        Order order = orderMapper.queryOrderAndUserLazy("20140921001");
        System.out.println(order.getOrder_number());
        System.out.println(order.getUser());
    }

## 分页插件

* Mybatis提供了plugin机制，***允许我们在mybatis的原有流程上加入自己的逻辑***，加上分页逻辑可以方便实现分页
* 原理是利用了拦截器，mybatis支持的拦截接口有四个：Executor、ParameterHandler、ResultSetHandlet、StatementHandler

### plugin实现原理

![](http://i.imgur.com/qP9KCdP.png)

原来的sql语句:

	select * from tb_user

经过拦截器后，变为:
	
	selectd * from tb_user limit 1,10

从而实现分页

### 使用PageHelper实现分页

PageHelper实现了通用的分页查询，其支持的数据库有Mysq、Oracle、DB2、PostgreSQL等主流数据库

### 导入依赖

	<dependency>
		<groupId>com.github.pagehelper</groupId>
		<artifactId>pagehelper</artifactId>
		<version>3.7.5</version>
	</dependency>
	<dependency>
		<groupId>com.github.jsqlparser</groupId>
		<artifactId>jsqlparser</artifactId>
		<version>0.9.1</version>
	</dependency>

### 配置插件

	<plugins>
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <!--数据库方言-->
            <property name="dialect" value="mysql"/>
            <!--设置为true时，使用RowBounds分页会进行count查询，即会去查询出总数-->
            <property name="rowBoundsWithCount" value="true"/>
        </plugin>
    </plugins>

### 在执行查询时设置分页参数

	@Test
    public void testPage(){
        //开启分页，第一个参数：当前的页数；第二个参数：每页几条数据
        PageHelper.startPage(2,5);
		//紧接着的第一个查询会被执行分页
        List<User> users = userMapper.queryAllUser();
        for(User u : users){
            System.out.println(u);
        }

        //获取分页信息
        PageInfo<User> pageInfo = new PageInfo<User>(users);
		pageInfo.getList();//与users为同一个引用
        System.out.println("总记录数：" + pageInfo.getTotal());
        System.out.println("总页数 " + pageInfo.getPages());
        System.out.println("当前页数 " + pageInfo.getPageNum());
        System.out.println("每页多少数据 " + pageInfo.getPageSize());
        System.out.println("当前页的数据条目数 " + pageInfo.getSize());
    }

## mybatis和spring整合

### 官方文档

https://github.com/rhapsody1290/MybatisStudy/tree/master/doc/%E4%B8%AD%E6%96%87%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3/Mybatis-Spring-1.2.2

### 什么是MyBatis-Spring

* MyBatis-Spring 会帮助你将 MyBatis 代码 ***无缝地整合*** 到 Spring 中，使用这个类库中的类, Spring 将会***加载必要的 MyBatis 工厂类和 session 类***
* 这个类库也提供一个简单的方式来注入 MyBatis 数据映射器和 SqlSession 到业务层的 bean 中
* 它也会处理事务, 翻译 MyBatis 的异常到 Spring 的 DataAccessException 异常中
* 最终,它并不会依赖于 MyBatis,Spring 或 MyBatis-Spring 来构建应用程序代码

### 导入依赖

mybatis-spring整合包

	<dependency>
	  <groupId>org.mybatis</groupId>
	  <artifactId>mybatis-spring</artifactId>
	  <version>1.2.2</version>
	</dependency>

c3p0依赖，一个开源的JDBC连接池

	<dependency>
		<groupId>c3p0</groupId>
		<artifactId>c3p0</artifactId>
		<version>0.9.1.2</version>
	</dependency>

### c3p0.properties(数据源)

	c3p0.driverClass=com.mysql.jdbc.Driver
	c3p0.url=jdbc:mysql://localhost:3306/mybatis
	c3p0.user=root
	c3p0.password=root

### 配置spring的配置文件applicationContext.xml

* 引入配置文件c3p0.properties
* 配置连接池
* 配置sqlSessionFactory
	* 引入数据源
	* 引入mybatis的全局配置
	* 包扫描方式配置别名typeAliasesPackage
	* mapper.xml所在路径(Mapper.xml和java代码分离)
* mapper接口所在的包


	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <!-- 引入配置文件 -->
	    <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	        <property name="locations">
	            <list>
	                <value>classpath:c3p0.properties</value>
	            </list>
	        </property>
	    </bean>
	
	    <!-- 配置连接池 -->
	    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
	        <property name="driverClass" value="${c3p0.driverClass}"></property>
	        <property name="jdbcUrl" value="${c3p0.url}"></property>
	        <property name="user" value="${c3p0.user}"></property>
	        <property name="password" value="${c3p0.password}"></property>
	    </bean>
	
	    <!--配置sqlSessionFactory。SqlSessionFactoryBean 是用于创建 SqlSessionFactory 的-->
	    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	        <!--引入数据源-->
	        <property name="dataSource" ref="dataSource" />
	        <!--引入mybatis的全局配置-->
	        <property name="configLocation" value="classpath:mybatis-config.xml"/>
	        <!--mybatis别名包-->
	        <property name="typeAliasesPackage" value="cn.apeius.mybatis.domain"/>
	        <!--mapper.xml所在路径,可以使mapper接口与mapper.xml分离；可以使用通配符，**表示所有目录-->
	        <property name="mapperLocations" value="classpath*:mapper/**/*.xml" />
	    </bean>
	
	    <!--指定扫描包，mapper接口所在的包-->
	    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	        <!--多个扫描包可以通过逗号或分号进行分割-->
	        <property name="basePackage" value="cn.apeius.mybatis.mapper" />
	    </bean>
	
	</beans>

### mybatis-config.xml

	<?xml version="1.0" encoding="UTF-8" ?>
	<!DOCTYPE configuration
	        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
	        "http://mybatis.org/dtd/mybatis-3-config.dtd">
	<configuration>
	    <settings>
	        <!-- 打开延迟加载的开关 -->
	        <setting name="lazyLoadingEnabled" value="true"/>
	        <!-- 将积极加载改为消极加载，即延迟加载 -->
	        <setting name="aggressiveLazyLoading" value="false"/>
	    </settings>
	    <plugins>
	        <plugin interceptor="com.github.pagehelper.PageHelper">
	            <!--数据库方言-->
	            <property name="dialect" value="mysql"/>
	            <!--设置为true时，使用RowBounds分页会进行count查询，即会去查询出总数-->
	            <property name="rowBoundsWithCount" value="true"/>
	        </plugin>
	    </plugins>
	</configuration>

***问题：为什么配置SqlSessionFactory时，class为SqlSessionFactory***

* 在基本的 MyBatis 中, SqlSessionFactory 可以使用 SqlSessionFactoryBuilder 来创建，而在 MyBatis-Spring 中,则使用 SqlSessionFactoryBean 来替代
* SqlSessionFactoryBean 实现了 Spring 的 FactoryBean 接口，这就说明了由 Spring 最终创建的 bean ***不是 SqlSessionFactoryBean 本身*** ,  而是工厂类的 getObject()返回的方法的结果。这种情况下,Spring 将会在应用启动时为你 创建 SqlSessionFactory 对象,然后将它以 SqlSessionFactory 为名来存储

### mybatis和spring整合测试

    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserMapper userMapper = context.getBean(UserMapper.class);
    List<User> users = userMapper.queryAllUser();
    for(User user : users){
        System.out.println(user.getId() + " " + user.getUser_name());
    }

## mapper整合servcie

### servcie接口

	public interface UserServiceInter {
	
	    public List<User> queryAll();
	}

### service实现

	@Service
	public class UserServiceImp implements UserServiceInter {
	
	    @Autowired
	    private UserMapper userMapper;
	
	    @Override
	    public List<User> queryAll() {
	        return userMapper.queryAllUser();
	    }
	}

### applicationContext.xml配置

使service属性自动注入
	
	<context:component-scan base-package="cn.apeius.mybatis.service.imp"></context:component-scan>

### 测试

	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
	UserServiceInter userServiceInter = context.getBean(UserServiceImp.class);
	List<User> users = userServiceInter.queryAll();
	for(User user : users){
	    System.out.println(user);
	}

## mybatis整合事务管理（未成功）

### 配置事务管理

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
	       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
		http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">
	
	    <!-- 定义事务管理器 -->
	    <bean id="transactionManager"
	          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	        <property name="dataSource" ref="dataSource" />
	    </bean>
	
	    <!-- 定义事务策略 -->
	    <tx:advice id="txAdvice" transaction-manager="transactionManager">
	        <tx:attributes>
	            <!--所有以query开头的方法都是只读的 -->
	            <tx:method name="query*" read-only="true" />
	            <!--其他方法使用默认事务策略 -->
	            <tx:method name="*" />
	        </tx:attributes>
	    </tx:advice>
	
	    <aop:config>
	        <!--pointcut元素定义一个切入点，execution中的第一个星号 用以匹配方法的返回类型，
	            这里星号表明匹配所有返回类型。 com.abc.dao.*.*(..)表明匹配cn.itcast.mybatis.service包下的所有类的所有方法 -->
	        <aop:pointcut id="myPointcut" expression="execution(* cn.apeius.mybatis.service.*.*(..))" />
	        <!--将定义好的事务处理策略应用到上述的切入点 -->
	        <aop:advisor advice-ref="txAdvice" pointcut-ref="myPointcut" />
	    </aop:config>
	
	</beans>

### service

	@Service
	public class UserServiceImp implements UserServiceInter {
	
	    @Autowired
	    protected UserMapper userMapper;
	
	    @Override
	    public List<User> queryAll() {
	        return userMapper.queryAllUser();
	    }
	
	    @Override
	    public void saveUser(User user) {
	        userMapper.addUser(user);
	        //int a = 9 / 0;
	    }
	}

### 测试

	@Before
    public void setUp(){
        ApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"applicationContext.xml","applicationContext-transaction.xml"});
        userServiceInter = context.getBean(UserServiceImp.class);
    }

	@Test
    public void testSaveUser(){
        User user = new User();
        user.setUser_name("陈冠希");
        user.setName("dsfa");
        userServiceInter.saveUser(user);
    }
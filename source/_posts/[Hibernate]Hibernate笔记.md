---
title: Hibernate笔记

date: 2016-08-12 17:55:00

categories:
- Hibernate

tags:
- Hibernate
- Java

---

## Hibernate简介

* ***Hibernate是对jdbc进行轻量级封装的ORM框架，位于数据持久层***
* ORM全称对象关系映射，框架在Java对象与关系数据库之间建立 ***映射***，以实现 ***直接存取*** Java对象

## Hibernate参考资料

Hibernate3.2API.chm
https://github.com/rhapsody1290/Hibernate_Study/blob/master/doc/Hibernate3.2API.chm

hibernate3.2_reference.pdf
https://github.com/rhapsody1290/Hibernate_Study/blob/master/doc/hibernate3.2_reference.pdf
## ORM概述

* 传统数据持久化编程中，需要使用JDBC并配合大量的SQL语句。***JDBC API与SQL语句夹杂在一起***，开发效率都很低下
* 后来出现DAO模式，所有的JDBC API和SQL语句均移到了DAO层，***但仍然需要编写大量的SQL语句***
* ORM框架的思路是通过配置文件，将Java对象 ***映射*** 到关系型数据库，***自动生成SQL语句*** 并执行
* 举个例子，***插入数据*** 时就是把POJO的各个属性拼装成SQL语句，保存进数据库；***读取数据*** 时，就是用SQL语句读取数据库，然后拼装成POJO对象返回

## 为什么需要Hibernate？

* 使用jdbc操作数据库，SQL语句编写比较麻烦
* 切换数据库时需要重写SQL语句
* 我们程序员希望不关注数据库本身，而是关注业务本身

![](http://i.imgur.com/0aHzHBf.png)

　　引入Hibernate后，程序员在业务逻辑中使用hql语句（一种万能语句），Hibernate会自动完成数据库的操作，这种方式程序员只需关注业务本身，提高开发效率，程序也具有很好的移植性
　　***学习Hibernate关键是1、Hibernate API 2、Hibernate核心配置文件 3、对象关系映射文件***

## Hibernate开发的三种方式

![](http://i.imgur.com/9vM5xkp.png)

数据库中的表与java domain对象，通过Hibernate的对象关系映射文件关联起来，该文件会说明表和对象的关系，以及***对象的属性与表的字段的对应关系***
* 开发方式一：由Domain对象 ——> Mapping ——> DB
* 开发方式二：由DB开始，用工具生成mapping和Domain object（使用较多）
* 开发方式三：由映射文件开始

## Hibernate快速入门（第二种开发方式）★★★★★★

### github代码

https://github.com/rhapsody1290/Hibernate_Study

### 创建employee表

![](http://i.imgur.com/tdPdlVh.png)

### 开发domain对象

* 建议***domain对象*** 的名称就是对应表的首字母大写，同时
	1.	需要一个无参的构造函数(用于hibernate反射该对象)
	2.	应当有一个无业务逻辑的主键属性.
	3.	给每个属性提供 get/set 方法.
	4.	在domian对象中的属性，只有配置到了对象映射文件后，才会被hibernate管理.
	5.	属性一般是private范围

* 该pojo按照规范应当序列化，目的是可以唯一标该对象。同时可以在网络和文件上传输


	public class Employee implements Serializable{
	    private Integer id;
	    private String name;
	    private String email;
	    private  java.util.Date hiredate;
	
	    public String getEmail() {
	        return email;
	    }
	
	    public void setEmail(String email) {
	        this.email = email;
	    }
	
	    public Date getHiredate() {
	        return hiredate;
	    }
	
	    public void setHiredate(Date hiredate) {
	        this.hiredate = hiredate;
	    }
	
	    public Integer getId() {
	        return id;
	    }
	
	    public void setId(Integer id) {
	        this.id = id;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	}

### 对象关系映射文件

对象关系映射文件作用是用于指定domain对象和表的映射关系，该文件的取名有规范：***domain对象.hbm.xml***，一般我们放在和domain对象同一个文件夹下(包下)

	<?xml version='1.0' encoding='utf-8'?>
	<!DOCTYPE hibernate-mapping PUBLIC
	        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
	<hibernate-mapping package="cn.apeius.domain">
	
	    <class name="Employee" table="employee">
	        <!--id文件用于指定主键属性-->
	        <id name = "id" column="id" type="java.lang.Integer">
	            <generator class="increment"/>
	        </id>
	        <!--对其他属性配置-->
	        <property name="name" type="java.lang.String">
	            <column name="name" length="255" not-null="true"/>
	        </property>
	        <property name="email" type="java.lang.String">
	            <column name="email" length="255" not-null="false"/>
	        </property>
	        <property name="hiredate" type="java.util.Date">
	            <column name="hiredate" length="255" not-null="false"/>
	        </property>
	    </class>
	</hibernate-mapping>

***细节***

对象关系文件中，有些属性是可以不配，hibernate会采用默认机制，比如

	<class table="?" > table值不配，则以类的小写做表名
	<property type="?"> type不配置，则hibernate会根据类的属性类型，选择一个适当的类型


### 手动配置我们的hibernate.cfg.xml文件
该文件用于配置连接的数据库的类型、driver、用户名、密码、url等，同时管理对象关系映射文件。***该文件的名称，我们一般不修改***.

	<?xml version="1.0" encoding="utf-8"?>
	<!DOCTYPE hibernate-configuration PUBLIC
	        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
	<hibernate-configuration>
	    <session-factory>
	        <!--hibernate常用改的配置见：hibernate.properties-->
	        <!-- 配置dialect方言,明确告诉hibernate连接是哪种数据库 -->
	        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
	        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
	        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/hibernate</property>
	        <property name="hibernate.connection.username">root</property>
	        <property name="hibernate.connection.password">root</property>
	        <!--显示出对应的SQL语句-->
	        <property name="show_sql">true</property>
			<property name="hibernate.format_sql">true</property>
			<property name="hibernate.hbm2ddl.auto">update</property>
	        <mapping resource="Employee.hbm.xml"/>
	    </session-factory>
	</hibernate-configuration>

### 增加用户

	import cn.apeius.domain.Employee;
	import org.hibernate.Session;
	import org.hibernate.SessionFactory;
	import org.hibernate.Transaction;
	import org.hibernate.cfg.Configuration;
	
	/**
	 * Created by Asus on 2016/8/16.
	 */
	public class Main {
	    public static void main(String[] args){
	        //1、创建configuration，该对象用于读取hibernate.ctf.xml，并完成初始化
	        Configuration configuration = new Configuration().configure();
	        //2、创建sessionFactory，这是一个会话工厂，是个重量级的对象，应当保证连接一个数据库SessionFactory是单例
	        SessionFactory sessionFactory = configuration.buildSessionFactory();
	        //3、创建session，相当于jdbc connection
	        Session session = sessionFactory.openSession();
	        //4、在进行增加、删除、修改的时候使用事务提交
	        Transaction transaction = session.beginTransaction();
	        //添加一个雇员
	        Employee employee = new Employee();
	        employee.setName("qm1");
	        employee.setEmail("qm1@126.com");
	        employee.setHiredate(new java.util.Date());
	        //保存
	        session.save(employee);
	        //提交
	        transaction.commit();
	        session.close();
	    }
	}


### 修改用户（先查后改）

注意：SessionFactory是个重量级对象，应保证其实单例。在util包中封装了SessionFactory

	//获取一个会话
	Session session = MySessionFactory.getInstatnce().openSession();
	Transaction transaction = session.beginTransaction();
	//修改用户1、获得要修改的对象2、修改
	//load是通过主键属性，获取该对象实例
	Employee employee = (Employee) session.load(Employee.class,2);//产生select .. where id = 2
	employee.setName("钱明");//这句话会产生update语句
	transaction.commit();
	session.close();

### 删除用户

	//获取一个会话
	Session session = MySessionFactory.getInstatnce().openSession();
	Transaction transaction = session.beginTransaction();
	Employee employee = (Employee) session.load(Employee.class,2);
	session.delete(employee);
	transaction.commit();
	session.close();

## 模版（加入了异常回滚）★★★★★★

	public static void updateEmployee() {
        //获取一个会话
        //Session session = MySessionFactory.getInstatnce().openSession();
        Session session = HibernateUtil.getCurrentSession();
		Transaction transaction = null;
        try{
            transaction = session.beginTransaction();
            //do...
            //修改用户1、获得要修改的对象2、修改
            //load是通过主键属性，获取该对象实例
            Employee employee = (Employee) session.load(Employee.class,3);//产生select .. where id = 2
            employee.setName("钱明");//这句话会产生update语句
            //出现异常
            //int i = 9/0;
            transaction.commit();
        }catch (Exception e){
            if(transaction != null){
                transaction.rollback();
            }
            throw new RuntimeException(e.getMessage());
        }finally {
            //关闭session
            if(session != null && session.isOpen()){
                session.close();
            }
        }
    }

## SessionFactory单例

* SessionFactory是个重量级对象，在开发中保证只有一个SessionFactory
* 一个数据库对应一个SessionFactory对象


	//单例模式
	public class MySessionFactory {
	    private MySessionFactory(){}
	    private static class HoldClass{
	        private static final SessionFactory instance = new Configuration().configure().buildSessionFactory();
	    }
	    public static SessionFactory getInstatnce(){
	        return HoldClass.instance;
	    }
	}

## Maven下载各数据库JDBC及Hibernate配置文件

各数据库连接配置与maven依赖安装  
http://blog.163.com/luowei505050@126/blog/static/119907206201210223827126/

## Hibernate切换数据库★★★★★

* 使用Hibernate自动完成domain ——> 映射文件 ——> 表的工作
* 重新配置Hibernate数据库，以sqlserver2000为例
* <font color='red'>增加属性hibernate.hbm2ddl.auto</font>
	* create : 当我们的应用程序加载hibernate.cfg.xml [ new Configuration().config(); ]就会根据映射文件，创建出数据库, 每次都会重新创建， 原来表中的数据就没有!!!
	* update: 如果数据库中没有该表，则创建，如果有表，则看有没有变化，如果有变化，则更新.
	* create-drop: 在显示关闭 sessionFactory时，将drop掉数据库的schema
	* validate: 相当于每次插入数据之前都会验证数据库中的表结构和hbm文件的结构是否一致
	* 在开发测试中，我们配置哪个都可以测试，但是如果项目发布后，最好自己配置一次，让对应的数据库生成，完后取消配置， 
* 修改主键生成策略


	<?xml version="1.0" encoding="utf-8"?>
	<!DOCTYPE hibernate-configuration PUBLIC
	        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
	<hibernate-configuration>
	    <session-factory>
	        <!--hibernate常用改的配置见：hibernate.properties-->
	        <!-- 配置dialect方言,明确告诉hibernate连接是哪种数据库 -->
	        <property name="hibernate.dialect">org.hibernate.dialect.SQLServerDialect</property>
	        <property name="hibernate.connection.driver_class">net.sourceforge.jtds.jdbc.Driver</property>
	        <property name="hibernate.connection.url">jdbc:jtds:sqlserver://localhost:1433/hibernate</property>
	        <property name="hibernate.connection.username">sa</property>
	        <property name="hibernate.connection.password">sa</property>
	        <!--显示出对应的SQL语句-->
	        <property name="show_sql">true</property>
	        <!--让hibernate自动创建表 create:如果没有这张表则创建-->
	        <property name="hibernate.hbm2ddl.auto">update</property>
	        <mapping resource="Employee.hbm.xml"/>
	    </session-factory>
	</hibernate-configuration>

## Hibernate的核心类和接口★★★★★★★★★★

![](http://i.imgur.com/Xi1skxd.png)

### Configuration 类

* 读取hibernate.cfg.xml
* 加载hibernate的驱动、url、用户..
* 管理对象关系映射文件 `<mapping resource="">`
* 管理hibernate配置信息

### SessionFactory （会话工厂）

* 可以缓存sql语句和数据(称为session级缓存)!!
* 是一个重量级的类，因此我们需要保证一个数据库，有一个SessionFactory
* 如果某个应用访问多个数据库，则要创建多个会话工厂，一个数据库一个会话工厂实例
![](http://i.imgur.com/3zKcZfm.png)
* 通过SessionFactory接口可以获得Session实例，有两种方式：

	* openSession() 是获取一个新的session
	* getCurrentSession () 获取和当前线程绑定的session,换言之，在同一个线程中，我们获取的session是同一session，这样更利于事务控制
	* 使用 getCurrentSession 需要配置 hibernate.cfg.xml中配置
	①如果使用本地事务（jdbc事务）
`<property name="hibernate.current_session_context_class">thread</property>`
	②如果使用全局事务（jta事务）
`<property name="hibernate.current_session_context_class">jta</property>`
	* 如何选择
	①如果需要在同一线程中，保证使用***同一个Session***，则使用getCurrentSession()
	②如果在一个线程中，需要使用***不同的Session***,则使用opentSession()
	* 通过 getCurrentSession() 获取的session在事务提交后，会自动关闭，通过openSession()获取的session则必须手动关闭
	* ***如果是通过getCurrentSession()获取sesssion,进行查询需要事务提交***.


　　***使用同一个Session案例***

![](http://i.imgur.com/pgTOK72.png)

　　在一个http请求中，需要同时进行操作、更新、删除等操作，三个操作在不同service中进行，但要将三个操作在一个事务中进行控制。三个service分别会调用hibernate，使用getCurrentSession，可以将三个操作在一个事务中进行（http请求只要不结束，就看成一个线程）

　　***本地事务与全局事务***

![](http://i.imgur.com/1xjDVPD.png)

　　本地事务：增对一个数据库的事务  
　　全局事务：跨数据库的事务（JTA）。以银行转账为例，需要在农行账户中增加10元，在工行账户中减去10元，涉及到多个数据库中的事务

### session接口的理解

* Session(会话)接口的理解：SessionFactory常驻内存，每次连接数据库会化工厂建立与数据库的session，图中横线就是一个session。

* 如何查看session是否关闭：查看数据库连接端口是否关闭，如mysql 3306端口是否关闭


![](http://i.imgur.com/CrHatM0.png)

![](http://i.imgur.com/cvdhJFx.png)

session接口它的主要功能和作用是:

* Session一个实例代表与数据库的一次操作(当然一次操作可以是crud组合)
* Session实例通过SessionFactory获取，用完需要关闭
* ***Session是线程不同步的(不安全),因此要保证在同一线程中使用,可以用 getCurrentSessiong()***
* Session可以看做是持久化管理器,它是与持久化操作相关的接口

### Session查询详解★★★★★★
	

* 如果查询不到数据，get 会返回 ***null***,但是不会报错, load 如果查询不到数据，则***报错ObjectNotFoundException***
<pre>
Employee employee1 = (Employee) session.load(Employee.class,10);
System.out.println(employee1);//ObjectNotFoundException
Employee employee2 = (Employee) session.get(Employee.class,10);
System.out.println(employee2);//null
</pre>

* 使用get去查询数据，先到缓存（session缓存、二级缓存）中去查，如果没有就到DB中取查，即立即向db发出查询请求(select ...),
* 如果你使用的是load查询数据，先到缓存(session缓存、二级缓存)中查询，如果没有则返回一个代理对象（不马上到DB中去查）。等后面使用这个代理对象时才到DB中查询，如果后面没有使用查询结果，它不会真的向数据库发select，这个现象我们称为懒加载(lazy)***【load是懒加载，返回代理对象，使用时候才去数据库查询】***
<pre>
Employee employee1 = (Employee) session.load(Employee.class,1);
System.out.println(employee1);//如果这句话注释，不会向数据库发送sql语句，等到使用给代理对象时才到DB中查询
</pre>
* 通过修改配置文件，我们可以取消懒加载
`<class  name="Employee" lazy="false" table="employee">`
* ***如何选择使用哪个：如果你确定DB中有这个对象就用load(),不确定就用get()（这样效率高）***

### Hibernate缓存原理★★★★★★

![](http://i.imgur.com/FIm3Wt8.png)

* 接受到一个load查询，先去session缓存中查询，如果没有再去二级缓存查找，如果还没有，就不查询了，返回一个代理对象proxy obj = load
* 如果使用这个代理对象后，就会去数据库查询，***并把这条记录放入二级缓存***
* 等到下次查询时，先查询session缓存，找不到，然后去二级缓存中取得对象，并返回结果。***同时把这条记录放入一级缓存***
* get查询也类似，先查询一级缓存，再查询二级缓存<br>
* Hibernate缓存机制可以减少对数据库的查询
<pre>
Employee employee1 = (Employee) session.load(Employee.class,1);//发送select语句，放缓存
System.out.println(employee1);
Employee employee2 = (Employee) session.get(Employee.class,1);//从缓存中取，没有select语句
System.out.println(employee2);
Employee employee3 = (Employee) session.get(Employee.class,100);//缓存中找不到，向数据库发送select语句
System.out.println(employee3);
</pre>

### HibernateUtil（线程局部模式：获取全新的Session或与线程绑定的Session）★★★★★★
注意：不再需要配置 hibernate.cfg.xml

	final public class HibernateUtil { //SqlHelper
	    private static SessionFactory sessionFactory = null;
	    //使用线程局部模式
	    private static ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();
	
	    static {
	        sessionFactory=new Configuration().configure().buildSessionFactory();
	    }
	
	    private HibernateUtil(){}
	
	    //获取全新的全新的sesession
	    public static Session openSession(){
	        return sessionFactory.openSession();
	    }
	    //获取和线程关联的session
	    public static Session getCurrentSession(){
	        Session session = threadLocal.get();
	        //判断是否得到
	        if(session == null){
	            session = sessionFactory.openSession();
	            //把session对象设置到 threadLocal,相当于该session已经和线程绑定
	            threadLocal.set(session);
	        }
	        return session;
	    }
	}

***测试***

	System.out.println(HibernateUtil.getCurrentSession().hashCode());
	System.out.println(HibernateUtil.getCurrentSession().hashCode());
	System.out.println(HibernateUtil.openSession().hashCode());
	System.out.println(HibernateUtil.openSession().hashCode());
	结果：
		725455968
		725455968
		1473790157
		914784201

***线程局部模式***

![](http://i.imgur.com/sAHw9by.png)

在一个线程中，某个时间点通过set将对象放入ThredLocal，又在某个点通过get方法取出对象。这个变量与线程绑定

### query接口★★★★★★

通过query接口我们可以完成更加复杂的查询任务
举例: 通过用户来查询数据.

	//获取query引用
	Query query = session.createQuery("from Employee where id = 1");
	//通过List方法获取结果，这个list会自动封装成domain对象
	List<Employee> list = query.list();
	for(Employee e : list){
	    System.out.println(e.getName() + " " + e.getHiredate());
	}

注意：Employee是Java对象而不是表名，id也是类的属性名

### Criteria接口（不常用）
Criteria接口也可用于面向对象方式的查询，关于它的具体用法我们这里先不做介绍,简单看几个案例.

	Criteria cri = session.createCriteria(Employee.class)
	        .setMaxResults(2)
	        .addOrder(Order.asc("id"));
	List<Employee> list = cri.list();
	for(Employee e : list){
	    System.out.println(e.getId() + " " + e.getName());
	}

## 工具生成domain对象和对象关系映射文件

* 使用IntelliJ IDEA开发SpringMVC网站（三）数据库配置★★★
http://blog.csdn.net/chenxiao_ji/article/details/50849365

* 在创建工程的时候勾选上hibernate支持。

![](http://i.imgur.com/tadYPC3.png)

* 在主界面右侧找到database，点击添加数据库

![](http://i.imgur.com/592uDna.png)

* 在新界面中添加数据库驱动和数据库链接信息

![](http://i.imgur.com/dpI2qNp.png)

* 保存后在主面板左侧有persistence，在hibernate图标上点击右键-Generate Persistence Mapping-By Database Scheme

![](http://i.imgur.com/duTZSlM.png)

* 选好数据库，选好包的位置，在下面勾上要生成的表对应的pojo，并且勾上为每一个pojo生成XML即可

![](http://i.imgur.com/PXz7CeO.png)

## IDEA一对多关系设置

* 模拟一个学生选课系统 ，创建三张表：student、studCourse、course
* 数据库备份
https://github.com/rhapsody1290/Hibernate_Study/blob/master/doc/hibernate.sql
* student与studCourse的关系为一对多，即一个学生可以选择多门课程。Student类中有一个studCourse的集合属性，studCourse中有一个Student的成员变量
* 设置student与studCourse之间的关系

	* 原则：在有外键的表上设置表之间的关系，本例中在studCourse上设置关系	
	* 在上图中选中studCourse表，studCourse通过外键sid和cid关联student表和couse表。注意外键不打勾
	* 点击左上角绿色的加号或右键选择`add relationship`
	* studCourse类中有一个类型为Course的course变量，Course类中有一个名为studCourses的集合，两个表通过cid关联起来
![](http://i.imgur.com/6hDMVCk.png)

* 同理设置studCourse与student关系
* 设置完成后观察属性是否正确
![](http://i.imgur.com/5tEWDh7.png)
* 生成的domain对象及hdm文件见github
https://github.com/rhapsody1290/Hibernate_Study/tree/master/src/main/java/cn/apeius/domain
https://github.com/rhapsody1290/Hibernate_Study/tree/master/src/main/resources

## HQL语句详解★★★★★★

### 关系模型和对象模型映射

![](http://i.imgur.com/6cu2JfO.png)

* studCourse表有两个外键，一个是与student表关联的sid，另一个是与course关联的cid
* 一个学生可以选择多门课程，一个课程可以被多名学生选择，两个表是多对多的关系
* 在对象模型中，一个学生可以选择多个课程，所以它有一个set集合的成员变量，存放studcourse
* 在对象模型中，一个课程可以可以被多名学生选择，所以它有一个set集合的成员变量，存放studcourses

### 取出部分属性

* 在讲解jdbc中，要查询什么字段就查询什么字段，不要`select * from..`
* 但是在Hibernate中，建议把整个对象的属性都查询

***查询整个对象的属性***
list方法返回的是整个Student对象

	List<Student> list = session.createQuery("from Student").list();
	for(Student s : list){
	    System.out.println(s.getSname() + " "  + s.getSid());
	}

***查询部分属性***

因为只是查询部分属性，hibernate没有把返回的结果封装成Student对象，而只是Object数组

	List list = session.createQuery("select sname,sid from Student").list();
	for(int i = 0; i < list.size(); i++){
	    Object[] obj = (Object[]) list.get(i);
	    System.out.println(obj[0] + " " + obj[1]);
	}

或（推荐）

	List<Object[]> list = session.createQuery("select sname,sid from Student").list();
	for(Object[] obj : list){
	    System.out.println(obj[0] + " " + obj[1]);
	}
原理：创建list对象，对于每条记录创建一个对象数组，并加入list中

![](http://i.imgur.com/jO4KEIX.png)

***如果我们返回的是一列数据***

	//这时我们的取法是直接取出list->object 而不是 list->Object[]
	List<Object> list = session.createQuery("select sname from Student").list();
    for(Object obj : list){
        System.out.println(obj);
    }

### 对象模型关联查询

***查询学生就能查出于课程关联的全部信息***

	List<Student> list = session.createQuery("from Student").list();
	for(Student s : list){
	    System.out.print(s.getSname());
	    if(s.getStudcourses().size() == 0){
	        System.out.println("没有选课");
	    }else{
	        System.out.print("选了");
	        for(Studcourse studcourse : s.getStudcourses()){
	            System.out.print(studcourse.getCourse().getCname() + " ");
	        }
	        System.out.println();
	    }
	}

***请显示所有选择了21号课程的学生信息***

	String sql = "select student.sname,student.sage from Studcourse where course.cid = 21";
	List<Object[]> list = HibernateUtil.executeQuery(sql,null);
	for(Object[]  s : list){
	    System.out.println(s[0] + " " + s[1]);
	}

### uniqueResult方法(只有一个对象)

当`session.createQuery("from xxx where cardid='xxx'").uniqueResult();`返回的结果***只有一个对象***时，可以使用`uniqueResult()`得到该对象，效率高。但是，如果结果是多条，使用该方法就会抛出异常。

	Student student = (Student) session.createQuery("from Student where id = 20050003").uniqueResult();
	if(student != null) System.out.println(student.getSname()); else System.out.println("记录不存在");
        
### 模糊查询

	String sql = "select sname,sage from Student where sname like '林%'";
	List<Object[]> list = HibernateUtil.executeQuery(sql,null);
	for(Object[] s : list){
	    System.out.println(s[0] + " " + s[1]);
	}

### distinct的用法（过滤重复的记录）
比如，显示所有学生的性别和年龄

	List list=session.createQuery("select distinct sage,ssex from Student").list();
	for(int i=0;i<list.size();i++){
		Object []  objs=(Object[]) list.get(i);
		System.out.println(objs[0].toString()+" "+objs[1].toString());
	}

### between and

年龄在20岁到22岁的学生

	List list=session.createQuery("select distinct sage,ssex,sname from Student where sage between 20 and 22").list();
	for(int i=0;i<list.size();i++){
		Object []  objs=(Object[]) list.get(i);
		System.out.println(objs[0].toString()+" "+objs[1].toString()+objs[2].toString());
	}

### in /not in

查询计算机系和外语系的学生信息
				
	List<Student> list=session.createQuery("from Student where sdept in ('计算机系','外语系')").list();
	for(Student s:list){
		System.out.println(s.getSname()+" "+s.getSaddress()+" "+s.getSdept());
	}

### group by使用

显示各个系的学生的平均年龄

	List<Object[]> list=session.createQuery("select avg(sage),sdept from  Student group by sdept").list();
	for(Object[] obj:list){
		System.out.println(obj[0].toString()+" "+obj[1].toString());
	}

### having的使用
对分组查询后的结果，进行筛选

***1.请显示人数大于3的系名称***

	//a. 查询各个系分别有多少学生 b.筛选	
	List<Object[]> list=session.createQuery("select count(*) as c1,sdept from  Student group by sdept having count(*)>3").list();
	//取出1. for 增强
	for(Object[] obj:list){
		System.out.println(obj[0].toString()+" "+obj[1].toString());
	}

***2.查询女生少于200人的系***

	List<Object[]> list = session.createQuery("select count(*),sdept from Student  where ssex = 'F'group by sdept having count(*) < 200").list();
	for(Object[] obj : list){
	    System.out.println(obj[0] + " " + obj[1]);
	}

### 聚集函数的使用 count(),avg(),max(),min(),sum()


***1.查询计算机系共多少人***

	Long count = (Long) session.createQuery("select count(*) from Student where sdept='计算机系'").uniqueResult();
	System.out.println(count);

***2.查询选修11号课程的最高分和最低分***

	List<Object[]> list=session.
	createQuery("select 11,max(grade),min(grade) from Studcourse where course.cid=11").list();
	for(Object[] obj:list){
		System.out.println(obj[0].toString()+" max="+obj[1].toString()+" min="+obj[2].toString());
		}

***3.计算各个科目不及格的学生数量***

	List<Object[]> list=session.createQuery("select count(*),course.cname from Studcourse where grade < 60 group by course.cname").list();
	for(Object[] obj:list){
	    System.out.println(obj[0].toString()+ " " + obj[1]);
	}

***4.显示各科考试不及格学生的名字，科目和分数***

    List<Object[]> list=session.createQuery("select student.sname,course.cname,grade from Studcourse where grade < 60").list();
    for(Object[] obj:list){
        System.out.println(obj[0].toString()+ " " + obj[1] + " " + obj[2]);
    }

### 分页
分页原理见Java基础常用

***函数使用***

  List q=session.createQuery(hql).setFirstResultl(从第几条取//从0开始计算).setMaxResult(取出几条).list();

***据用户输入的pageNow 和pageSize显示对象***

	Session session = HibernateUtil.getCurrentSession();
    Transaction transaction = null;

    int pageNow  = 1;
    int pageSize = 3;
    int pageCount = 0;
    int rowCount = 0;
    try{
        transaction = session.beginTransaction();
        //do...
        rowCount = Integer.parseInt(session.createQuery("select count(*) from Student").uniqueResult().toString());
        pageCount = (rowCount -1)/pageSize + 1;
        //遍历
        for(int i = 0; i <= pageCount; i++){
            System.out.println("***********************************");
            List<Student> list = session.createQuery("from Student").setFirstResult(pageSize*i).setMaxResults(pageSize).list();
            for(Student s : list){
                System.out.println(s.getSname());
            }
        }
        transaction.commit();
    }catch (Exception e){
        e.printStackTrace();
        if (transaction != null){
            transaction.rollback();
        }
        throw new RuntimeException(e.getMessage());
    }finally {
        if(session != null && session.isOpen()){
            session.close();
        }
    }

### SQL注入

	select * from student where sage=2412 or 1=1

上面我们使得WHERE恒真，所以该查询中WHERE已经不起作用了

### 参数绑定

***使用参数绑定的好处：***
1.	可读性提高
2.	效果高 
3.	防止sql注入漏洞

***面试题: 如果不使用参数绑定，怎样防止登录时， sql注入?***

　　思路: 1. 通过用户名，查询出该用户名在数据库中对应的密码，然后再与用户输入的密码比较，如果相等，则用户和法，否则，非法.

***参数绑定有两种形式***

	Query q=session.createQuery(from Student where sdept=:dept and sage>:age)

如果我们的参数是:冒号形式给出的，则我们的参数绑定应当这样:

	List<Student> list=session.createQuery("from Student where sdept=:a1 and sage>:sage").setString("a1", "计算机系").setString("sage", "2").list();

还有一种形式:

	Query q=session.createQuery(from Student where sdept=? and sage>?)

如果我们的参数是以 ? 形式给出的则，参数绑定应当:

	List<Student> list=session.createQuery("from Student where sdept=? and sage>?").setString(0, "计算机系").setString(1, "2").list();

参数的绑定，可以分开写：

	Query query=session.createQuery("from Student where sdept=? and sage>?");		
	query.setString(0, "计算机系");
	query.setString(1, "2");
	List <Student> list=query.list();
	for(int i=0;i<list.size();i++){
		Student s= list.get(i);
		System.out.println(s.getSname()+" "+s.getSage());
	}

把HibernateUtil升级了

### 多表查询★★★

在实际项目中，我们不可能只对一张表进行查询，通常有多张表联合查询

#### hibernate对象之间关系

1.	one – to – one : 身份证<--->人 
2.	one – to – many  部门 <---> 员工
3.	many-to-one   员工<--->部门
4.	many-to-many  学生<--->老师 

#### 多对多关系转换成两个一对多

![](http://i.imgur.com/d8X9T9W.png)

* 一个学生可以选择多门课程，一门课程可以被多名学生选择，在实际开发中应将其转成两个一对多或多两个多对一。这样程序好控制，同时不会有冗余
* 对象配置文件可以体现出，Student.hbm.xml和Course.xml中是one-to-many，而Studcourse是many-to-one

#### 举个例子

请显示林青霞 选择的所有课程名，和成绩
	
	String sql = "select course.cname,grade from Studcourse where student.sname = '林青霞'";
	List<Object[]> list = HibernateUtil.executeQuery(sql,null);
	for(Object[] s : list){
	    System.out.println(s[0] + " " + s[1]);
	}

## Criteria—略讲

	//查询年龄大于10岁的学生
	//获取一个会话
	Session session = HibernateUtil.getCurrentSession();
	Transaction transaction = null;
	try {
	    transaction = session.beginTransaction();
	    Criteria cri = session.createCriteria(Student.class);
	    //增加检索条件
	    cri.add(Restrictions.gt("sage",10));
	    List<Student> list = cri.list();
	    for(Student s : list){
	        System.out.println(s.getSname());
	    }
	    transaction.commit();
	} catch (Exception e) {
	    if (transaction != null) {
	        transaction.rollback();
	    }
	    throw new RuntimeException(e.getMessage());
	} finally {
	    //关闭session
	    if (session != null && session.isOpen()) {
	        session.close();
	    }
	}

## hibernate对象的三种状态

### 如果判断一个对象处于怎样的状态？

主要的依据是: 1. 看该对象是否处于session管理下 2. 看在数据库中有没有对应的记录

* 瞬时态: <font color='blue'>没有session管理</font>，同时数据库没有对应记录
* 持久态: <font color='red'>有session管理</font>，同时在数据库中有记录。相关联的session没有关闭，事务没有提交，持久对象状态发生改变，在事务提交时会影响到数据库，即hibernate能检测到变化
* 脱管态/游离态： <font color='blue'>没有session管理</font>，但是在数据库中有记录。脱管对象状态发生改变，hibernate不能检测到

### 对象三种状态

	//对象三种状态
	Course c1 = new Course();//没有在session管理下，数据库没记录，c1就是瞬时态
	c1.setCid(100);
	c1.setCcredit(3);
	c1.setCname("php");
	
	Session session = null;
	Transaction tx  = null;
	try{
	    session = HibernateUtil.getCurrentSession();
	    tx = session.beginTransaction();
	    session.save(c1);//c1这时处于session管理下，同时c1对象被保存到数据库，c1处于持久态
	    tx.commit();
	    session.close();
	    //c1没有处于session管理下，单被保存到数据库中，c1就是脱管态（游离态）
	    System.out.println(c1.getCname());
	}catch (Exception e){
	    e.printStackTrace();
	}

![](http://i.imgur.com/Vdl8u49.png)

### 持久态中改变属性，反应到数据库中
<pre>
//对象三种状态
Course c1 = new Course();//c1就是瞬时态
c1.setCid(100);
c1.setCcredit(3);
<font color="red">c1.setCname("php1");</font>

Session session = null;
Transaction tx  = null;
try{
    session = HibernateUtil.getCurrentSession();
    tx = session.beginTransaction();
    session.save(c1);//c1这时处于session管理下，同时c1对象被保存到数据库，c1处于持久态
    <font color='red'>c1.setCname("php2");</font>//处于持久态，c1的改变有效，加入的数据php2
    tx.commit();
    session.close();
    //这时c1被保存到数据库中，同时没有处于session管理下，c1就是脱管态（游离态）
    <font color='red'>c1.setCname("php3");</font>
    System.out.println(c1.getCname());//<font color='blue'>结果为php3，但数据库中时php2</font>
}catch (Exception e){
    e.printStackTrace();
}
</pre>

### 持久态中删除记录

	Course c1 = new Course();//c1就是瞬时态
	c1.setCid(100);
	c1.setCcredit(3);
	c1.setCname("php1");
	Session session = null;
	Transaction tx  = null;
	try{
	    session = HibernateUtil.getCurrentSession();
	    tx = session.beginTransaction();
	    session.save(c1);//c1这时处于session管理下，同时c1对象被保存到数据库，c1处于持久态
	    c1.setCname("php2");//处于持久态，c1的改变有效，加入的数据php2
	    session.delete(c1);//删除数据库记录，c1处于瞬时态
	    tx.commit();
	    session.close();
	    //c1对应的记录被删除，同时没有处于session管理下，c1处于瞬时态
	    c1.setCname("php3");
	    System.out.println(c1.getCname());
	}catch (Exception e){
	    e.printStackTrace();
	}

### 对象状态 - 完整版

![](http://i.imgur.com/uXvnw1s.png)

## Hibernate关系映射★★★★★★★

### 多对一（内含懒加载问题）★★★★★★★

#### 多对一案例

![](http://i.imgur.com/DouTjzj.png)

采用开发方式一：从Domain和对象关系映射文件开始写，自动创建对应表

***Department.java***

	package cn.apeius.domain;
	
	public class Department implements java.io.Serializable {
	
		private Integer id;
		private String name;
		public Integer getId() {
			return id;
		}
		public void setId(Integer id) {
			this.id = id;
		}
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
	}

***Department.hbm.xml***

<pre>
&lt;?xml version="1.0" encoding="utf-8"?>
&lt;!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
&lt;hibernate-mapping package="cn.apeius.domain">
	&lt;class name="Department" lazy="false" table='department'>
		&lt;!-- 配置主键属性 -->
		&lt;id name="id" column='id' type="java.lang.Integer">
			&lt;!-- 生成策略 -->
			&lt;generator class="increment"/>
		&lt;/id>
		&lt;property name="name" type="java.lang.String">
			&lt;column name="name" length="255" not-null="true"/>
		&lt;/property>
	&lt;/class>
&lt;/hibernate-mapping>
</pre>

***Intern.java***

<pre>
package cn.apeius.domain;

public class Intern implements java.io.Serializable{

	private Integer id;
	private String name;
	<font color='red'>private Department dept;</font>
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public Department getDept() {
		return dept;
	}
	public void setDept(Department dept) {
		this.dept = dept;
	}
}
</pre>

***Intern.hbm.xml***

<pre>
&lt;?xml version="1.0" encoding="utf-8"?>
&lt;!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
&lt;hibernate-mapping package="cn.apeius.domain">
	&lt;class name="cn.apeius.domain.Intern" table='intern'>
		&lt;id name="id" column='id' type="java.lang.Integer">
			&lt;generator class="increment"/>
		&lt;/id>
		&lt;property name="name" type="java.lang.String">
			&lt;column name="name" length="255"/>
		&lt;/property>
		&lt;!--对于private Department dept;就不能使用property-->
		&lt;!--column="dept_id" 表示将来自动生成的表的外键名-->
		&lt;!--class可选，默认是通过反射得到属性类型-->
		<font color='red'>&lt;many-to-one name="dept" class = 'Department' column="dept_id"/></font>
	&lt;/class>
&lt;/hibernate-mapping>	
</pre>

***hibernate.cfg.xml***

	<property name="hibernate.hbm2ddl.auto">update</property>

***Main***

	//创建实习生
    Intern intern = new Intern();
    intern.setName("宋江");
    //创建部门
    Department department = new Department();
    department.setName("财务部");
    //实习生分配部门
    intern.setDept(department);
    //保存
    session.save(department);
    session.save(intern);

***结论一***

* 应当先保存部门，再保存实习生
* 保存部门后，产生部门id；当保存实习生时再将部门id存入实习生表中
* 若先保存实习生，此时还不知道部门id，dept_id的值为null；当保存部门并生成部门id时，在更新实习生表；这样比方式一效率低

#### 懒加载

当我们查询一个对象的时候，在默认情况下,返回的只是该对象的***普通属性***，当用户去***使用对象属性*** 时，才会向***数据库*** 发出再一次的查询.这种现象我们称为lazy现象.

	Intern intern = (Intern) session.get(Intern.class,3);
	//System.out.println(intern.getName() + " " + intern.getDept().getName());//可以读出部门名称
	transaction.commit();
	session.close();
	System.out.println(intern.getName() + " " + intern.getDept().getName());//session关闭，不可以读出部门名称

* 在session未关闭时，可以通过学生名得到部门名
* session关闭后，由于懒加载机制，会报错，不能得到部门名

#### 解决懒加载

* 显示初始化Hibernate.initized(代理对象)

<pre>
Intern intern = (Intern) session.get(Intern.class,3);
<font color='red'>Hibernate.initialize(intern.getDept());</font>
transaction.commit();
session.close();//关闭session，若没有显示初始化代理对象，则会报错
System.out.println(intern.getName() + " " + intern.getDept().getName());//session关闭，不可以读出部门名称
</pre>

* 在department映射文件中加入lazy='false'


		<class name="Department" lazy="false">

* 通过过滤器(web项目) openSessionInView


	many-to-one的many这方，如果你配置了 <class name="Student" lazy="false">
	那么hibernate就会在 查询学生 many 方时，把它相互关联的对象也查询,这里我们可以看出，对select语句查询影响不大,

	one-to-many 的one 的这方，如果你配置 <set name="stus" cascade="save-update" lazy="false">
	当你去查询一个部门的时候，该部门关联的学生全部返回，不管你使用否!!!
	如果设置lazy="true"，在session关闭后如果需要查询部门所在的学生就会报错
	
***矛盾: 如何让我们在需要使用的时候才去查询？即如何让我们的session范围更大？***

* 一个http请求所在一个session，缺点是session关闭会延时
![](http://i.imgur.com/wI2ANRU.png)
* 过滤器配合实现


	public class MyFilter1 extends HttpServlet implements Filter {
	
		public void doFilter(ServletRequest arg0, ServletResponse arg1,
				FilterChain arg2) throws IOException, ServletException {
			// TODO Auto-generated method stub
			Session s=null;
			Transaction tx=null;
			try {
				s =HibernateUtil.getCurrentSession();
				tx=s.beginTransaction();
				arg2.doFilter(arg0, arg1);
				//代码运行到这，整个请求结束
				tx.commit();
				
			} catch (Exception e) {
				if(tx!=null){
					tx.rollback();
				}
				throw new RuntimeException(e.getMessage());
			}finally{		
				HibernateUtil.closeCurrentSession();
			}
		}
	}

* HibernateUtil加入关闭session方法


	public static void closeCurrentSession() {
	
	    Session s = getCurrentSession();
	
	    if (s != null && s.isOpen()) {
	        s.close();
	        threadLocal.set(null);
	    }
	}

* 利用Spring可以更好得解决

### 一对多（内含级联操作）

需求：通过一个部门号1，来获取该部门的所有学生?

![](http://i.imgur.com/wRgVmDq.png)

#### 案例

***Department.java***
<pre>
package cn.apeius.domain;

import java.util.Set;

public class Department implements java.io.Serializable {

	private Integer id;
	private String name;
	//配置一个set集合，对应多个学生
	<font color='red'>private Set&lt;Intern> interns;</font>
	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public Set<Intern> getInterns() {
		return interns;
	}

	public void setInterns(Set<Intern> interns) {
		this.interns = interns;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public static long getSerialVersionUID() {
		return serialVersionUID;
	}
	
}
</pre>

***Department.hbm.xml***

<pre>
&lt;?xml version="1.0" encoding="utf-8"?>
&lt;!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
&lt;hibernate-mapping package="cn.apeius.domain">
    &lt;class name="Department" lazy="false" table = 'department'>
        &lt;!-- 配置主键属性 -->
        &lt;id name="id" type="java.lang.Integer" column = 'id' >
            &lt;!-- 生成策略 -->
            &lt;generator class="increment"/>
        &lt;/id>
        &lt;property name="name" type="java.lang.String">
            &lt;column name="name" length="255" not-null="true"/>
        &lt;/property>
        <font color='red'>&lt;!--配置one-to-many的关系-->
        &lt;set name="interns" <font color='blue'>cascade="save-update"></font>
            &lt;!--指定intern类对应的外键-->
            &lt;key column="dept_id"/>
            &lt;!--集合中的类名-->
            &lt;one-to-many class="Intern"/>
        &lt;/set></font>
    &lt;/class>
&lt;/hibernate-mapping>
</pre>

***Intern.java***

<pre>
package cn.apeius.domain;
public class Intern implements java.io.Serializable{
	private Integer id;
	private String name;
	<font color='red'>private Department dept;</font>
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public Department getDept() {
		return dept;
	}
	public void setDept(Department dept) {
		this.dept = dept;
	}
}
</pre>

***Intern.hbm.xml***

<pre>
&lt;?xml version="1.0" encoding="utf-8"?>
&lt;!DOCTYPE hibernate-mapping PUBLIC "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
"http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
&lt;hibernate-mapping package="cn.apeius.domain">
	&lt;class name="cn.apeius.domain.Intern">
		&lt;id name="id" type="java.lang.Integer" column = 'id'>
			&lt;generator class="increment"/>
		&lt;/id>
		&lt;property name="name" type="java.lang.String">
			&lt;column name="name" length="255"/>
		&lt;/property>
		&lt;!--对于private Department dept;就不能使用property-->
		&lt;!--column="dept_id" 表示将来自动生成的表的外键名-->
		&lt;!--class可选，默认是通过反射得到属性类型-->
		<font color='red'>&lt;many-to-one name="dept" class='Department' column="dept_id"/></font>
	&lt;/class>
&lt;/hibernate-mapping>
</pre>

***查询部门的实习生***

	Department department = (Department) session.get(Department.class,1);
	//取出该部门的实习生
	Set<Intern> set = department.getInterns();
	for(Intern intern : set)
	    System.out.println(intern.getName());

***部门中添加实习生***

<pre>
//创建部门
Department department = new Department();
department.setName("业务部");
//创建实习生
Intern intern1 = new Intern();
intern1.setName("实习生1");
Intern intern2 = new Intern();
intern2.setName("实习生2");
//部门分配学生
Set<Intern> set = new HashSet<Intern>();
set.add(intern1);
set.add(intern2);
<font color='red'>//级联添加，在department.hdm.xml中添加cascade属性 &lt;set name="interns" cascade="save-update"></font>
department.setInterns(set);
//保存
session.save(department);
</pre>

#### 级联操作

所谓级联操作就是说，当你进行某个操作(添加/修改/删除...)，就由hibernate自动给你完成

***案例:如何配置级联操作，当删除某个部门的时候，我们自动删除其学生.***

***首先我们在 配置文件中修改:***

	<!-- 配置one-to-many关系	 cascade="delete" 当删除该部门的时候(主对象，则级联删除它的学生从对象) -->
	<set name="stus" cascade="delete">
	<!-- 指定Student类对应的外键 -->
		<key column="dept_id" />
		<one-to-many class="Student" />
	</set>

***java代码中操作:***

	//演示删除级联
	//获取到某个部分
	Department department=(Department) s.get(Department.class, 41);
	s.delete(department);

***演示save-update***

	配置文件:
	<set name="stus" cascade="save-update">
	<!-- 指定Student类对应的外键 -->
	<key column="dept_id" />
	<one-to-many class="Student" />
	</set>

***代码：***

	//添加学生
	Department department=new Department();
	department.setName("业务部门3");
	
	Student stu1=new Student();
	stu1.setName("顺平6");
	Student stu2=new Student();
	stu2.setName("小明6");
	
	Set<Student> students=new HashSet<Student>();
	students.add(stu1);
	students.add(stu2);
	department.setStus(students);
	
	s.save(department);

***说明: ***

	① 在集合属性和普通属性中都能使用cascade
	② 一般讲cascade配置在one-to-many(one的一方,比如Employee-Department),和one-to-one(主对象一方)

### 一对一

#### 基于主键的one-to-one

![](http://i.imgur.com/8pSzSoU.png)

测试代码如下

* 生成两张表，person表和idCard表。person中id为主键，idCard表中id既为主键，也为外键
* person指定id和name，IdCard的主键由person的id指定
* <font color='red'>IdCard设置外键可以多对一，但当外键同时也为主键时，可以保证一对一</font>


	Person p1 = new Person();
	p1.setId(100);
	p1.setName("成龙");
	IdCard idCard = new IdCard();
	idCard.setValidateDte(new Date());
	//表示idCard对象是属于p1这个对象
	idCard.setPerson(p1);
	
	session.save(p1);
	session.save(idCard);

#### 基于外键的one-to-one

![](http://i.imgur.com/Qwp0a37.png)

* 在关系模型中，idCard设置外键person_id，则person与idCard是一对多的关系。如何限制约束使得一个person对应一个idCard，只需在references增加约束unique
* IdCard关系映射文件，在many-to-one中增加约束unique='true'，外键名为person_id

测试代码

	Person p1 = new Person();
	p1.setId(10);
	p1.setName("成龙");
	
	IdCard idCard = new IdCard();
	idCard.setId(100);
	idCard.setValidateDte(new Date());
	//表示idCard对象是属于p1这个对象
	idCard.setPerson(p1);
	
	session.save(p1);
	session.save(idCard);

### 多对多

在操作和性能方面都不太理想，所以多对多的映射使用较少，实际使用中最好转换成一对多的对象模型

![](http://i.imgur.com/dobH4GE.png)

## Hibernate一级缓存

### 缓存原理图

![](http://i.imgur.com/6uIqrv1.png)

从上图看出: 当我们去查询对象的时候，首先到一级缓存去取数据，如果有，则不到数据库中取，如果没有则到数据库中取，同时在一级缓存中放入对象.

### 缓存的细节

①什么操作会向一级缓存放入数据

	save,update,saveOrUpdate,load,get,list,iterate,lock

save 案例:

	//添加一个学生
	Student student=new Student();
	student.setName("小东");	
	s.save(student);//放入一级缓存
	
	//我马上查询
	Student stu2=(Student) s.get(Student.class, student.getId()); //select
	System.out.println("你刚刚加入的学生名字是"+stu2.getName());

②什么操作会从一级缓存取数据

	get/load/list

* get/load会首先从一级缓存中取，如没有,再有不同的操作
* get会立即向数据库发请求，而load 会返回一个代理对象，直到用户真的去使用数据，才会向数据库发请求


list会不会从session缓存取数据？

	//查询45号学生
	Student stu=(Student) s.get(Student.class, 45);
	System.out.println("|||||||||||||||||||");
	String hql="from Student where id=45";
	Student stu2=(Student) s.createQuery(hql).uniqueResult();
	System.out.println(stu2.getName());

从上面的案例，我看出 query.list() query.uniueResut() ***不会从一级缓取数据*** ! 但是query.list 或者query.uniqueRestu() 会***向一级缓存放数据***的

③一级缓存不需要配置，就可以使用,它本身没有保护机制，所以我们程序员要考虑这个问题,我们可以同 evict 或者 clear来清除session缓存中对象. evict 是清除一个对象，clear是清除所有的sesion缓存对象

④session级缓存中对象的生命周期, 当session关闭后，就自动销毁

⑤我们自己用HashMap来模拟一个Session缓存，加深对缓存的深入
	
	import java.util.*；
	public class MyCache {
		//使用map来模拟缓存
		static Map<Integer,Student> maps=new HashMap<Integer,Student>();
	
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			getStudent(1);
			getStudent(1);
			getStudent(1);
			getStudent(1);
			getStudent(3);
			getStudent(3);
				
		}
		
		public static Student getStudent(Integer id){  //s.get()
						
			//先到缓存去
			if(maps.containsKey(id)){
				//在缓存有
				System.out.println("从缓存取出");
				return maps.get(id);
			}else{
				System.out.println("从数据库中取");
				//到数据库取
				Student stu=MyDB.getStudentFromDB(id);
				//放入缓存
				maps.put(id, stu);
				return stu;
			}	
			
		}
	
	}
	
	//我的数据库
	class MyDB{
		
		static List<Student> lists=new  ArrayList<Student>();
		
		//初始化数据库,假设有三个学生
		static{
			Student s1=new Student();
			s1.setId(1);
			s1.setName("aaa");
			Student s2=new Student();
			s2.setId(2);
			s2.setName("bbb");
			Student s3=new Student();
			s3.setId(3);
			s3.setName("ccc");
			lists.add(s1);
			lists.add(s2);
			lists.add(s3);
			
		}
		
		public static Student getStudentFromDB(Integer id){
			for(Student s: lists){
				if(s.getId().equals(id)){
					return s;
				}
			}
			return null;// 在数据库中没有.
		}
	}
	
	class Student{
		private Integer id;
		private String name;
		public Integer getId() {
			return id;
		}
		public void setId(Integer id) {
			this.id = id;
		}
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
	}

## Hibernate二级缓存

### 为什么需要二级缓存？

因为一级缓存有限(生命周期短),所以我们需要二级缓存(SessionFactory缓存)来弥补这个问题，SessionFactory关闭后缓存才消失

1.	需要配置
2.	二级缓存是交给第三方去处理,常见的Hashtable , OSCache , EHCache
3.	二级缓存的对象可能放在内存，也可能放在磁盘.
4.	二级缓存的原理

![](http://i.imgur.com/1MPLBrn.png)

### 快速入门案例

使用OsCache来演示二级缓存的使用

1、配置二级缓存

	<!-- 启动二级缓存 -->
	<property name="cache.use_second_level_cache">true</property>
	<!-- 指定使用哪种二级缓存 -->
	<property name="cache.provider_class">org.hibernate.cache.OSCacheProvider</property>
	<mapping resource="com/hsp/domain/Department.hbm.xml" />
	<mapping resource="com/hsp/domain/Student.hbm.xml" />
	<!-- 指定哪个domain启用二级缓存 
	特别说明二级缓存策略:
		1. read-only只读缓存
		2. read-write读写缓存
		3. nonstrict-read-write不严格读写缓存
		4. transcational事务缓存
	-->
	<class-cache class="com.hsp.domain.Student" usage="read-write"/>

2、把oscahe.properties文件放在 src目录下，这样你可以指定放入二级缓存的对象capacity 大小默认1000

3、使用

	//通过获取一个sesion,让hibernate框架运行(config->加载hibernate.cfg.xml)
	Session s=null;
	Transaction tx=null;	
	try {
		//我们使用基础模板来讲解.
		s=HibernateUtil.openSession();
		tx=s.beginTransaction();		
		//查询45号学生	
		Student stu1=(Student) s.get(Student.class, 45);//45->一级缓存		
		System.out.println(stu1.getName());
		tx.commit();		
	} catch (Exception e) {
		e.printStackTrace();
		if(tx!=null){
			tx.rollback();
		}
	}finally{		
		if(s!=null && s.isOpen()){
			s.close();
		}
	}
	
	System.out.println("*********************************");
	try {
		//我们使用基础模板来讲解.
		s=HibernateUtil.openSession();
		tx=s.beginTransaction();		
		//查询45号学生
		Student stu1=(Student) s.get(Student.class, 45);	
		System.out.println(stu1.getName());
		
		Student stu3=(Student) s.get(Student.class, 46);	
		System.out.println(stu3.getName());
			tx.commit();
		
	} catch (Exception e) {
		e.printStackTrace();
		if(tx!=null){
			tx.rollback();
		}
	}finally{
		
		if(s!=null && s.isOpen()){
			s.close();
		}
	}
	
	//完成一个统计，统计的信息在Sessfactory
	//SessionFactory对象.
	Statistics statistics= HibernateUtil.getSessionFactory().getStatistics();
	System.out.println(statistics);
	System.out.println("放入"+statistics.getSecondLevelCachePutCount());
	System.out.println("命中"+statistics.getSecondLevelCacheHitCount());
	System.out.println("错过"+statistics.getSecondLevelCacheMissCount());


4、在配置了二级缓存后，请大家要注意可以通过 Statistics,查看你的配置命中率高不高

## 主键增长策略 

① increment

	自增，每次增长1, 适用于所有数据库。但是不要使用在多进程、主键类型是数值型
	select max(id) from Student

② identity

	自增，每次增长1, 适用于支持identity的数据(mysql,sql server), 主键类型是数值

③ sequence

	依赖于底层数据库系统的序列，前提条件:需要数据库支持序列机制（如:oracle等）,而且OID必须为数值类型,比如long,int,short类型。

④ native

	会根据数据类型来选择，使用identity,sequence,hilo 
	select hibernate_sequence.nextval from dual
	主键类型是数值long , short ,int
	<id name="id" type="java.lang.Integer"> 
		<generator class="native"/>
	</id>

⑤ hilo

	hilo标识符生成器由Hibernate按照一种high/low算法生成标识符
	用法:
	<id name="id" type="java.lang.Integer" column="ID">
		<generator class="hilo">
			<param name="table">my_hi_value</param>
			<param name=”column”>next_value</param>
		</generator>
	</id>

⑥ uuid

	会根据uuid算法，生成128-bit的字串
	主键属性类型不能是数值型，而是字串型

⑦ assigned

	用户自己设置主键值，所以主键属性类型可以是数值，字串

⑧ 映射复合主键
⑨ foreign 

	在one-to-one的关系中，有另一张表的主键(Person)来决定自己主键/外键(IdCard)[既是主键也是外键]
	
### 一个简单选择原则
	
* 针对 mysql [主键是 int/long/short 建议使用increment/assigend，如果是字串 UUId/assigned]
* 针对 sql server [主键是 int/long/short 建议使用 identity/native/assinged，如果主键是字串，使用uuid/assigned ]
* one-to-one 又是基于主键的则使用foreign

## Hibernate最佳实践

### Hibernate不适合的场景

* 不适合OLAP(On-Line Analytical Processing ***联机分析处理*** )，以查询分析数据为主的系统；适合OLTP（on-line transaction processing ***联机事务处理*** ）

![](http://i.imgur.com/kqShTak.png)

* 对于些关系模型设计不合理的老系统，也不能发挥hibernate优势
* 数据量巨大，性能要求苛刻的系统，hibernate也很难达到要求, 批量操作数据的效率也不高 

### Hibernate最佳实践

* 对于数据量大，性能要求高系统，不太适用使用hiberante
* 主要用于事务操作比较多的项目(oa/某个行业软件[石油、税务、crm, 财务系统]
* OLAP->hibernate用的比较少   
* OLTP->hibernate

## Hibernate开发过程（总结）

***hibernate.cfg.xml***

	Hibernate快速入门 - hibernate.cfg.xml

***domain对象***

	Hibernate快速入门 - 开发domain对象

***关系映射文件***

	Hibernate快速入门 - 手动配置我们的hibernate.cfg.xml文件

***基础模版***

    //获取一个会话
    Session session = HibernateUtil.getCurrentSession();
	Transaction transaction = null;
    try{
        transaction = session.beginTransaction();
        //do...
        
        transaction.commit();
    }catch (Exception e){
		e.printStackTrace();
        if(transaction != null){
            transaction.rollback();
        }
        throw new RuntimeException(e.getMessage());
    }finally {
        //关闭session
        if(session != null && session.isOpen()){
            session.close();
        }
    }

***HibernateUtil（增删改查）***
https://github.com/rhapsody1290/Hibernate_Study/blob/master/src/main/java/cn/apeius/util/HibernateUtil.java

***查询***

	#查询全部
    String sql = "from Student";
    List<Student> list = HibernateUtil.executeQuery(sql, null);
    for(Student s : list)
        System.out.println(s.getSname());
	#条件查询
	String sql = "from Student where sdept = ? and sage > ?";
	String[] parameters = {"计算机系", "3"};
	List<Student> list = HibernateUtil.executeQuery(sql, parameters);
	for(Student s : list)
		System.out.println(s.getSname());
	#部分查询
	String sql = "select sname, saddress from Student where sdept = ? and sage > ?";
	String[] parameters = {"计算机系", "3"};
	List<Object[]> list = HibernateUtil.executeQuery(sql, parameters);
	for(Object[] s : list)
	    System.out.println(s[0] + " " + s[1]);

***分页***

	String sql = "from Student order by sage";
	List<Student> list = HibernateUtil.executeQueryByPage(sql, null,4,2);
	for(Student s : list)
	    System.out.println(s.getSname());

***添加***

	Course c = new Course();
	c.setCid(61);
	c.setCname("servlet");
	HibernateUtil.save(c);

***修改***

	String sql = "update Course set ccredit = 2 where cid = 61";
	HibernateUtil.executeUpdate(sql,null);

***删除***

	String sql = "delete from Course where cid = 61";
	HibernateUtil.executeUpdate(sql,null);


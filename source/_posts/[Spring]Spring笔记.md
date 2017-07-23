---
title: Spring笔记

date: 2016-07-015 08:51:00

categories:
- Spring

tags:
- JavaEE
- Spring

---
## Spring是什么

* struts是web框架 (jsp/action/actionfrom)
* hibernate是orm框架，处于持久层
* spring是***容器框架***，用于配置各个层的bean（action/service/domain/dao），并维护bean之间关系的框架 

面试回答：**Spring就是一个轻量级的控制反转（IoC）和面向切面（AOP）的容器框架。**

## spring中重要概念★★★★★

### bean

* **Spring管理的对象为bean**
* bean是java中的任何一种对象（javabean/service/action/数据源/dao），spring作用是配置各个层中的组件（bean），并维持组件（bean）之间的关系
* JavaBean必须拥有一个无参的构造器，通过get/set方法访问参数，同时支持持久化

### 控制反转

* ioc(inverse of control，控制反转)，控制反转就是把<font color='red'>创建对象（bean）和维护对象（bean）的关系的权利</font>从程序中转移到spring容器中（applicationContext.xml），只需配置一下就能完成
* 使用Spring，程序中几乎所有重要的组件的创建工作和维护组件之间的依赖关系都移交给Spring

### 依赖注入

* di(dependency injection，依赖注入)，***实际上di和ioc是同一概念***，spring设计者认为di更准确表示spring核心技术
* 依赖注入接管对象的创建工作，并将该对象的引用注入需要该对象的组件

<pre>
/*
<font color='red'>有两个组件A和B，A依赖于B，且A中的importantMethod方法调用了B的方法，
使用B前，类A必须先获得组件B的实例（具体类可以new一个B实例，但如果B是接口，使用B的一个实现类，会降低A的可重用性）
使用依赖注入，框架会接管对象B的创建工作，并将B对象的引用注入到A中，具体是类A中的setB方法会被框架调用，注入一个B的实例，
这样类A的importantMethod方法在使用B的userfulMethod方法前不再需要创建一个B的实例</font>
*/
public class A {
    private B b;

    public B getB() {
        return b;
    }

    public void setB(B b) {
        this.b = b;
    }

    public void importantMethod(){
        b.userfulMethod();
    }
}
</pre>

## Spring在程序中的位置

　　Spring层次图如下图所示

![](http://i.imgur.com/xPyY3mN.png)

* Login.jsp与用户交互，将数据传递给Action.java处理器，Action.java一般与表单ActionForm关联，验证成功跳转到ok.jsp。这一层是web层，Struts位于web层
* 验证过程中，会调用UserService.java，这是业务层。业务层会有一个domain对象，Users.java[或者叫javabean，pojo]
* 业务层下是DAO层，它是对数据的操作
* 下面就是数据持久层，hibernate就位于持久层，它是一个orm框架
* 最底层的就是数据库
* model层分为业务层（Service），DAO层和数据持久层，在开发过程中，可以根据实际情况进行选择组合，并不是必须把model层分得这么细
* Spring横跨web层、业务层、DAO层、持久层，可以配置各个层的组件（JavaBean），并且维护各个bean之间的关系
* 具体的来说，spring可以配置web层的action[解决actin单例问题]，业务层的domain/service/dao以及数据持久层配置数据源
* 在配置文件中，体现出Spring创建各种组件及维持组件之间的关系
<pre>
&lt;bean id = "bean1" class = "">配置bean
	&lt;property name = "" value = "">&lt;/property>
&lt;/bean>
&lt;bean id = "bean2" class = "">维护bean之间的关系(bean2依赖bean1)
	&lt;property name = "" ref="">&lt;/property>
&lt;/bean>
</pre>

## Spring快速入门

### github

https://github.com/rhapsody1290/Spring_Study

### 引入依赖

	<properties>
        <!-- spring版本号 -->
        <spring.version>4.0.2.RELEASE</spring.version>
        <!-- log4j日志文件管理包版本 -->
        <slf4j.version>1.7.7</slf4j.version>
        <log4j.version>1.2.16</log4j.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <!-- 表示开发的时候引入，发布的时候不会加载此包 -->
            <scope>test</scope>
        </dependency>
        <!-- 导入Mysql数据库链接jar包 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.13</version>
        </dependency>
        <!-- spring核心包 -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-oxm</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!--spring事务-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.6.2</version>
        </dependency>
        <!-- 格式化对象，方便输出日志 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.1.41</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
    </dependencies>

### 创建组件

创建组件A、B，其中A组件依赖B

类A

	public class A {
	    private B b;
	
	    public void importantMethod(){
	        System.out.println("A:importantMethod");
	        b.usefulMethod();
	    }
	    public B getB() {
	        return b;
	    }
	
	    public void setB(B b) {
	        this.b = b;
	    }
	}

类B 

	public class B {
	
	    public void usefulMethod() {
	        System.out.println("B:usefulMethod");
	    }
	}

### applicationContext.xml中配置bean

applicationContext.xml是spring的一个核心配置文件, [hibernate有核心文件hibernate.cfg.xml struts核心文件 struts-config.xml]

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	    
		<!-- 在容器文件中配置bean(service/dao/domain/action/数据源) -->
	    <!-- bean元素的作用是，当我们的spring框架加载时候，spring就会自动的创建一个bean对象,并放入内存-->
	    <bean id = "A" class="hello.A">
	        <!-- 这里就体现出注入的概念，将B对象的引用注入到A中的b属性-->
	        <property name="b" ref="B"/>
	    </bean>
	    <bean id="B" class="hello.B"/>
	
	</beans>

### 获得bean并调用方法

	public class testSpringAPI {

	    private ApplicationContext context;

	    @Before
	    public void setUp() throws Exception {
	        //1.得到spring 的applicationContext对象(容器对象)
	        context = new ClassPathXmlApplicationContext("applicationContext.xml");
	    }
	
	    @Test
	    public void testName() throws Exception {
	        //2、利用java反射机制获取bean对象
	        A a  = (A) context.getBean("A");
	        a.importantMethod();
	    }
	}

## Spring运行原理图

![](http://i.imgur.com/IYAnjQ8.png)

* 当ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");执行的时候，Spring容器对象将被创建，同时applicationContext.xml中配置的bean就会被创建
* UserService us = (UserService)ac.getBean('UserService');调用后取出bean，取出的bean对象是单例
* bean的存储结构类似HashMap/HashTable，分为id和bean对象。id对应配置文件中bean元素的id，对象在Spring容器对象创建时创建并存放。若有引用关系，则指向引用对象的id
* Spring框架扫描XML文件，利用Java反射机制，创建一个个bean对象
* 可以利用dom4j+java反射机制模拟Spring运行流程。扫描xml文件，检测到bean元素，利用Java反射机制创建对象，并设置属性值，存入HashMap
	<pre>
	userService = Class.forName("com.service.UserService");
	userService.setName("韩顺平");
	
	applicationContext = new HashMap();
	applicationContext.put("userService",userService);
	</pre>

## Spring接口编程

spring开发提倡接口编程,配合di技术可以层与层的解耦

举例说明:
现在我们体验一下spring的di配合接口编程的，完成一个字母大小写转换的案例:
思路:
1.	创建一个接口 ChangeLetter
<pre>
public interface ChangeLetter {
　　public String change();
}
</pre>
2.	两个类实现接口
<pre>
public class LowerLetter implements ChangeLetter {
			private String str;
			
			public String getStr() {
			    return str;
			}
			
			public void setStr(String str) {
			    this.str = str;
			}
			
			@Override
			public String change() {
			    return str.toLowerCase();
			}
}</pre><pre>
public class UpperLeter implements ChangeLetter {
		    private String str;
		
		    public String getStr() {
		        return str;
		    }
		
		    public void setStr(String str) {
		        this.str = str;
		    }
		    @Override
		    public String change() {
		        return str.toUpperCase();
		    }
}
</pre>
3.	把对象配置到spring容器中
<pre>
    &lt;bean id="changeLetter" class="cn.apeius.inter.UpperLeter">
        &lt;property name="str" value="abc">&lt;/property>
    &lt;/bean>
&lt;!--    &lt;bean id="changeLetter" class="cn.apeius.inter.LowerLetter">
        &lt;property name="str" value="ABC">&lt;/property>
    &lt;/bean>-->
</pre>
4.	使用
<pre>
ApplicationContext ac = new ClassPathXmlApplicationContext("beans.xml");
ChangeLetter us= (ChangeLetter) ac.getBean("changeLetter");
System.out.println(us.change());
</pre>

通过上面的案例，我们可以初步体会到di配合接口编程，的确可以减少层(web层) 和 业务层的耦合度.

## XML配置文件

* 配置文件的根元素通常为
	  <?xml version="1.0" encoding="UTF-8"?>
	  <beans xmlns="http://www.springframework.org/schema/beans"
	   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	  </beans>
* 如果需要更强的Spring配置能力，可以在***schemaLocation***属性中添加相应地schema
* ***配置文件可以是一份，也可以是多份***，ApplicationContext的实现类支持读取多份配置文件

      ApplicationContext context = new ClassPathXmlApplicationContext(new  String[]{"applicationContext.xml","applicationContext2.xml"});

## 三种获取ApplicationContext对象引用的方法

ApplicationContext对象代表一个***Spring控制反转容器***，org.springframework.context.ApplicationContext接口有多个实现，包括：

1、ClassPathXmlApplicationContext -> 通过类路径加载配置文件

	ApplicationContext context  = new ClassPathXmlApplicationContext("applicationContext.xml");

2、FileSystemXmlApplicationContext -> 通过文件路径加载配置文件

<pre>
ApplicationContext context = new FileSystemXmlApplicationContext("配置文件绝对路径");
</pre>

3、XmlWebApplicationContext 从web系统中加载

## 两个获取bean的方式★★★

### 从ApplicationContex应用上下文容器中获取bean
 
#### 基本模式创建一个bean实例★★★★★

Spring通过***默认无参的构造器***来创建一个bean实例

配置文件

	<bean id="B" class="hello.B"/>

获取bean

	//1.得到spring 的applicationContext对象(容器对象)
	ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
	//从容器中取出一个bean的实例
	B b = ac.getBean("B", B.class);
    b.usefulMethod();

#### 工厂模式创建一个bean实例

1、定义一个接口

	public interface Animal {
	    public void sayHello();
	}

2、接口的两个实现类

Cat

	public class Cat implements Animal {
	    public void sayHello() {
	        System.out.println("Cat");
	    }
	}

Dog 

	public class Dog implements Animal {	
	    public void sayHello() {
	        System.out.println("Dog");
	    }
	}

3、AnimalFactory工厂中包含了一个getAnimal的***静态方法***，该方法将根据传入的参数决定创建哪个对象。这是典型的静态工厂设计模式

	public class AnimalFactory {
	    public static Animal getAnimal(String type){
	        if("Cat".equals(type)){
	            return new Cat();
	        }else if("Dog".equals(type)){
	            return new Dog();
	        }
	        return null;
	    }
	}
	
4、Spring配置文件中作如下配置

	<bean id="cat" class="Factory.AnimalFactory" factory-method="getAnimal">
        <constructor-arg value="Cat"/>
    </bean>

    <bean id="dog" class="Factory.AnimalFactory" factory-method="getAnimal">
        <constructor-arg value="Dog"/>
    </bean>

* 使用静态工厂方法创建Bean实例时，class属性也必须指定，但此时***class属性并不是指定Bean实例的实现类，而是静态工厂类***
* 需要使用factory-method来指定静态工厂方法名，Spring将调用静态工厂方法来返回一个Bean实例，使用<constructor-arg />元素来为静态工厂方法指定参数
* 当使用静态工厂方法来创建Bean时，这个factory-method必须要是***静态的***

### 从bean工厂容器中获取bean
			
	//如果我们使用beanfactory去获取bean，当你创建Spring容器时bean不被实例化,只有当你去使用getBean某个bean时，才会实时的创建	
	BeanFactory factory = new XmlBeanFactory(
			new ClassPathResource("applicationContext.xml"));
	factory.getBean("A");

### 结论

1. 如果使用ApplicationContext ，则配置的bean如果是singlton不管你用不用，都被实例化.(好处就是可以预先加载,缺点就是耗内存)
2. 如果是 BeanFactory ,则当你获取beanfacotry时候，配置的bean不会被马上实例化，当你使用的时候，才被实例(好处节约内存,缺点就是速度)
3. **规定: 一般没有特殊要求，应当使用ApplicatioContext完成**(实际项目中90%都采用这种方式)</font>

## bean的生命周期
***为什么总是一个生命周期当做一个重点?***

　　例如我们经常需要知道Servlet的生命周期，初始化init和销毁destroy，还有讨论java对象生命周期
　　不懂servlet的生命周期，你一样可以做开发。但是只有把一个新的技术对应的对象的生命周期弄清楚时，你才能真正的驾驭他。你知道Servlet创建的时候会调用init，才会把初始化工作放在init中，你知道Servlet销毁时会调用destroy，才会把文件备份的工作放在destroy中

***bean的生命周期***

bean被载入到容器中时，它的生命周期就开始了

![](http://i.imgur.com/n6bHb14.png)

①	实例化★

* 当我们的程序加载`beans.xml`文件，把我们的bean实例化到内存
* 默认调用无参的构造方法
* 以上我们考虑的是`scope=singleton`，单例模式最复杂

②	调用set方法设置属性★
③	如果你实现了bean名字关注接口`BeanNameAware`则，可以通过`setBeanName`获取id号
④	如果你实现了bean工厂关注接口`BeanFactoryAware`,则可以获取`BeanFactory`
⑤	如果你实现了`ApplicationContextAware`接口，则可以获得应用程序上下文
 
	//该方法传递ApplicationContext
	public void setApplicationContext(ApplicationContext arg0)
			throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("setApplicationContext"+arg0);
	}

⑥	如果bean和一个后置处理器关联，则实例化一个bean时，会自动去调用before方法。体现AOP（面向切片编程），放置一些公共方法，例如过滤ip，给对象添加属性等★

***自定义一个类myBeanPostProcessor，实现BeanPostProcessor接口，重写before和after两个方法***

<pre>
# myBeanPostProcessor.java
public class myBeanPostProcessor <font color='red'>implements BeanPostProcessor</font> {
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
		<font color='red'>//对象o对实例化的bean对象，s为bean的id</font>
        System.out.println("postProcessBeforeInitialization");
        return o;
    }

    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        System.out.println("postProcessAfterInitialization");
        return o;
    }
}
</pre>

***xml文件中配置***

	<bean id = "myBeanPostProcessor" class="cn.apeius.beanlift.myBeanPostProcessor"></bean>

***运行***

	ApplicationContext ac = new ClassPathXmlApplicationContext("beans.xml");
    PersonService personService = (PersonService) ac.getBean("PersonService");
    personService.sayHi();

***结果：bean对象创建后会自动调用before和after方法。类似JavaWeb中的过滤器***

	构造函数被调用
	调用set方法
	postProcessBeforeInitialization
	postProcessAfterInitialization
	你好啊 钱钱

⑦	如果你实现InitializingBean接口，则会调用afterPropertiesSet
⑧	如果自己配置`<bean init-method="init" />`，则可以在bean定义自己的初始化方法init。也可以通过注解的方式@PostConstruct
⑨	如果bean和一个后置处理器关联,则会自动去调用Object postProcessAfterInitialization方法
⑩	使用我们的bean
⑪ 容器关闭
⑫ 可以通过实现`DisposableBean`接口来调用方法`destory`
⑬ 可以在`<bean destory-method="fun1"/>`调用定制的销毁方法。可以通过注解的方式@PreDestroy

***小结: 我们实际开发中往往，没有用的这么的过程,常见的是:1->2->6->10->9->11 ***

问题:通过BeanFactory来获取bean对象，bean的生命周期是否和 Applicationcontext 是一样吗?

	不是一样的，bean是工厂中创建的生命周期会简单一些:
	比起ApplicationContext创建bean，通过BeanFactory少了一下步骤
	⑤ 如果你实现了ApplicationContextAware接口，则可以获得应用程序上下文
	⑥ 如果bean和一个后置处理器关联，则实例化一个bean时，会自动去调用before方法。
	⑨ 如果bean和一个后置处理器关联,则会自动去调用Object postProcessAfterInitialization方法
![](http://i.imgur.com/1KHveOv.png)


## bean装配的细节

### 注入的写法★

基础数据类型注入

<pre>
一个标签
&lt;property name="name" <font color='red'>value="财务部"</font>/>
两个标签
&lt;property name="name">
	<font color='red'>&lt;value>财务部&lt;/value></font>
&lt;/property>
</pre>

对象注入

<pre>
# 方法一：利用property的ref属性，这是一种简写方式
&lt;bean id="Department"  class="cn.apeius.collections.Department">
	<font color='red'>&lt;property name="emp" ref="Emp"/></font>
&lt;/bean>
&lt;bean id="Emp" class="cn.apeius.collections.Emp">
	&lt;property name="id" value="1"/>
	&lt;property name="name" value="qm"/>
&lt;/bean>

# 方法二：ref标签
&lt;bean id="Department"  class="cn.apeius.collections.Department">
	&lt;property name="emp">
		<font color='red'>&lt;ref bean="Emp"/></font>
	&lt;/property>
&lt;/bean>
&lt;bean id="Emp" class="cn.apeius.collections.Emp">
	&lt;property name="id" value="1"/>
	&lt;property name="name" value="qm"/>
&lt;/bean>

# 方法三：内部配置

&lt;bean id="Department"  class="cn.apeius.collections.Department">
    &lt;property name="emp">
        <font color='red'>&lt;bean class="cn.apeius.collections.Emp">
			&lt;property name="id" value="1"/>
			&lt;property name="name" value="qm"/>
		&lt;/bean></font>
    &lt;/property>
&lt;/bean>

</pre>

### bean的作用域scope

![](http://i.imgur.com/1Dzlr62.png)

* singleton(默认)，单态，尽量使用scope="singleton",不要使用prototype,因为这样对我们的性能影响较大，除非有必要.
<pre>
//获取两个student
Student s1=(Student) ac.getBean("student");
Student s2=(Student) ac.getBean("student");
System.out.println(s1 == s2);//一样
</pre>
* prototype
<pre>
//获取两个student
Student s1=(Student) ac.getBean("student");
Student s2=(Student) ac.getBean("student");
System.out.println(s1 == s2);//不一样
</pre>
* request
* session
* global-session 是在web开发中才有意义

### 如何给集合类型注入值

java中主要的集合有几种: map set list / 数组 

***给数组注入值***

	<property name="empName">
		<list>
			<value>小明</value>
			<value>小明小明</value>
			<value>小明小明小明小明</value>
		</list>
	</property>

***给list注入值 list 中可以有相同的对象***

	<property name="empList">
		<list>
			<ref bean="emp2" />
			<ref bean="emp1"/>
			<ref bean="emp1"/>
		</list>
	</property>

	<bean id="emp1" class="com.hsp.collection.Employee">
		<property name="name" value="北京"/>
		<property name="id" value="1"/>
	</bean>

	<bean id="emp2" class="com.hsp.collection.Employee">
		<property name="name" value="天津"/>
		<property name="id" value="2"/>
	</bean>

***给set注入值set不能有相同的对象***

	<property name="empsets">
		<set>
			<ref bean="emp1" />
			<ref bean="emp2"/>
			<ref bean="emp2"/>
		</set>
	</property>

***给map注入值，key为索引，value指定值，如果为Java对象，则使用ref指定，或者使用bean定义。如果key为对象，使用key-ref属性***

	<property name="empMaps">
		<map>
			<entry key="11" value-ref="emp1" /> 
			<entry key = "12" value = "emp2" />
			<entry key-ref="13" value="emp3" />
			<entry key-ref="14" value-ref="emp4" />
		</map>
	</property>

***给属性集合配置，即Property对象***

	<property name="pp">
		<props>
			<prop key="pp1">abcd</prop>
			<prop key="pp2">hello</prop>
		</props>
	</property>

### 内部bean

	<bean id="foo" class="....Foo">
		<property name="属性">
			<!—第一方法引用-->
			<ref bean="bean对象名"/>
			
			<!—第二种方法，内部bean-->
			<bean> 
				<properyt name="" value=""></property>
			</bean>
		</property>
	</bean>

### 继承配置

	public class Student 
	public class Gradate extends Student
	
	在beans.xml文件中体现配置 
	<!-- 配置一个学生对象 -->
	<bean id="student" class="com.hsp.inherit.Student">
		<property name="name" value="顺平" />
		<property name="age" value="30"/>
	</bean>
	<!-- 配置Grdate对象 -->
	<bean id="grdate" parent="student" class="com.hsp.inherit.Gradate">
		<!-- 如果自己配置属性name,age,则会替换从父对象继承的数据  -->
		<property name="name" value="小明"/>
		<property name="degree" value="学士"/>
	</bean>


### 通过构造函数注入值★★★

* 目前我们都是通过set方式给bean注入值，spring还提供其它的方式注入值，比如通过构造函数注入值!
* set注入的缺点是无法清晰表达哪些属性是必须的，哪些是可选的；构造注入的优势是通过构造器***强制依赖关系***
* 每个constructor-arg配置一个参数，参数有先后顺序，顺序要与构造函数相同

#### 通过参数名传递参数

	<bean id="product" class="constructor.Product">
        <constructor-arg name="name" value="洗衣机"/>
        <constructor-arg name="description" value="家用洗衣服的工具"/>
    </bean>

#### 通过指数方式传递参数

	<bean id="product" class="constructor.Product">
        <constructor-arg index="0" value="洗衣机"/>
        <constructor-arg index="1" value="家用洗衣服的工具"/>
    </bean>

### 初始化bean和销毁bean的时候执行某个方法

方法一：通过注解@PostConstruct 和 @PreDestroy 方法 实现初始化和销毁bean之前进行的操作

	public class DataInitializer{     
	    @PostConstruct  
	    public void initMethod() throws Exception {  
	        System.out.println("initMethod 被执行");  
	    }  
	    @PreDestroy  
	    public void destroyMethod() throws Exception {  
	        System.out.println("destroyMethod 被执行");  
	    }  
	}  

方法二：通过 在xml中定义 init-method 和  destory-method 方法★★★★★

DataInitializer

	public class DataInitializer{  
	    public void initMethod() throws Exception {  
	        System.out.println("initMethod 被执行");  
	    }  
	    public void destroyMethod() throws Exception {  
	        System.out.println("destroyMethod 被执行");  
	    }  
	}  

applicationContext.xml

	<bean id="dataInitializer" class="com.somnus.demo.DataInitializer" init-method="initMethod" destory-method="destroyMethod"/>  

方法三：通过bean实现InitializingBean和 DisposableBean接口

<pre>
public class DataInitializer <font color='red'>implements InitializingBean，DisposableBean</font>{  
      
    @Override  
    public void afterPropertiesSet() throws Exception {  
        System.out.println("afterPropertiesSet 被执行");  
    }  
      
    @Override  
    public void destroy() throws Exception {  
        System.out.println("destroy 被执行");  
    }  
  
}  
</pre>

原理

<pre>
<font color='red'>//判断该bean是否实现了实现了InitializingBean接口，如果实现了InitializingBean接口，则只掉调用bean的afterPropertiesSet方法</font>  
<font color='blue'>boolean isInitializingBean = (bean instanceof InitializingBean);  </font>
if(isInitializingBean)
	((InitializingBean) bean).afterPropertiesSet();  
</pre>

## 注解装配Bean

### 引入context名称空间，并配置扫描包

	<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd">

    <context:component-scan base-package="annotation"/>

	</beans>

### @Component

	@Component(value = "a")等价于<bean id = "a" class = "A">

	@Component(value = "a")
	public class A {
	
	    public void say(){
	        System.out.println("A");
	    }
	
	}

### 依赖注入

#### 简单类型数据注入 

spring3.0 提供 @Value 注解，可以注入简单数据类型

	@Component(value = "a")
	public class A {
	
	    @Value(value = "qm")
	    private String name;
	
	    public void say(){
	        System.out.println("A" + name);
	    }
	
	}

#### 复杂对象类型数据注入 

**第一种：按类型注入，@Autowired**

A

	@Component(value = "a")
	public class A {
	
	    @Autowired
	    B b;
	
	    public void say(){
	        b.say();
	    }
	
	}

B

	@Component(value = "b")
	public class B {
	
	    public void say(){
	        System.out.println("B");
	    }
	
	}

**第二种：按名称注入，使用@Autowired 结合 @Qualifier 注解 (Spring 2.0 )**

A：

	@Component(value = "a")
	public class A {
	
	    @Autowired
	    @Qualifier(value = "c_qualifier")
	    B b;
	
	    public void say(){
	        b.say();
	    }
	
	}

B：

	@Component(value = "c_qualifier")
	public class B {
	
	    public void say(){
	        System.out.println("B");
	    }
	
	}



**第三种： 使用@Resouce注解 （JSR-250标准 ）**
 
	按照名称注入

**第四种： 使用 @Inject 注解 （JSR-330标准 ）**

	导入 javax.inject-1.jar 

## 自动装配bean的属性值

自动装配只有在属性没有设置时，才会进行

![](http://i.imgur.com/wL8acDX.png)

### byName的用法:

	<!-- 配置一个master对象 -->
	<bean id="master" class="com.hsp.autowire.Master" autowire="byName">
		<property name="name">
			<value>顺平</value>
		</property>
	</bean>
	<!-- 配置dog对象 -->
	<bean id="dog" class="com.hsp.autowire.Dog">
		<property name="name" value="小黄"/>
		<property name="age" value="3"/>
	</bean>

原理图:

property中没有注入dog值，master中的dog为null。但当设置属性autowire="byName"后，通过检测发现内存中有一个名字为dog的对象，则自动进行引用连接

![](http://i.imgur.com/aYY9TE0.png)

### byType的用法

寻找和属性类型相同的bean，此时id为dog11也能够找到并进行装配；找不到、装不上、找到多个抛异常

	<!-- 配置一个master对象 -->
	<bean id="master" class="com.hsp.autowire.Master" autowire="byType">
		<property name="name">
			<value>顺平</value>
		</property>
	</bean>
	<!-- 配置dog对象 -->
	<bean id="dog11" class="com.hsp.autowire.Dog">
		<property name="name" value="小黄"/>
		<property name="age" value="3"/>
	</bean>

### constructor的用法

与byType类似， 查找和bean的构造参数一致的一个或多个bean，若找不到或找到多个，抛异常。按照参数的类型装配  

*master写构造函数*
	
	public master(Dog dog){
		this.dog = dog;
	}

*配置*

	<!-- 配置一个master对象 -->
	<bean id="master" class="com.hsp.autowire.Master" autowire="constructor">
		<property name="name">
			<value>顺平</value>
		</property>
	</bean>
	<!-- 配置dog对象 -->
	<bean id="dog" class="com.hsp.autowire.Dog">
		<property name="name" value="小黄"/>
		<property name="age" value="3"/>
	</bean>

### autodetect的用法

`autowire="autodetect"`(3)和(2)之间选一个方式。不确定性的处理与(3)和(2)一致

### defualt

这个需要在`<beans defualt-autorwire="指定" />`
当你在`<beans>`指定了default-atuowrite后，所有的bean的默认的autowire就是指定的装配方法;
如果没有在`<beans defualt-autorwire="指定" />`没有defualt-autorwire="指定"，则默认是defualt-autorwire=”no”

### no: 不自动装配

这是autowire的默认值

## 使用spring的特殊bean,完成分散配置

将配置文件分成几个分散的配置文件，如一个项目中连接多个数据库，每个数据库各对应一个db.properties文件

* 引入我们的db.properties文件，并在要注入值的地方用$占位符


	<context:property-placeholder location="classpath:com/hsp/dispatch/db.properties,classpath:com/hsp/dispatch/db2.properties"/>
	<!-- 配置一DBUtil对象 $占位符号 -->
	<bean id="dbutil" class="com.hsp.dispatch.DBUtil">
		<property name="name" value="${name}" />
		<property name="drivername" value="${drivername}" />
		<property name="url" value="${url}" />
		<property name="pwd" value="${pwd}" />
	</bean>

注意：当通过`context:property-placeholder`引入属性文件的时候，有多个需要使用`,`间隔.

* db.properties


	name=scott
	drivername=oracle:jdbc:driver:OracleDirver
	url=jdbc:oracle:thin:@127.0.0.1:1521:hsp
	pwd=tiger


* 测试


	ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
	DBUtil dBUtil = (DBUtil) ac.getBean("dbutil");
	System.out.println(dbUtil.getDrivername());

## AOP编程

* <font color='red'>**AOP(Aspect Oriented Programming )，面向切面编程：AOP编程就是将共有的代码，如日志记录、权限控制、事务控制等全部抽取出来，放在某个地方集中管理，若需要使用这些功能，由容器动态织入这些共有代码**</font>，这样的好处：
1、程序员在编写业务逻辑时只需关心核心的业务逻辑处理方法，提高工作效率，使代码变得简洁
2、业务逻辑代码和共有代码分开存放，使维护工作变得轻松

![](http://i.imgur.com/CBjTgky.png)

* 举个例子，在开发过程中，很多对象需要做同一类的操作，例如权限控制、日志记录、事务控制等，AOP编程就是把相同工作剥离出来，若对象需要使用其中的一个功能，则将其织入进去
![](http://i.imgur.com/zNOHFYJ.png)

* AOP编程，实际上在开发框架本身用的多，在实际项目中，用的不是很多，但是将来会越来越多，这是一个趋势

参考：http://blog.csdn.net/liujiahan629629/article/details/18864211

### AOP技术的实现原理

* AOP技术是建立在 **Java语言的反射机制** 与 **动态代理机制** 之上的
* 业务逻辑组件在运行过程中，<font color='red'>**AOP容器** 会动态创建一个 **代理对象(Service的代理对象)** 供使用者调用，该代理对象已经按Java EE程序员的意图将 **切面** 成功切入到 **目标对象** 的 **连接点** 上</font>，从而使 **切面的功能** 与 **业务逻辑的功能** 同时得以执行
* 从原理上讲，调用者直接调用的其实是AOP容器动态生成的代理对象，再由**代理对象调用目标对象完成原始的业务逻辑处理**，而代理对象则已经将切面与业务逻辑方法进行了合成

![](http://i.imgur.com/OJoBkfk.jpg)

现将图6-6中涉及到的一些概念解释如下：(<font color='red'>**切面包括通知和切入点，容器将切面切入到目标对象的连接点上，返回一个代理对象，这个过程叫做织入**</font>)

**切面（Aspect）：**其实就是<font color='red'>**共有功能的实现，包括通知和切入点**</font>,如日志切面、权限切面、事务切面等。在实际应用中通常是一个存放共有功能实现的普通Java类，之所以能被AOP容器识别成切面，是在配置中指定的。
**通知（Advice）：** ***是切面的具体实现***。以目标方法为参照点，根据 ***放置的地方不同***，可分为前置通知（Before）、后置通知（AfterReturning）、环绕通知（Around）、异常通知（AfterThrowing）、最终通知（After）5种。在实际应用中通常是切面类中的一个方法，具体属于哪类通知，同样是在配置中指定的。、
**切入点（Pointcut）：*****用于定义通知应该切入到哪些连接点上***。不同的通知通常需要切入到不同的连接点上，这种精准的匹配是由切入点的 ***正则表达式*** 来定义的

**连接点（Joinpoint）**：就是程序在运行过程中能够***插入切面的地点***。例如，方法调用、异常抛出或字段修改等
**目标对象（Target）**：就是那些即将切入切面的对象，也就是那些被通知的对象。这些对象中已经只剩下干干净净的核心业务逻辑代码了，所有的共有功能代码等待AOP容器的切入。
**代理对象（Proxy）：**将通知应用到目标对象之后被动态创建的对象。可以简单地理解为，代理对象的功能等于目标对象的核心业务逻辑功能加上共有功能。代理对象对于使用者而言是透明的，是程序运行过程中的产物。
**织入（Weaving）：** ***将切面应用到目标对象从而创建一个新的代理对象的过程。***这个过程可以发生在编译期、类装载期及运行期，当然不同的发生点有着不同的前提条件。譬如发生在编译期的话，就要求有一个支持这种AOP实现的特殊编译器；发生在类装载期，就要求有一个支持AOP实现的特殊类装载器；只有发生在运行期，则可直接通过Java语言的反射机制与动态代理机制来动态实现。

举个例子解释术语：

![](http://i.imgur.com/Snv0YKO.png)

AOP编程可以在不增加原来业务逻辑方法代码的情况下，**扩展某个方法**，传统的方法是采用 **继承** 的方式，现在可以采用**动态代理技术**

连接点表示哪些方法 **可以被扩展**（拦截）；切入点表示哪些方法 **需要被扩展**（拦截）；织入是把通知应用到目标上，生成动态代理类的过程；切面表示公用的业务逻辑，包括多个切入点和多个通知

### AOP底层实现

#### JDK动态代理

动态代理可以让一个类 代理多个不同的目标类，而且可以 代理不同的方法

详见Java——设计模式版块~
http://qianmingxs.com/2016/07/06/[%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F]%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E7%9A%84%E8%89%BA%E6%9C%AF%E4%B9%8B%E9%81%93%E7%AC%94%E8%AE%B0/

1、必须针对接口进行代理 
2、生成代理对象，通过Proxy类进行代理，传入目标对象类加载器、目标对象接口 、处理类 
3、自己实现InvocationHandler 接口 
 
![](http://i.imgur.com/fzfob6x.png)

说明：

#### Cglib动态代理机制 

JDK只能对接口进行代理，如果目标对象没有接口，无法使用JDK动态代理，则可以使用cglib 

**什么是CGLIB？**

CGLIB(Code Generation Library)是一个开源项目！是一个强大的，高性能，高质量的Code生成类库，它可以在运行期扩展Java类与实现Java接口。Cglib可以对接口或者类进行代理 ！

#### Spring AOP

Spring AOP 就是基于JDKProxy 和 CglibProxy 

1、如果目标对象有接口，优先使用JDK Proxy
2、如果目标对象没有接口， 使用CglibProxy 

面试题：spring的aop中，当你通过代理对象去实现aop的时候，获取的ProxyFactoryBean是什么类型？
答: 返回的是一个代理对象,如果目标对象实现了接口，则spring使用jdk 动态代理技术,如果目标对象没有实现接口，则spring使用CGLIB技术.

### Spring AOP三种配置详细介绍

**AOP框架三足鼎立：**

1. AspectJ
2. Jboss AOP
3. Spring  AOP


**Spring提供4种AOP支持**

1. 基于代理的经典AOP
2. 纯POJO切面（使用XML）
3. @AspcetJ注解驱动的切面
4. 注入式AspcetJ切面

#### 基于代理的经典AOP

![](http://i.imgur.com/eK3KdQ1.png)

现在有个需求，在调用sayHello()方法前写日志，思路如下：

1、面向接口编程，定义一个TestServiceInter接口，声明函数sayHello()
2、两个类Test1Service和Test2Service实现这两个接口
3、传统的方式很简单，在sayHello()方法前加入日志操作的代码，但如果有多个业务逻辑方法都需要写日志操作，是会有很多冗余代码。**可以引入一个类，它的功能是写日志**，

	Test1Service t1 = new Test1Service();
	//采用传统方法，此处写日志操作
	t1.sayHello();
	
	Test2Service t2 = new Test2Service();
	//采用传统方法，此处写日志操作
	t2.sayHello();

4、Service类与日志类如何关联起来呢，需引入一个代理类，Spring提供一个代理对象类ProxyFactoryBean

##### 步骤

1、定义接口

	TestServiceInter.java
	public interface TestServiceInter {
	    public void sayHello();
	}

2、编写对象(也称作被代理对象或目标对象)，实现接口

	Test1Service.java
	public class Test1Service implements TestServiceInter {
	    private String name;
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	
	    @Override
	    public void sayHello() {
	        System.out.println("Hi " + name);
	    }
	}

3、编写通知（以前置通知为例，前置通知在目标方法调用前调用）

	# MyMethodBeforeAdvice.java
	public class MyMethodBeforeAdvice implements MethodBeforeAdvice {
	    @Override
	    public void before(Method method, Object[] objects, Object o) throws Throwable {
	        System.out.println("记录日志：" + method.getName());
	    }
	}

4、在beans.xml文件配置（分三部分：通知、被代理对象、代理对象）
　　4.1	配置目标对象
　　4.2	配置通知
　　4.3	配置代理对象 是 ProxyFactoryBean的对象实例
　　　　4.3.1 代理接口集
　　　　4.3.2 织入通知
　　　　4.3.3 配置目标对象

	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	    <!--配置前置通知，比如日志-->
	    <bean id="MyMethodBeforeAdvice" class="cn.apeius.AOP.MyMethodBeforeAdvice"></bean>
	
	    <!--配置目标对象-->
	    <bean id="Test1Service" class="cn.apeius.AOP.Test1Service">
	        <property name="name" value="钱明"/>
	    </bean>
	
	    <!--配置代理对象,spring提供-->
	    <bean id="proxyFactoryBean" class="org.springframework.aop.framework.ProxyFactoryBean">
	        <!--配置代理接口集-->
	        <property name="proxyInterfaces">
	            <list>
	                <value>cn.apeius.AOP.TestServiceInter</value>
	            </list>
	        </property>
	        <!--通知-->
	        <property name="interceptorNames">
	            <!--相当于包MyMethodBeforeAdvice前置通知和代理对象关联，我们也可以把通知看成拦截器-->
	            <value>MyMethodBeforeAdvice</value>
	        </property>
	        <!--配置被代理对象-->
	        <property name="target" ref="Test1Service"/>
	    </bean>
	    
	</beans>

##### 通知类型

![](http://i.imgur.com/DR0KqCg.png)
除了四种基本通知外，还有引入通知

前置通知

	public class MyMethodBeforeAdvice implements MethodBeforeAdvice {
	    @Override
	    public void before(Method method, Object[] objects, Object o) throws Throwable {
	        System.out.println("记录日志：" + method.getName());
	    }
	}

后置通知

	public class MyAfterReturningAdvice implements AfterReturningAdvice {
	    @Override
	    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
	        System.out.println("关闭资源");
	    }
	}

环绕通知

	public class MyMethodInterceptor implements MethodInterceptor {
	    @Override
	    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
	        System.out.println("环绕前");
	        Object o = methodInvocation.proceed();
	        System.out.println("环绕后");
	        return o;
	    }
	}

异常通知
　　ThrowsAdvice为标记接口，在接口中没有任何方法，因为方法被反射机制调用，实现类必须实现以下形式，见文档

	public class MyThrowsAdvice implements ThrowsAdvice {
	    public void afterThrowing(Exception e){
	        System.out.println("出大事了" + e.getMessage());
	    }
	}

引入通知
　　引入通知不需要编写相应的类，只需要进行配置，目的是用来指定哪些方法需要执行相应的通知，如，我们想指定只有sayHello（）方法执行前置通知，

	<bean id="myMethodBeforeAdviceFilter" class="org.springframework.aop.support.NameMatchMethodPointcutAdvisor">  
	    <property name="advice" ref="myMethodBeforeAdvice"></property>  
	    <property name="mappedNames">  
	        <list>  
	            <value>sayHello</value>  
	        </list>  
	    </property>  
	</bean>

	拦截器名集中引入
 	<value>myMethodBeforeAdviceFilter</value>

#### 传统AOP切面编程

极大简化了spring切面的配置工作，同时也让程序透明化，隐藏了切面的很多细节。上面所有内容都可以作为理解 **spring配置AOP的基础**，是最原始的配置方式，也体现了spring处理的过程。

使用ProxyFactoryBean配置有些欠优雅，在spring2.0里新的xml配置元素体现了改进。Spring2.0在aop命名空间里提供了一些配置元素，简化了把类转化为切面的操作。

**本质的使用同上，只是简化配置，隐藏细节**

##### AspectJ切入点

传统AOP切入点，使用正则表达式语法，不推荐使用。AspectJ切入点，是通过函数进行配置
 
常用语法说明简介:
execution 执行，语法：execution(<访问修饰符>?<返回类型><方法名>(<参数>)<异常>)

	execution(* *(..))  第一个* 任意返回类型 ， 第二个* 任意方法名 , .. 任意参数 
	
	execution(* cn.itcast.service.UserService.*(..)) 匹配UserService所有方法 第一个星 任意返回类型
	
	execution(* cn.itcast.service.UserService+.*(..)) 匹配UserService子类所有方法 + 子类
	
	execution(* cn.itcast.service..*.*(..)) 第一个.. 任意子包 *.*任何类的任何方法 
	
within 根据包匹配

	语法：within(包名..*) 
	within(cn.itcast.service..*) 拦截service下所有类的方法 
	
this根据目标类型匹配

	语法：this(类名) 
	this(cn.itcast.service.UserService) 拦截 UserService所有方法 (包括代理对象)
	
target 根据目标类型匹配

	语法 ：target(类名)
	target(cn.itcast.service.UserService) 拦截UserService所有方法 （不包括代理对象 ）
	
args 根据参数匹配

	args(java.lang.String) 拦截所有参数为String类的方法 
	
bean 根据bean name匹配 

	bean(userService) 拦截bean id/name为userService对象所有方法 

##### 步骤

接口
	
	public interface IAopService {
		public void withAop() throws Exception;
	}

实现

	public class AopServiceImpl implements IAopService {
		public void withAop() throws Exception {
			System.out.println("业务逻辑处理中");
		}
	}

通知

	public class MyMethodInterceptor implements MethodInterceptor{
	    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
	        System.out.println("环绕前");
	        Object o = methodInvocation.proceed();
	        System.out.println("环绕后");
	        return o;
	    }
	}

配置文件

	<beans
	        xmlns="http://www.springframework.org/schema/beans"
	        xmlns:aop="http://www.springframework.org/schema/aop"
	        xmlns:tx="http://www.springframework.org/schema/tx"
	        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	        xmlns:p="http://www.springframework.org/schema/p"
	        xsi:schemaLocation="http://www.springframework.org/schema/beans
	    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
	    http://www.springframework.org/schema/aop
	    http://www.springframework.org/schema/aop/spring-aop.xsd">
	
	    <!--1、目标-->
	    <bean id="aopServiceImpl" class="AOP.AopServiceImpl"/>
	    <!--2、通知-->
	    <bean id="myMethodInterceptor" class="AOP.MyMethodInterceptor"/>
	    <!--<aop:config> 配置切面=通知+切入点-->
	    <aop:config>
	        <!--aop:pointcut配置切入点-->
	        <aop:pointcut id="service" expression="execution(* AOP.*.*(..))"/>
	        <!--aop:advisor配置传统Spring AOP切面，只能有一个切入点和一个通知-->
	        <aop:advisor advice-ref="myMethodInterceptor" pointcut-ref="service"/>
	    </aop:config>
	</beans>

测试

	public class AopRun {
	
		public static void main(String[] args) throws Exception {
	
			ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
			IAopService hello = (IAopService) context.getBean("aopServiceImpl");
			hello.withAop();
		}
	
	}
####  AspectJ AOP切面编程

接口

	public interface IAopService {
		public void withAop() throws Exception;
	}

实现

	public class AopServiceImpl implements IAopService {
		public void withAop() throws Exception {
			System.out.println("业务逻辑处理中");
		}
	}

通知

	public class MyAspect {
	    public void before1(JoinPoint jointPoint){
	        System.out.println("before1");
	    }
	    public void before2(JoinPoint jointPoint){
	        System.out.println("before2");
	    }
	}

配置

    <!--1、目标-->
    <bean id="aopServiceImplAspectJ" class="AOP.AspectJ.AopServiceImpl"/>
    <!--2、通知-->
    <bean id="myMethodInterceptorAspectJ" class="AOP.AspectJ.MyAspect"/>
    <!--配置切面-->
    <aop:config>
        <aop:aspect ref="myMethodInterceptorAspectJ">
            <!--标签before决定前置通知-->
            <aop:before method="before1" pointcut-ref="serviceAspectJ"/>
            <aop:before method="before2" pointcut-ref="serviceAspectJ"/>
            <!--切入点-->
            <aop:pointcut id="serviceAspectJ" expression="execution(* AOP.AspectJ.*.*(..))" />
        </aop:aspect>
    </aop:config>

测试

	public class AopRun {
	
		public static void main(String[] args) throws Exception {
	
			ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
			IAopService hello = (IAopService) context.getBean("aopServiceImplAspectJ");
			hello.withAop();
		}
	
	}

#### @AspcetJ注解驱动的切面★★★★★★★★★

1、接口

	public interface IAopService {
		public void withAop() throws Exception;
	}

2、实现

	@Service
	public class AopServiceImpl implements IAopService {
		public void withAop() throws Exception {
			System.out.println("业务逻辑处理中");
		}
	}


3、切面，包括切入点和通知

	@Aspect
	//表示当前类是一个通知
	public class MyAspect {
	
	    private static final Logger logger = Logger.getLogger(MyAspect.class);
	
	    //定义切入点方法一：定义在一个空方法上
	    @Pointcut("execution(* *.*(..)))")
	    private void myPointcut() {
	    }
	
	    //定义切点方法二：直接在通知上定义切入点
	    @Before("execution(* *.*(..)))")
	    public void before(JoinPoint joinPoint) {
	        String className = joinPoint.getTarget().getClass().getName();
	        String methodName = joinPoint.getSignature().getName();
	        logger.warn(className + "的" + methodName + "执行了");
	        Object[] args = joinPoint.getArgs();
	        StringBuilder log = new StringBuilder("参数为：");
	        for (Object arg : args) {
	            log.append(arg + " ");
	        }
	        logger.warn(log.toString());
	    }
	
	    @AfterReturning(value = "execution(* *.*(..)))", returning = "returnVal")
	    public void afterReturin(Object returnVal) {
	        logger.warn("方法正常结束了,方法的返回值:" + returnVal);
	    }
	
	    @AfterThrowing(value = "myPointcut()", throwing = "e")
	    public void afterThrowing(Throwable e) {
	        logger.error("AfterThrowing" + e.getMessage(),e);
	    }
	
	    @Around(value = "myPointcut()")
	    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
	        logger.warn("Around:前置增强");
	        Object result = null;
	        try {
	            result = proceedingJoinPoint.proceed();
	        } catch (Exception e) {
	            logger.error("Around:" + e.getMessage(),e);
	        }
	        logger.warn("Around:后置增强");
	        return result;
	    }
	}

4、配置

	<?xml version="1.0" encoding="UTF-8"?>
	<beans
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p" xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="AOP"/>
    <!--1、开启AspectJ注解开发的配置-->
    <aop:aspectj-autoproxy/>
    <!--配置切面-->
    <bean id = "MyAspect" class="AOP.AspectJAnnotation.MyAspect"/>

</beans>

5、测试

	public class AopRun {
	
		public static void main(String[] args) throws Exception {
			ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
			IAopService aopService = (IAopService) context.getBean(IAopService.class);
			aopService.withAop();
		}
	
	}

spring 的环绕通知和前置通知，后置通知有着很大的区别，主要有两个重要的区别：

1. 目标方法的调用由环绕通知决定，即你可以决定是否调用目标方法，而前置和后置通知   是不能决定的，他们只是在方法的调用前后执行通知而已，即目标方法肯定是要执行的。

2. 环绕通知可以控制返回对象，即你可以返回一个与目标对象完全不同的返回值，虽然这很危险，但是你却可以办到。而后置方法是无法办到的，因为他是在目标方法返回值后调用





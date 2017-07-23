---
title: Shiro笔记

date: 2016-10-17 23:00:00

categories:
- SSM

tags:
- Shiro

---

## 术语（Terminology）

Authentication（鉴权，身份验证）：校验Subject（主体）的身份是否合法
Authorization（授权）：根据主体的角色和权限来决定允许或拒绝访问请求
Cipher：执行加密或解密的算法
Principal：一个主体的标识属性，独一无二的标识，可以是昵称、用户ID、安全码……
Credential（证书）：验证主体身份的信息
Cryptography（密码学）：密码学保护信息不被不比希望让他看的人的看到
Hash：数据经过hash方法得到的结果不可逆
Permission：描述原始的功能，定义应用程序能够做的一些操作
Realm（领域）：可以访问应用程序的安全数据，例如用户、角色和权限的组件。可以当做安全的DAO，将应用程序数据翻译成Shiro能够理解的格式，这样Shiro可以提供简洁的API，不管有多少数据源或者应用程序
**Role：权限的集合，并取一个独一无二的名字**
Session：主体与软件交互过程中的有状态数据环境，当用户使用应用程序时可以在Session中增加/读取/删除数据，当用户退出应用程序，Session终止
Subject：应用程序用户，但他不一定但指一个人，它也可以代表调用你应用程序的外部进程，或者一个间断执行操作的守护系统账户，它可以抽象表示为操作应用程序的主体

## 简介

Apache Shiro是一个强大易用的Java安全框架，提供鉴权、授权、加密、会话管理的综合解决方法。在实践中达到管理应用程序安全的作用，而减少与应用程序的耦合

Shiro可以在任何环境中运行，从最简单的命令行应用程序到企业级web程序和集群应用程序

## Shiro快速入门（j2se）

官方地址：http://shiro.apache.org/10-minute-tutorial.html

### Shiro API

	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.authc.*;
	import org.apache.shiro.config.IniSecurityManagerFactory;
	import org.apache.shiro.mgt.SecurityManager;
	import org.apache.shiro.session.Session;
	import org.apache.shiro.subject.Subject;
	import org.apache.shiro.util.Factory;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	
	public class Quickstart {
	
	    private static final transient Logger log = LoggerFactory.getLogger(Quickstart.class);
	
	    public static void main(String[] args) {
	
			/*
			* 创建Shiro SecurityManager，同时设置realms、users、roles、permissions，
			* 最简单的方式是创建一个.ini，通过它返回一个SecurityManager实例。ini文件见下一节
			*/
	        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
	        SecurityManager securityManager = factory.getInstance();
	
			// 设置 SecurityManager 为一个静态实例，可以被 JVM 访问到
	        SecurityUtils.setSecurityManager(securityManager);
	
			// 设置Shiro环境完毕，看下它的操作★★★★★
	
	        // 获得当前操作的用户，此时currentUser为空，用户还未登录
	        Subject currentUser = SecurityUtils.getSubject();
	
	        // 用Session存储数据（web或EJB容器不需要！！！）
	        Session session = currentUser.getSession();
	        session.setAttribute("someKey", "aValue");
	        String value = (String) session.getAttribute("someKey");
	        if (value.equals("aValue")) {
	            log.info("Retrieved the correct value! [" + value + "]");
	        }
	
			// 用户登录，校验角色和权限
	        if (!currentUser.isAuthenticated()) {
				//获得用户名和密码
	            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
	            token.setRememberMe(true);
	            try {
	                currentUser.login(token);
	            } catch (UnknownAccountException uae) {
	                log.info("There is no user with username of " + token.getPrincipal());
	            } catch (IncorrectCredentialsException ice) {
	                log.info("Password for account " + token.getPrincipal() + " was incorrect!");
	            } catch (LockedAccountException lae) {
	                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
	                        "Please contact your administrator to unlock it.");
	            }
	            // 捕捉更多的异常，可以是用户自定义
	            catch (AuthenticationException ae) {
	                //unexpected condition?  error?
	            }
	        }
	
	        // 查看用户标识
	        log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");
	
	        // 测试用户是否具备角色
	        if (currentUser.hasRole("schwartz")) {
	            log.info("May the Schwartz be with you!");
	        } else {
	            log.info("Hello, mere mortal.");
	        }
	
	        // 测试用户是否具备某个权限
	        if (currentUser.isPermitted("lightsaber:weild")) {
	            log.info("You may use a lightsaber ring.  Use it wisely.");
	        } else {
	            log.info("Sorry, lightsaber rings are for schwartz masters only.");
	        }
	
	        // 测试用户是否具备某个权限
	        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
	            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
	                    "Here are the keys - have fun!");
	        } else {
	            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
	        }
	
	        // 注销用户
	        currentUser.logout();
	
	        System.exit(0);
	    }
	}

### shiro.ini
	
<pre>
# 用户和他们分配的角色
[users]
# 用户'root'的密码是'secret'，包含'admin'的角色
root = secret, admin
# 用户'guest'的密码是'guest'，包含'guest'的角色
guest = guest, guest
# 用户'presidentskroob'的密码是'12345'，包含'president'的角色
presidentskroob = 12345, president
# 用户'darkhelmet'的密码是'ludicrousspeed'，包含角色'darklord'和schwartz'
darkhelmet = ludicrousspeed, darklord, schwartz
# 用户'lonestarr'的密码是'vespa'，包含角色'goodguy'和'schwartz'
lonestarr = vespa, goodguy, schwartz

# 角色和他们分配的权限
[roles]
# 'admin'的角色拥有所有权限，用通配符'*'表示
admin = *
# 角色'schwartz'拥有lightsaber下的任意权限
schwartz = lightsaber:*
# 角色'goodguy'允许'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
goodguy = winnebago:drive:eagle5
</pre>

## Web应用程序整合

官方文档：http://shiro.apache.org/webapp-tutorial.html

### 开启Shiro环境

Shiro可以用多重方式配置，取决于你使用的web应用程序框架，如Spring，Guice等，这里采用Shiro默认，最简单的方式：基于INI文件的配置

#### 添加 shiro.ini 文件

位置：src/main/webapp/WEB-INF/shiro.ini

	[main]
	cacheManager = org.apache.shiro.cache.MemoryConstrainedCacheManager
	securityManager.cacheManager = $cacheManager

这个INI文件只简单包含main部分，和一些最小配置

* 定义 cacheManager 的实例
* 在 Shiro securityManager 上配置新的cacheManager的实例

#### 在web.xml文件中支持Shiro

加载shiro.ini文件启动新的Shiro环境，使Web应用程序支持Shiro环境

	<listener>
	    <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
	</listener>
	
	<filter>
	    <filter-name>ShiroFilter</filter-name>
	    <filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
	</filter>
	
	<filter-mapping>
	    <filter-name>ShiroFilter</filter-name>
	    <url-pattern>/*</url-pattern>
	    <dispatcher>REQUEST</dispatcher>
	    <dispatcher>FORWARD</dispatcher>
	    <dispatcher>INCLUDE</dispatcher>
	    <dispatcher>ERROR</dispatcher>
	</filter-mapping>

* 定义一个 ServletContextListener，应用程序启动时启动Shiro环境（包括Shiro SecurityManager）。**监听器默认寻找 WEB-INF/shiro.ini 文件** 作为 Shiro 的配置文件
* 过滤器拦截所有请求，所以Shiro在请求达到应用程序之前，能够执行必要的身份验证和访问控制操作
* &lt;filter-mapping>声明确保所有请求类型都能被ShiroFilter拦截，一般filter-mapping声明不需要指定dispatcher元素，但是Shiro需要定义它们，这样可以过滤所有不同的请求类型

#### 运行webapp

	mvn tomcat:run

这些输出指明Shiro确定在你的webapp中运行

	15:41:22.296 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Starting Shiro environment initialization.
	15:41:22.733 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Shiro environment initialized in 435 ms.

### 连接到用户仓库

#### Realm

* 在我们登录、退出、执行访问控制等其他安全相关的操作之前，我们需要用户！所有我们会设置Shiro去访问用户仓库
* 用户仓库会有多种形式：MySQL数据库、MongoDB、LDAP、活动目录、简单文件、其他私有数据仓库，Shiro通过 **Realm** 来操作这些
* Realm扮演Shiro与安全数据之间的桥梁，也可以说是连接器。当用户执行鉴权（登录）或授权（访问控制），Shiro从应用程序配置的所有Realm中寻找数据
* Realm 是一个安全DAO，封装了连接细节，Shiro需要数据时通过Realm可以直接获取。Realm可以配置多个，但至少要有一个
* **默认为iniRealm**

#### ini

简单配置了两个用户，用户名密码分别为admin：admin，guest：guest

	[users]
	admin=admin
	guest=guest

### Shiro登录/退出与访问控制

#### ini文件，添加访问控制

	[main]
	# 对于Shiro任何默认过滤器都有一个loginUrl属性，authc过滤器会跳转到登录页面
	shiro.loginUrl = /login.jsp
	
	# 用户
	[users]
	admin = admin
	guest = guest
	
	# 过滤请求，路径相对于应用程序路径[HttpServletRequest.getContextPath()]
	[urls]
	# 匹配到请求/login.jsp，开启authc过滤器，跳转到loginUrl页面
	/login.jsp = authc
	# 匹配到请求/logout，开启logout过滤器，跳转到首页
	/logout = logout
	# 匹配到请求/admin/**，开启authc过滤器，跳转到loginUrl页面
	/admin/** = authc

#### 登录页面

Shiro会默认读取username、password、rememberMe

	<!DOCTYPE html>
	<html>
	<head>
	    <title>Apache Shiro Tutorial Webapp : Login</title>
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <!-- Add some nice styling and functionality.  We'll just use Twitter Bootstrap -->
	    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.2/css/bootstrap.min.css">
	    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.2/css/bootstrap-theme.min.css">
	    <style>
	        body{padding-top:20px;}
	    </style>
	</head>
	<body>
	    <div class="container">
	        <div class="row">
	            <div class="col-md-4 col-md-offset-4">
	                <div class="panel panel-default">
	                    <div class="panel-heading">
	                        <h3 class="panel-title">Please sign in</h3>
	                    </div>
	                    <div class="panel-body">
	                        <form name="loginform" action="" method="POST" accept-charset="UTF-8" role="form">
	                            <fieldset>
	                                <div class="form-group">
	                                    <input class="form-control" placeholder="Username or Email" name="username" type="text">
	                                </div>
	                                <div class="form-group">
	                                    <input class="form-control" placeholder="Password" name="password" type="password" value="">
	                                </div>
	                                <div class="checkbox">
	                                    <label>
	                                        <input name="rememberMe" type="checkbox" value="true"> Remember Me
	                                    </label>
	                                </div>
	                                <input class="btn btn-lg btn-success btn-block" type="submit" value="Login">
	                            </fieldset>
	                        </form>
	                    </div>
	                </div>
	            </div>
	        </div>
	    </div>
	
	    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
	    <script src="https://code.jquery.com/jquery.js"></script>
	    <script src="//netdna.bootstrapcdn.com/bootstrap/3.0.2/js/bootstrap.min.js"></script>
	    <!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
	    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
	    <!--[if lt IE 9]>
	    <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js"></script>
	    <script src="https://oss.maxcdn.com/libs/respond.js/1.3.0/respond.min.js"></script>
	    <![endif]-->
	</body>
	</html>

#### 登录验证页面

如果用户/密码正确，跳转到主页面，否则继续跳转到登录页

	public class LoginServlet extends HttpServlet {
	    @Override
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        String username = request.getParameter("username");
	        String password = request.getParameter("password");
	
	        Subject subject = SecurityUtils.getSubject();
	        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
	        try{
	            subject.login(token);
	        }catch (Exception e){
	            request.getRequestDispatcher("/login.jsp").forward(request,response);
	            return;
	        }
	        request.getRequestDispatcher("/WEB-INF/main.jsp").forward(request,response);
	    }
	}

### 基于角色的权限控制★★★★★

	[main]
	# 认证页面
	shiro.loginUrl = /login.jsp
	# 用户没有权限时跳转页面
	perms.unauthorizedUrl = /unauthorizedUrl.jsp
	roles.unauthorizedUrl = /unauthorizedUrl.jspgi
	
	[users]
	# admin的密码是admin，拥有admin角色，包含admin下的所有权限
	admin = admin,admin
	# guest的密码是guest，拥有user角色，包含user下的所有权限
	guest = guest,user
	# qm的密码是qm，不拥有任何角色，即没有任何权限
	qm = qm
	
	[roles]
	# admin拥有的权限
	admin = admin:*,user:*
	# user拥有的权限
	user = user:*
	
	[urls]
	/login.jsp = annon
	/logout = logout
	
	# /admin/**路径只有拥有admin角色的用户才能访问
	/admin/** = authc,roles[admin]
	# 访问/user/list.jsp必须拥有user:list权限，所以拥有admin或者user的角色都可以访问
	/user/list.jsp = authc,perms[user:list]
	# 访问/user/** 只要认证用户都可以访问
	/user/** = authc


当用户访问一个url时，Shiro从上到下扫描权限，如果有一个匹配则不继续往下进行扫描，所以优先级很重要。对于特殊的访问权限，放在上面

## Shiro细节

### 认证

#### 认证身份流程

![](http://i.imgur.com/aXcmnp3.png)

流程如下：
1、首先调用 Subject.login(token)进行登录，其会自动委托给 Security Manager，调用之前必
须通过 SecurityUtils. setSecurityManager()设置；
2、SecurityManager 负责真正的身份验证逻辑；它会委托给 Authenticator 进行身份验证；
3、 Authenticator 才是真正的身份验证者， Shiro API 中核心的身份认证入口点， 此处可以自定义插入自己的实现；
4、Authenticator 可能会委托给相应的 AuthenticationStrategy 进行多 Realm 身份验证，默认
ModularRealmAuthenticator 会调用 AuthenticationStrategy 进行多 Realm 身份验证；
5、Authenticator 会把相应的 token 传入 Realm，从 Realm 获取身份验证信息，如果没有返
回/抛出异常表示身份验证失败了。此处可以配置多个 Realm，将按照相应的顺序及策略进行访问。

#### 自定义realm

登录操作

	public class realm_test {
	    public static void main(String[] args){
	        // 设置SecurityManager
	        SecurityManager securityManager = new IniSecurityManagerFactory("classpath:shiro.ini").getInstance();
	        SecurityUtils.setSecurityManager(securityManager);
	
	        //获取主体
	        Subject subject = SecurityUtils.getSubject();
	        UsernamePasswordToken token = new UsernamePasswordToken("admin","admin");
	        try{
	            //登录操作，登录不成功会抛出异常
	            subject.login(token);
	            //输出认证的用户名
	            System.out.println(subject.getPrincipal());
	        }catch (UnknownAccountException e){
	            System.out.println("用户名不存在");
	        }catch (IncorrectCredentialsException e){
	            System.out.println("用户名密码错误");
	        }
	    }
	}

自定义realm

	public class MyRealm implements Realm{
	
	    //模拟数据库，放入一些用户名和密码
	    private static Map<String,String> DB = new HashMap<String,String>();
	    static {
	        DB.put("admin","admin");
	        DB.put("qm","qm");
	    }
	
	    @Override
	    //Realm的名称
	    public String getName() {
	        return "MyRealm";
	    }
	
	    @Override
	    //该realm支持哪些token
	    public boolean supports(AuthenticationToken authenticationToken) {
	        return authenticationToken instanceof UsernamePasswordToken;
	    }
	
	    @Override
	    //执行subject.login操作调用的方法
	    public AuthenticationInfo getAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
	        //获取token中的用户和密码
	        String username = authenticationToken.getPrincipal().toString();
	        String password = new String((char[])authenticationToken.getCredentials());
	
	        //验证失败
	        if(!DB.containsKey(username)) throw new UnknownAccountException("用户名不存在");
	        if(!password.equals(DB.get(username))) throw new IncorrectCredentialsException("用户名密码错误");
	        //System.out.println(username + " " + password);
	
	        //验证成功返回一个AuthenticationInfo
	        return new SimpleAuthenticationInfo(username, password,getName());
	    }
	}

配置使用自定义realm

	[main]
	#创建一个MyRealm实例，这句话等同于<bean id = "myRealm" class = "MyRealm"/>
	myRealm = MyRealm
	#依赖注入，在securityManager中传入自定义realm的实例
	securityManager.realms = $myRealm

#### 多 Realm 配置

	#声明一个 realm 
	myRealm1=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm1 
	myRealm2=com.github.zhangkaitao.shiro.chapter2.realm.MyRealm2 
	#指定 securityManager 的 realms 实现
	securityManager.realms=$myRealm1,$myRealm2 

securityManager 会按照 realms 指定的顺序进行身份认证。此处我们使用 **显示指定顺序的方式** 指定了 Realm 的顺序，如果删除"securityManager.realms=$myRealm1,$myRealm2"，那么 securityManager 会**按照 realm 声明的顺序进行使用**（即无需设置 realms 属性，其会自动
发 现），当我们显示指定 realm 后 ， 他没有指定 realm 将被忽 略， 如"securityManager.realms=$myRealm1"，那么 myRealm2 不会被自动设置进去。

#### Shiro 默认提供的 Realm及开发建议★★★★★

![](http://i.imgur.com/URoDJuv.png)

以后一般继承 **AuthorizingRealm** （授权）即可； 其继承了 AuthenticatingRealm （即身份验证） ，而且也间接继承了 CachingRealm（带有缓存实现）。其中主要默认实现如下：
* org.apache.shiro.realm.text.IniRealm： [users]部分指定用户名/密码及其角色； [roles]部分指定角色即权限信息；
* org.apache.shiro.realm.text.PropertiesRealm：user.username=password,role1,role2 指定用户名/密码及其角色；role.role1=permission1,permission2 指定角色及权限信息；
* org.apache.shiro.realm.jdbc.JdbcRealm：通过 sql 查询相应的信息，如 "select password from users where username =?"获取用户密码，"select password, password_salt from users where username =?"获取用户密码及盐；"select role_name from user_roles where username =?"
获取用户角色；"selectpermission from roles_permissions where role_name =?"获取角色对应的权限信息；也可以调用相应的 api 进行自定义 sql；

**jdbcRealm使用**

1、依赖

	<dependency> 
		<groupId>mysql</groupId> 
		<artifactId>mysql-connector-java</artifactId> 
		<version>5.1.25</version> 
	</dependency> 
	<dependency> 
		<groupId>com.alibaba</groupId> 
		<artifactId>druid</artifactId> 
		<version>0.2.23</version> 
	</dependency> 

2、配置数据库

查看jdbcReaml源代码，发现其中固定了一个sql语句，要求数据库中有一个users表，并且有username、password两个字段

![](http://i.imgur.com/b2Degfn.png)

3、配置

	[main]
	# 数据源
	dataSource = com.alibaba.druid.pool.DruidDataSource
	dataSource.driverClassName = com.mysql.jdbc.Driver
	dataSource.url = jdbc:mysql://localhost:3306/jdbcRealm
	dataSource.username = root
	dataSource.password = root
	
	#创建一个jdbcRealm实例，并且注入数据源
	jdbcRealm = org.apache.shiro.realm.jdbc.JdbcRealm
	jdbcRealm.dataSource = $dataSource
	securityManager.realms = $jdbcRealm

4、验证

	public class realm_test {
	    public static void main(String[] args){
	        // 设置SecurityManager
	        SecurityManager securityManager = new IniSecurityManagerFactory("classpath:shiro.ini").getInstance();
	        SecurityUtils.setSecurityManager(securityManager);
	
	        //获取主体
	        Subject subject = SecurityUtils.getSubject();
	        UsernamePasswordToken token = new UsernamePasswordToken("admin","admin");
	        try{
	            //登录操作，登录不成功会抛出异常
	            subject.login(token);
	            //输出认证的用户名
	            System.out.println(subject.getPrincipal());
	        }catch (UnknownAccountException e){
	            System.out.println("用户名不存在");
	        }catch (IncorrectCredentialsException e){
	            System.out.println("用户名密码错误");
	        }
	    }
	}

#### Authenticator接口及其子类源码分析★★★★★★

1、定义接口Authenticator，定义了一个authenticate认证方法

	public interface Authenticator {
	    public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)
	            throws AuthenticationException;
	}

2、抽象类AbstractAuthenticator实现Authenticator接口，类的作用是通过doAuthenticate方法来获取AuthenticationInfo并返回；doAuthenticate方法为抽象方法，由子类重写

	public abstract class AbstractAuthenticator implements Authenticator, LogoutAware {

		public final AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
			AuthenticationInfo info;
			info = doAuthenticate(token);
		}
		protected abstract AuthenticationInfo doAuthenticate(AuthenticationToken token)
		            throws AuthenticationException;
		}
	}

3、ModularRealmAuthenticator继承了AbstractAuthenticator，通过**调用realm的getAuthenticationInfo方法获得AuthenticationInfo**

<pre>
public class ModularRealmAuthenticator extends AbstractAuthenticator {

	<font color='red'>//保存realm的实例</font>
    private Collection&lt;Realm> realms;
    private AuthenticationStrategy authenticationStrategy;

    protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
	
		assertRealmsConfigured();
	    Collection<Realm> realms = getRealms();
	    if (realms.size() == 1) {
	        return <font color='red'>doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);</font>
	    } else {
	        return <font color='red'>doMultiRealmAuthentication(realms, authenticationToken);</font>
	    }
	
	}

	protected AuthenticationInfo doSingleRealmAuthentication(Realm realm, AuthenticationToken token) {
        <font color='red'>AuthenticationInfo info = realm.getAuthenticationInfo(token);</font>
        return info;
    }


	protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token) {
		//若有多个realm，遍历所有realm，根据AuthenticationStrategy 接口指定的策略，返回AuthenticationInfo；默认使用AtLeastOneSuccessfulStrategy策略
		AuthenticationStrategy strategy = getAuthenticationStrategy();
		AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);
		for (Realm realm : realms) {	
			aggregate = strategy.beforeAttempt(realm, token, aggregate);
			AuthenticationInfo info = null;
			info = realm.getAuthenticationInfo(token);
			aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);
		}    
		aggregate = strategy.afterAllAttempts(token, aggregate);   
		return aggregate;
	}
}
</pre>

### 授权

#### 角色

角色代表了权限的集合，赋予用户角色，这样用户就可以拥有一组权限，赋予权限比较方便

* 隐式角色：通过 **判断用户是否有某个角色** 来判断用户有没有操作权限，颗粒度是以角色为单位进行访问控制的，颗粒度较粗，如果应用中允许CTO、技术总监、开发工程师使用打印机，假设某天不允许开发工程师使用打印机了，就需要在相关代码的判断逻辑中移除技术总监角色，造成多处代码的修改
* 显示角色：程序中 **通过权限** 控制谁能访问某个资源。假设哪个角色不能访问某个资源，只需要从角色的权限集合中移除即可，颗粒度是以资源/实例为单位的，颗粒度较细

#### 基于角色的访问控制（隐式角色）

判断用户是否有某个角色来进行权限控制，颗粒度较大，很多地方对角色进行判断，如果有一天不需要了就要**修改代码**，在判断这个角色的地方把它删除

**ini配置文件：**

	[users]
	admin = admin,r1,r2

**Shiro 提供了 hasRole/hasRole 用于判断用户是否拥有某个角色/某些权限**

	//判断是否拥有角色 r1
    System.out.println(subject.hasRole("r1"));
    //必须同时拥有r1和r2才会true，否则返回false
    System.out.println(subject.hasAllRoles(Arrays.asList("r1","r2")));
    //返回boolean[]数组，相当于判断是否拥有每个角色
    System.out.println(Arrays.toString(subject.hasRoles(Arrays.asList("r1","r3"))));

**Shiro 提供的 checkRole/checkRoles 和 hasRole/hasAllRoles 不同的地方是它在判断为假的情
况下会抛出 UnauthorizedException 异常**

	subject().checkRole("role1"); 

#### 基于资源的访问控制（显示角色）

**ini配置文件**

	[users]
	admin = admin,r1
	
	[roles]
	r1=user:create,user:update
	r2:user:create,user:delete

**测试**

	System.out.println(subject.isPermitted("user:create"));
	System.out.println(subject.isPermittedAll("user:create","user:update"));

**checkPermission/checkPermissions**会在失败的情况下抛出 UnauthorizedException 异常

**规则**

基于资源的访问控制（显示角色），也可以叫基于权限的访问控制，这种方式的一般规则是"资源标识符：操作"，即是资源级别的粒度；这种方式的好处就是如果要修改基本都是一个资源级别的修改，不会对其他模块代码产生影响，粒度小。但是实现起来可能稍微复杂点，需要维护"用户——角色，角色——权限（资源：操作）"之间的关系

#### 权限

规则："资源标识符：操作：对象实例ID"，即对哪个资源的哪个实例可以进行什么操作。其默认支持通配符权限字符串，":"表示资源/操作/实例的分割；","表示操作的分割；"*"表示任意资源/操作/实例。

1、单个资源单个权限

	#ini文件
	role1 = system:user:update
	#测试
	subject().checkPermissions("system:user:update"); 

2、单个资源多个权限

	#ini文件
	role41=system:user:update,system:user:delete
	#测试
	subject().checkPermissions("system:user:update", "system:user:delete");

3、单个资源全部权限

	#ini文件
	role52=system:user:* 

4、所有资源全部权限

	#ini文件
	role61=*:view
	#测试
	subject().checkPermissions("user:view"); 

5、实例级别的权限

《跟我我学Shiro》26页

#### 授权流程

![](http://i.imgur.com/S7tKDiy.png)

流程如下：
1、首先调用 Subject.isPermitted*/hasRole*接口，其会委托给 SecurityManager，而SecurityManager 接着会委托给 Authorizer；
2、Authorizer 是真正的授权者，如果我们调用如 isPermitted("user:view")，其首先会通过PermissionResolver 把字符串转换成相应的 Permission 实例；
3、在进行授权之前，其会调用相应的 Realm 获取 Subject 相应的角色/权限用于匹配传入的角色/权限；
4、Authorizer 会判断 Realm 的角色/权限是否和传入的匹配，如果有多个 Realm，会委托给ModularRealmAuthorizer 进行循环判断，如果匹配如 isPermitted*/hasRole*会返回 true，否则返回 false 表示授权失败。

#### 授权源码分析★★★★★

1、定义接口Authorizer接口，包括一些isPermitted、checkPermission、hasRole、checkRole一些方法

	public interface Authorizer {
	
	    boolean isPermitted(PrincipalCollection principals, String permission);
	
	    boolean isPermitted(PrincipalCollection subjectPrincipal, Permission permission);
	
	    boolean[] isPermitted(PrincipalCollection subjectPrincipal, String... permissions);
	
	    boolean[] isPermitted(PrincipalCollection subjectPrincipal, List<Permission> permissions);
	
	    boolean isPermittedAll(PrincipalCollection subjectPrincipal, String... permissions);
	
	    boolean isPermittedAll(PrincipalCollection subjectPrincipal, Collection<Permission> permissions);
	
	    void checkPermission(PrincipalCollection subjectPrincipal, String permission) throws AuthorizationException;
	
	    void checkPermission(PrincipalCollection subjectPrincipal, Permission permission) throws AuthorizationException;
	
	    void checkPermissions(PrincipalCollection subjectPrincipal, String... permissions) throws AuthorizationException;
	
	    void checkPermissions(PrincipalCollection subjectPrincipal, Collection<Permission> permissions) throws AuthorizationException;
	
	    boolean hasRole(PrincipalCollection subjectPrincipal, String roleIdentifier);
	
	    boolean[] hasRoles(PrincipalCollection subjectPrincipal, List<String> roleIdentifiers);
	
	    boolean hasAllRoles(PrincipalCollection subjectPrincipal, Collection<String> roleIdentifiers);
	
	    void checkRole(PrincipalCollection subjectPrincipal, String roleIdentifier) throws AuthorizationException;
	
	    void checkRoles(PrincipalCollection subjectPrincipal, Collection<String> roleIdentifiers) throws AuthorizationException;
	
	    void checkRoles(PrincipalCollection subjectPrincipal, String... roleIdentifiers) throws AuthorizationException;
	    
	}

2、同样，和认证一样有一个ModularRealmAuthorizer实现，委托给realm的方法去实现

public class ModularRealmAuthorizer implements Authorizer, PermissionResolverAware, RolePermissionResolverAware {

	<font color='red'>//保存realm的实例</font>
	protected Collection<Realm> realms;
	protected PermissionResolver permissionResolver;
	protected RolePermissionResolver rolePermissionResolver;


    public boolean hasRole(PrincipalCollection principals, String roleIdentifier) {
        assertRealmsConfigured();
        for (Realm realm : getRealms()) {
            if (!(realm instanceof Authorizer)) continue;
            if (((Authorizer) realm).hasRole(principals, roleIdentifier)) {
                return true;
            }
        }
        return false;
    }

}

### Realm及相关对象★★★★★

#### 用户、角色、权限的关系

![](http://i.imgur.com/0Nrv45E.png)

* 用户——角色之间是多对多关系，角色——权限之间是多对多关系，用户和权限之间通过角色建立关系
* 在系统中验证时通过权限验证，角色只是权限集合，即显示角色


#### realm接口及其子类源码分析

1、定义Realm接口，吃、从数据源中获得安全数据（如用户，角色，权限），判断用户身份是否合法，判断用户是否有权限

	public interface Realm {
	
	    //realm的名称
	    String getName();
	
	    //判断此 Realm 是否支持此 Token
	    boolean supports(AuthenticationToken token);
	
	    //传入用户凭证（用户名、密码），返回AuthenticationInfo
	    AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException;
	
	}

2、CachingRealm，增加缓存的功能：维护一个CacheManager的成员变量；增加一个name成员变量，实现getName()方法

	public abstract class CachingRealm implements Realm, Nameable, CacheManagerAware, LogoutAware {
	
		private CacheManager cacheManager;
	
	}

3、抽象类AuthenticatingRealm继承CachingRealm，主要功能是：1、从数据源中获取用户信息 2、利用CredentialsMatcher类，将用户传入的密码与数据库返回的密码进行比较（一般我们会自己创建service方法，判断用户名是否存在、密码是否正确，如果出现错误则抛出异常，验证通过返回AuthenticationInfo，此时Shiro会在帮你验证一次，其实没有必要，但框架这么写着你必须使用正确的CredentialsMatcher，使验证通过）；**抽象方法doGetAuthenticationInfo需要由子类重写**

<pre>
public abstract class AuthenticatingRealm extends CachingRealm implements Initializable {

	<font color='red'>//CredentialsMatcher用于密码校验</font>
	private CredentialsMatcher credentialsMatcher;

	public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
		

	    AuthenticationInfo info = getCachedAuthenticationInfo(token);
	    if (info == null) {
	        //otherwise not cached, perform the lookup:
			<font color='red'>//从数据源获得用户信息</font>
	        <font color='red'>info = doGetAuthenticationInfo(token);</font>
	        log.debug("Looked up AuthenticationInfo [{}] from doGetAuthenticationInfo", info);
	        if (token != null && info != null) {
	            cacheAuthenticationInfoIfPossible(token, info);
	        }
	    } else {
	        log.debug("Using cached authentication info [{}] to perform credentials matching.", info);
	    }
	
	    if (info != null) {
			<font color='red'>//在Shiro中验证密码</font>
	        <font color='red'>assertCredentialsMatch(token, info);</font>
	    } else {
	        log.debug("No AuthenticationInfo found for submitted AuthenticationToken [{}].  Returning null.", token);
	    }
	
	    return info;
	}

	<font color='red'>//从数据源获得用户信息由子类改写</font>
    protected abstract AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException;

	<font color='red'>//利用CredentialsMatcher验证密码是否正确，默认是SimpleCredentialsMatcher，调用其doCredentialsMatch方法，根据字节来判断密码是否一样</font>
    protected void assertCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) throws AuthenticationException {
       <font color='red'>CredentialsMatcher cm = getCredentialsMatcher();</font>
        if (cm != null) {
            if (<font color='red'>!cm.doCredentialsMatch(token, info)</font>) {
                //not successful - throw an exception to indicate this:
                String msg = "Submitted credentials for token [" + token + "] did not match the expected credentials.";
                throw new IncorrectCredentialsException(msg);
            }
        } else {
            throw new AuthenticationException("A CredentialsMatcher must be configured in order to verify " +
                    "credentials during authentication.  If you do not wish for credentials to be examined, you " +
                    "can configure an " + AllowAllCredentialsMatcher.class.getName() + " instance.");
        }
    }
	
}
</pre>

4、抽象类AuthorizingRealm，继承AuthenticatingRealm，主要功能是完成Authorizer委托的任务，完成isPermitted、hasRole等方法，所以实现了Authorizer接口；**doGetAuthorizationInfo方法需要由子类改写**

注意：
**成员变量PermissionResolver、RolePermissionResolver，由Authorizer传入**

<pre>
public abstract class AuthorizingRealm extends AuthenticatingRealm
    implements Authorizer, Initializable, PermissionResolverAware, RolePermissionResolverAware {

	//成员变量PermissionResolver、RolePermissionResolver，由Authorizer传入
	private PermissionResolver permissionResolver;
	private RolePermissionResolver permissionRoleResolver;

	<font color='red'>/*
	* 1、调用getAuthorizationInfo方法从数据库获得权限信息
	* 2、查看用户是否有该操作的权限
	*/</font>
	public boolean hasRole(PrincipalCollection principal, String roleIdentifier) {
        AuthorizationInfo info = getAuthorizationInfo(principal);
        return hasRole(roleIdentifier, info);
    }

    protected boolean hasRole(String roleIdentifier, AuthorizationInfo info) {
        return info != null && info.getRoles() != null && info.getRoles().contains(roleIdentifier);
    }

	protected abstract AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals);

	//权限：1、调用PermissionResolver，先把权限字符串转换为Permission
    public boolean isPermitted(PrincipalCollection principals, String permission) {
        Permission p = getPermissionResolver().resolvePermission(permission);
        return isPermitted(principals, p);
    }

	//2、判断权限
    public boolean isPermitted(PrincipalCollection principals, Permission permission) {
        AuthorizationInfo info = getAuthorizationInfo(principals);
        return isPermitted(permission, info);
    }

	//3、调用Permission的implies方法，将数据库的权限与用户传入的进行对比
    protected boolean isPermitted(Permission permission, AuthorizationInfo info) {
        Collection<Permission> perms = getPermissions(info);
        if (perms != null && !perms.isEmpty()) {
            for (Permission perm : perms) {
                if (perm.implies(permission)) {
                    return true;
                }
            }
        }
        return false;
    }
}
</pre>

#### realm相关对象

实际开发过程中，自定义的realm继承AuthorizingRealm，改写doGetAuthenticationInfo和doGetAuthorizationInfo方法

* doGetAuthenticationInfo 获取身份验证相关信息：1、调用Subject.login(token)触发方法，根据token获得User信息，没找到用户、密码不正确时抛出异常 2、生成AuthenticationInfo信息并返回 3、父类AuthenticatingRealm 使用 CredentialsMatcher 进行判断密码是否匹配， 如果不匹配将抛出密码错误异常，如果密码重试重试次数太多将抛出重试次数异常
* doGetAuthorizationInfo 获取授权信息：根据用户名获得角色及权限信息，并组装成AuthorizationInfo返回

**AuthenticationToken**

AuthenticationToken 用于收集用户提交的身份（如用户名）及凭据（如密码）

Shiro 提供了一个直接拿来用的 UsernamePasswordToken，用于实现用户名/密码 Token 组，
另外其实现了 RememberMeAuthenticationToken 和 HostAuthenticationToken， 可以实现记住
我及主机验证的支持。

**AuthenticationInfo**

AuthenticationInfo 有两个作用：
1、如果 Realm 是 AuthenticatingRealm 子类，则提供给 AuthenticatingRealm 内部使用的
CredentialsMatcher 进行凭据验证； （如果没有继承它需要在自己的 Realm 中自己实现验证）；
2、提供给 SecurityManager 来创建 Subject（提供身份信息）；

**AuthorizationInfo**

AuthorizationInfo 用于聚合授权信息的

当我们使用 AuthorizingRealm 时 ， 如果身份验证成功 ， 在进行授权时就通过
doGetAuthorizationInfo 方法获取角色/权限信息用于授权验证

**Subject**

Subject 是 Shiro 的核心对象，基本所有身份验证、授权都是通过 Subject 完成。


### SecurityManager源码分析★★★★★

1、定义SecurityManager接口，同时继承Authenticator、Authorizer、SessionManager，为什么？**代理模式，SecurityManager代理Authenticator、Authorize和SessionManager的操作，需要实现三者的接口**

	public interface SecurityManager extends Authenticator, Authorizer, SessionManager {
	
	    Subject login(Subject subject, AuthenticationToken authenticationToken) throws AuthenticationException;
	
	    void logout(Subject subject);
	
	    Subject createSubject(SubjectContext context);
	
	}

![](http://i.imgur.com/NMGskjt.png)

2、CachingSecurityManager维护一个CacheManager成员变量

	public abstract class CachingSecurityManager implements SecurityManager, Destroyable, CacheManagerAware, EventBusAware {
	
	    private CacheManager cacheManager;
	
	}
3、RealmSecurityManager，设置realm

	public abstract class RealmSecurityManager extends CachingSecurityManager {
	
		private Collection<Realm> realms;
	
	    public void setRealm(Realm realm) {
	        if (realm == null) {
	            throw new IllegalArgumentException("Realm argument cannot be null");
	        }
	        Collection<Realm> realms = new ArrayList<Realm>(1);
	        realms.add(realm);
	        setRealms(realms);
	    }
	}

4、AuthenticatingSecurityManager保存一个Authenticator，默认是ModularRealmAuthenticator，代理认证的功能

	public abstract class AuthenticatingSecurityManager extends RealmSecurityManager {
	
		private Authenticator authenticator;

	    public AuthenticatingSecurityManager() {
	        super();
	        this.authenticator = new ModularRealmAuthenticator();
	    }
		
	}

5、AuthorizingSecurityManager保存一个Authorizer功能，默认是ModularRealmAuthorizer，代理授权功能

	public abstract class AuthorizingSecurityManager extends AuthenticatingSecurityManager {
	
		private Authorizer authorizer;
	
		public AuthorizingSecurityManager() {
	        super();
	        this.authorizer = new ModularRealmAuthorizer();
	    }
	
	}

6、SessionsSecurityManager保存一个SessionManager，默认是DefaultSessionManager，代理会话管理

	public abstract class SessionsSecurityManager extends AuthorizingSecurityManager {
	
		private SessionManager sessionManager;
	
		public SessionsSecurityManager() {
	        super();
	        this.sessionManager = new DefaultSessionManager();
	        applyCacheManagerToSessionManager();
	    }
	}

7、DefaultSecurityManager：利用SubjectDAO、SubjectFactory创建、存储Subject

	public class DefaultSecurityManager extends SessionsSecurityManager {
	
	    protected RememberMeManager rememberMeManager;
	    protected SubjectDAO subjectDAO;
	    protected SubjectFactory subjectFactory;
	
		public DefaultSecurityManager() {
	        super();
	        this.subjectFactory = new DefaultSubjectFactory();
	        this.subjectDAO = new DefaultSubjectDAO();
	    }
	
	    protected SubjectContext createSubjectContext() {
	        return new DefaultSubjectContext();
	    }
	
	    public Subject createSubject(SubjectContext subjectContext) {
	        SubjectContext context = copy(subjectContext);
	        context = ensureSecurityManager(context);
	        context = resolveSession(context);
	        context = resolvePrincipals(context);
	        Subject subject = doCreateSubject(context);
	        save(subject);
	        return subject;
	    }
	
	}

8、DefaultWebSecurityManager：增加了一些Session的维护

### 加密

AuthenticatingRealm中的成员变量CredentialsMatcher进行密码验证服务，Shiro 提供了 redentialsMatcher 的散列实现 HashedCredentialsMatcher，它用于密码验证，且可以提供自己的盐

realm认证

    //判断认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        //返回AuthenticationInfo后在AuthenticatingRealm的assertCredentialsMatch方法中还需要进行验证
        //数据库取出的用户名和密码
        String username = "admin";
        String password = "b433ce675b32a824e24d762ca0fa1ba9";//数据库密码md5("admin","user")
        String salt = "user";

        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(username, password, getName());
        simpleAuthenticationInfo.setCredentialsSalt(ByteSource.Util.bytes(salt));
        return simpleAuthenticationInfo;
    }

设置密码匹配规则

	staticRealm = StaticRealm
	hashedCredentialsMatcher = org.apache.shiro.authc.credential.HashedCredentialsMatcher
	hashedCredentialsMatcher.hashAlgorithmName = md5
	staticRealm.credentialsMatcher = $hashedCredentialsMatcher
	securityManager.realms = $staticRealm

### Web集成

#### ShiroFilter入口

**与 Spring 集成**

	<filter> 
		<filter-name>shiroFilter</filter-name> 
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class> 
		<init-param> 
			<param-name>targetFilterLifecycle</param-name> 
			<param-value>true</param-value> 
		</init-param> 
	</filter> 
	<filter-mapping> 
		<filter-name>shiroFilter</filter-name> 
		<url-pattern>/*</url-pattern> 
	</filter-mapping>

DelegatingFilterProxy 作用是自动到 spring 容器查找名字为 shiroFilter（filter-name）的 bean，并把所有 Filter 的操作委托给它。然后将 ShiroFilter 配置到 spring 容器即可

	<bean id="shiroFilter"class="org.apache.shiro.spring.web.ShiroFilterFactoryBean"> 
		<propertyname="securityManager" ref="securityManager"/> 
		<!—忽略其他，详见与 Spring 集成部分  --> 
	</bean> 

最后不要忘了使用 org.springframework.web.context.ContextLoaderListener 加载这个 spring
配置文件即可

#### URL匹配

	[urls] 
	/login=anon 
	/unauthorized=anon 
	/static/**=anon 
	/authenticated=authc 
	/role=authc,roles[admin] 
	/permission=authc,perms["user:create"] 

urls中配置格式是：url=拦截器[参数]，拦截器[参数]，即 **如果当前请求的 url 匹配[urls]部分的某个 url 模式， 将会执行其配置的拦截器**

* anon 拦截器表示匿名访问（即不需要登录即可访问）
* authc 拦截器表示需要身份认证通过后才能访问
* roles[admin] 拦截器表示需要有 admin 角色授权才能访问
* perms["user:create"] 拦截器表示需要有“user:create”权限才能访问

url 模式使用 Ant 风格模式，Ant 路径通配符支持 `?` 、`*` 、`**`，注意通配符匹配不包括目录分隔符"/"：

* `?`：匹配一个字符，如"/admin?"将匹配/admin1，但不匹配/admin 或/admin2；
* `*`：匹配零个或多个字符串，如/admin*将匹配/admin、/admin123，但不匹配/admin/1；
* `**`：匹配路径中的零个或多个路径，如/admin/**将匹配/admin/a 或/admin/a/b。

**url 模式匹配顺序**

url 模式匹配顺序是按照在配置中的声明顺序匹配，即从头开始使用第一个匹配的 url 模式对应的拦截器链


### 拦截器机制

#### Shiro拦截器源码分析

Shiro拦截器的基础类图

![](http://i.imgur.com/Vkfw4UF.png)

* AbstractFilter：实现Filter接口，**类的主要功能是实现Filter接口init方法，保存FilterConfig作为成员变量，在init方法中调用无参的setFilterConfig，<font color='red'>子类的初始化方法通过改写onFilterConfigSet方法</font>**
* NameableFilter：给Filter起个名字。在类中增加一个name成员变量，set/get方法
* OncePerRequestFilter：一次请求中，保证过滤链中同一个过滤器只被执行一次，如内部的 forward 不会再多执行一次 doFilterInternal。**重写Filter接口的doFilter方法，同时设置了enabled属性，表示是否开启该拦截器，<font color='red'>子类需要重写doFilterInternal方法，加上具体的过滤操作</font>**

<pre>
public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)  
        throws ServletException, IOException {  
    String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();  
    if (request.getAttribute(alreadyFilteredAttributeName) != null || shouldNotFilter(request)) {  
        log.trace("Filter '{}' already executed.  Proceeding without invoking this filter.", getName());  
        // Proceed without invoking this filter...  
        filterChain.doFilter(request, response);  
    } else {  
        // Do invoke this filter...  
        log.trace("Filter '{}' not yet executed.  Executing now.", getName());  
        request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);  

        try {
			<font color='red'>//抽象方法，由子类实现</font>  
            doFilterInternal(request, response, filterChain);  
        } finally {  
            // Once the request has finished, we're done and we don't  
            // need to mark as 'already filtered' any more.  
            request.removeAttribute(alreadyFilteredAttributeName);  
        }  
    }  
}  
</pre>

* AbstractShiroFilter：Shiro过滤器前最后一个抽象类，有两个成员变量：WebSecurityManager（存储SecurityManager的引用，以便过滤器使用）、FilterChainResolver（首先拦截所有的url，再根据实际情况判断url是否需要拦截）；重写onFilterConfigSet方法，**<font color='red'>子类的初始化方法通过重写init方法完成初始化工作</font>**;**重写doFilterInternal方法，为subject的入口，重点★★★★★**


	//重写onFilterConfigSet方法，子类重写init方法增加自定义初始化工作
    protected final void onFilterConfigSet() throws Exception {
        //added in 1.2 for SHIRO-287:
        applyStaticSecurityManagerEnabledConfig();
        init();
        ensureSecurityManager();
        //added in 1.2 for SHIRO-287:
        if (isStaticSecurityManagerEnabled()) {
            SecurityUtils.setSecurityManager(getSecurityManager());
        }
    }

* ShiroFilter、SpringShiroFilter、SpringShiroFilter 实现 AbstractShiroFilter，主要功能是创建过滤器对象，并将SecurityManager和FilterChainResolver属性的注入


**另一条过滤器主线：与SpringMVC中的拦截器类似进行设计**

* AdviceFilter

AdviceFilter 提供了 AOP 风格的支持，类似于 SpringMVC 中的 Interceptor：

1、preHandler：类似于 AOP 中的前置增强；在拦截器链执行之前执行；如果返回 true 则继续
拦截器链；否则中断后续的拦截器链的执行直接返回；进行预处理（如基于表单的身份验
证、授权）
2、postHandle： 类似于 AOP 中的后置返回增强； 在拦截器链执行完成后执行； 进行后处理 （如
记录执行时间之类的）；
3、afterCompletion：类似于 AOP 中的后置最终增强；即不管有没有异常都会执行；可以进行
清理资源（如接触 Subject 与线程的绑定之类的）；

<pre>
//重写doFilterInternal方法
public void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)
        throws ServletException, IOException {

    Exception exception = null;

    try {
		<font color='red'>//前置方法，如果前置返回false，则不执行下一个过滤器</font>
        boolean continueChain = preHandle(request, response);
        if (log.isTraceEnabled()) {
            log.trace("Invoked preHandle method.  Continuing chain?: [" + continueChain + "]");
        }

        if (continueChain) {
            executeChain(request, response, chain);
        }
		<font color='red'>//后置方法，访问Servlet后执行</font>
        postHandle(request, response);
        if (log.isTraceEnabled()) {
            log.trace("Successfully invoked postHandle method");
        }

    } catch (Exception e) {
        exception = e;
    } finally {
		<font color='red'>//afterCompletion，不管有没有异常都会执行，可以进行清理资源</font>
        cleanup(request, response, exception);
    }
}
</pre>

* PathMatchingFilter：提供了基于 Ant 风格的请求路径匹配功能及拦截器参数解析的功能，如
"roles[admin,user]"自动根据"，"分割解析到一个路径参数配置并绑定到相应的路径

1、pathsMatch该方法用于 path 与请求路径进行匹配的方法；如果匹配返回 true；
2、onPreHandle：在 preHandle 中，当 pathsMatch 匹配一个路径后，会调用 opPreHandler 方法
并将路径绑定参数配置传给 mappedValue；然后可以在这个方法中进行一些验证（如角色
授权），如果验证失败可以返回 false 中断流程；默认返回 true；**也就是说子类可以只实现
onPreHandle 即可**， 无须实现 preHandle。 如果没有 path 与请求路径匹配， 默认是通过的 （即
preHandle 返回 true）。

<pre>
protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {

    if (this.appliedPaths == null || this.appliedPaths.isEmpty()) {
        if (log.isTraceEnabled()) {
            log.trace("appliedPaths property is null or empty.  This Filter will passthrough immediately.");
        }
        return true;
    }

    <font color='red'>for (String path : this.appliedPaths.keySet()) {
        // If the path does match, then pass on to the subclass implementation for specific checks
        //(first match 'wins'):
        if (pathsMatch(path, request)) {
            log.trace("Current requestURI matches pattern '{}'.  Determining filter chain execution...", path);
            Object config = this.appliedPaths.get(path);
            return isFilterChainContinued(request, response, path, config);
        }
    }</font>

    //no path matched, allow the request to go through:
    return true;
}

private boolean isFilterChainContinued(ServletRequest request, ServletResponse response,
                                       String path, Object pathConfig) throws Exception {

    if (isEnabled(request, response, path, pathConfig)) { //isEnabled check added in 1.2
        if (log.isTraceEnabled()) {
            log.trace("Filter '{}' is enabled for the current request under path '{}' with config [{}].  " +
                    "Delegating to subclass implementation for 'onPreHandle' check.",
                    new Object[]{getName(), path, pathConfig});
        }
        //The filter is enabled for this specific request, so delegate to subclass implementations
        //so they can decide if the request should continue through the chain or not:
        <font color='red'>return onPreHandle(request, response, pathConfig);</font>
    }

    if (log.isTraceEnabled()) {
        log.trace("Filter '{}' is disabled for the current request under path '{}' with config [{}].  " +
                "The next element in the FilterChain will be called immediately.",
                new Object[]{getName(), path, pathConfig});
    }
    //This filter is disabled for this specific request,
    //return 'true' immediately to indicate that the filter will not process the request
    //and let the request/response to continue through the filter chain:
    return true;
}
</pre>

* AnonymousFilter：继承PathMatchingFilter，重写preHandle方法，总是返回true，则允许所有请求通过

* AccessControlFilter：继承PathMatchingFilter，重写onPreHandle方法


	public boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return isAccessAllowed(request, response, mappedValue) || onAccessDenied(request, response, mappedValue);
    }


1、isAccessAllowed：表示是否允许访问；mappedValue 就是[urls]配置中拦截器参数部分，如果允许访问返回 true，否则 false；
2、onAccessDenied：表示当访问拒绝时是否已经处理了；如果返回 true 表示需要继续处理；如果返回 false 表示该拦截器实例已经处理了，将直接返回即可

**总结：**

1、如果用户有访问资源的权限，则isAccessAllowed返回true，onAccessDenied方法不会执行；
2、如果没有访问权限，则isAccessAllowed返回false，而且会执行onAccessDenied方法，方法中可以进行页面跳转等操作，方法体一般返回false，表示不需要执行后面的过滤器了

**两个方法为抽象方法，都需要都子类重写**

另外 AccessControlFilter 还提供了如下方法用于处理如登录成功后/重定向到上一个请求：

	void setLoginUrl(String loginUrl) //身份验证时使用，默认/login.jsp 
	String getLoginUrl() 
	Subject getSubject(ServletRequest request, ServletResponse response) //获取 Subject 实例
	boolean  isLoginRequest(ServletRequest  request,  ServletResponse  response)//当前请求是否是登录请求
	void  saveRequestAndRedirectToLogin(ServletRequest  request,  ServletResponse  response) 
	throws IOException //将当前请求保存起来并重定向到登录页面
	void saveRequest(ServletRequest request) //将请求保存起来，如登录成功后再重定向回该请
	求
	void redirectToLogin(ServletRequest request, ServletResponse response) //重定向到登录页面

* AuthenticationFilter继承AccessControlFilter，并重写isAccessAllowed方法，判断用户是否登录


	protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {  
	        Subject subject = getSubject(request, response);  
	        return subject.isAuthenticated();  
	    } 

#### Shiro拦截器源码分析总结

1、过滤器设计，顶层是接口，然后一个抽象类实现接口，每个抽象类中值承担一个职责！

* AbstractFilter保存filterConfig，并提供一个无参的onFilterConfigSet初始化方法
* NameableFilter为过滤器取一个名字
* OncePerRequestFilter重写doFilter方法，保证一个过滤器在一次请求中只执行一次，过滤器中个操作转移到doFilterInternal中。
* **之后过滤器子类分为两个分支**，一个是Shiro的主过滤器，配置在web.xml中，为Shiro的入口；另一个是AOP风格的过滤器形式，类时Spring MVC中的拦截器，进入ShiroFilter中先执行完Shiro的一些过滤器，在执行Tomcat中的其他过滤器
* ★★★★★AdviceFilter重写doFilterInternal方法，将过滤操作分为preHandle、postHandle、afterCompletion操作。如果preHandle返回true，执行下一个过滤器，否则访问终止
* PathMatchingFilter重写preHandle方法，进行路径匹配，路径匹配后是否执行Tomcat其他过滤器由onPreHandle的返回结果决定，供子类改写
* AccessControlFilter重写onPreHandle方法，可以重写isAccessAllowed、onAccessDenied来实现访问控制


**开发建议：**
**如果我们想进行访问访问的控制就可以继承AccessControlFilter；如果我们要添加一些通用数据我们可以直接继承 PathMatchingFilter**

#### 拦截器链

Shiro 对 Servlet 容器的 FilterChain 进行了代理，即 ShiroFilter 在继续 Servlet 容器的 Filter 链的执行之前，通过 ProxiedFilterChain 对 Servlet 容器的 FilterChain 进行了代理；**即先走Shiro 自己的 Filter 体系，然后才会委托给 Servlet 容器的 FilterChain 进行 Servlet 容器级别的 Filter 链执行**； 

Shiro 的 ProxiedFilterChain 执行流程： 
1、 执行 Shiro 自己的 Filter 链； 2、再执行 Servlet 容器的 Filter 链（即原始的 Filter）

**★★★★★如何实现先执行Shiro自己的Filter链，然后执行Servlet容器的Filter链？**

1、调用Shiro的主过滤器后，执行AbstractShiroFilter类的doFilterInternal方法

<pre>
protected void doFilterInternal(ServletRequest servletRequest, ServletResponse servletResponse, final FilterChain chain)
            throws ServletException, IOException {
	...
	subject.execute(new Callable() {
	                public Object call() throws Exception {
	                    updateSessionLastAccessTime(request, response);
	                    <font color='red'>executeChain(request, response, chain);</font>
	                    return null;
	                }
	            });
	...
}
</pre>

2、AbstractShiroFilter类的executeChain方法，会返回一个Servlet过滤链的代理类ProxiedFilterChain，当执行doFilter方法时，如果Shiro内部的过滤器没有执行完，先执行内部过滤器，等全部执行完成后，在执行Servlet的下一个过滤器。代理Chain对Servlet是透明的

    protected void executeChain(ServletRequest request, ServletResponse response, FilterChain origChain)
            throws IOException, ServletException {
        FilterChain chain = getExecutionChain(request, response, origChain);
        chain.doFilter(request, response);
    }

代理过滤器链ProxiedFilterChain的代码如下，包含原来的过滤链和所需要执行的Shiro过滤器；如果Shiro过滤器没有执行完，则首先执行Shiro过滤器，之后再执行 Servlet 过滤器

<pre>
public class ProxiedFilterChain implements FilterChain {

    //TODO - complete JavaDoc

    private static final Logger log = LoggerFactory.getLogger(ProxiedFilterChain.class);

	//原来的过滤链对象
    <font color='red'>private FilterChain orig;
    private List<Filter> filters;</font>
    private int index = 0;

    public ProxiedFilterChain(FilterChain orig, List<Filter> filters) {
        if (orig == null) {
            throw new NullPointerException("original FilterChain cannot be null.");
        }
        this.orig = orig;
        this.filters = filters;
        this.index = 0;
    }

    public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
        //如果执行完Shiro的权限过滤器，则继续执行Servlet中的过滤器
		if (this.filters == null || this.filters.size() == this.index) {
            //we've reached the end of the wrapped chain, so invoke the original one:
            if (log.isTraceEnabled()) {
                log.trace("Invoking original filter chain.");
            }
            <font color='red'>this.orig.doFilter(request, response);</font>
        } else {
            if (log.isTraceEnabled()) {
                log.trace("Invoking wrapped filter at index [" + this.index + "]");
            }
            <font color='red'>this.filters.get(this.index++).doFilter(request, response, this);</font>
        }
    }
}
</pre>

**代理过滤链的产生流程**

1、AbstractShiroFilter中的getExecutionChain方法，通过FilterChainResolver的getChain方法，获得过滤链

<pre>
protected FilterChain getExecutionChain(ServletRequest request, ServletResponse response, FilterChain origChain) {
    FilterChain chain = origChain;

    <font color='red'>FilterChainResolver resolver = getFilterChainResolver();</font>
    if (resolver == null) {
        log.debug("No FilterChainResolver configured.  Returning original FilterChain.");
        return origChain;
    }

    <font color='red'>FilterChain resolved = resolver.getChain(request, response, origChain);</font>
    if (resolved != null) {
        log.trace("Resolved a configured FilterChain for the current request.");
        chain = resolved;
    } else {
        log.trace("No FilterChain configured for the current request.  Using the default.");
    }

    return chain;
}
</pre>

2、FilterChainResolver接口的默认实现是PathMatchingFilterChainResolver，必须实现getChain方法。DefaultFilterChainManager 实现 FilterChainManager 接口，维护着 url 模式与拦截器链的关系，遍历url模式，如果与访问的url匹配，则根据匹配的 url 模式找到相应的过滤器，用来生成过滤器链；

<pre>
public FilterChain getChain(ServletRequest request, ServletResponse response, FilterChain originalChain) {
    FilterChainManager filterChainManager = getFilterChainManager();
    if (!filterChainManager.hasChains()) {
        return null;
    }

    String requestURI = getPathWithinApplication(request);

    for (String pathPattern : filterChainManager.getChainNames()) {

        if (pathMatches(pathPattern, requestURI)) {
            if (log.isTraceEnabled()) {
                log.trace("Matched path pattern [" + pathPattern + "] for requestURI [" + requestURI + "].  " +
                        "Utilizing corresponding filter chain...");
            }
            return filterChainManager.proxy(originalChain, pathPattern);
        }
    }

    return null;
}
</pre>

**PathMatchingFilterChainResolver维护着FilterChainManager和PatternMatcher两个成员变量**，FilterChainManager维护着url 模式与拦截器链的关系，默认是DefaultFilterChainManager；PatternMatcher为请求url与url 模式的匹配方式，默认是AntPathMatcher（PathMatchingFilterChainResolver中构造器时赋值）


**Shiro Filter初始化时，ShiroFilterFactoryBean调用getObject时，createInstance中创建了一些重要的组件，包括SecurityManager、FilterChainResolver（FilterChainManager为其成员变量）。**
<pre>
protected AbstractShiroFilter createInstance() throws Exception {

    log.debug("Creating Shiro Filter instance.");

    SecurityManager securityManager = getSecurityManager();
    if (securityManager == null) {
        String msg = "SecurityManager property must be set.";
        throw new BeanInitializationException(msg);
    }

    if (!(securityManager instanceof WebSecurityManager)) {
        String msg = "The security manager does not implement the WebSecurityManager interface.";
        throw new BeanInitializationException(msg);
    }

	//创建DefaultFilterChainManager
    <font color='red'>FilterChainManager manager = createFilterChainManager();</font>

    //Expose the constructed FilterChainManager by first wrapping it in a
    // FilterChainResolver implementation. The AbstractShiroFilter implementations
    // do not know about FilterChainManagers - only resolvers:
    PathMatchingFilterChainResolver chainResolver = new PathMatchingFilterChainResolver();
    <font color='red'>chainResolver.setFilterChainManager(manager);</font>

    //Now create a concrete ShiroFilter instance and apply the acquired SecurityManager and built
    //FilterChainResolver.  It doesn't matter that the instance is an anonymous inner class
    //here - we're just using it because it is a concrete AbstractShiroFilter instance that accepts
    //injection of the SecurityManager and FilterChainResolver:
    return new SpringShiroFilter((WebSecurityManager) securityManager, chainResolver);
}
</pre>

#### 拦截器链总结

1、当用户发送请求时，被Shiro Filter拦截，利用FilterChainResolver的getChain方法，获得代理的过滤链，这样可以先执行Shiro 中的一些过滤器，再执行Servlet的其他过滤器
2、获得过滤链代理的主要流程是：FilterChainResolver的实现类PathMatchingFilterChainResolver维护着FilterChainManager和PatternMatcher两个成员变量，FilterChainManager维护着url 模式与拦截器链的关系，PatternMatcher为请求url与url 模式的匹配方式，遍历url模式并与请求url进行匹配，获取配置的过滤器，组成代理的过滤器链，并返回

#### 默认拦截器

	public enum DefaultFilter {
	
	    anon(AnonymousFilter.class),
	    authc(FormAuthenticationFilter.class),
	    authcBasic(BasicHttpAuthenticationFilter.class),
	    logout(LogoutFilter.class),
	    noSessionCreation(NoSessionCreationFilter.class),
	    perms(PermissionsAuthorizationFilter.class),
	    port(PortFilter.class),
	    rest(HttpMethodPermissionFilter.class),
	    roles(RolesAuthorizationFilter.class),
	    ssl(SslFilter.class),
	    user(UserFilter.class);
		...
	}

![](http://i.imgur.com/rde5CK6.png)

#### 自己提出的问题★★★★★

**1、**Shiro Filter中SecurityManager是否需要绑定到内存的问题

详见AbstractShiroFilter.java

* 如果在Shiro过滤器初始化参数中设置了staticSecurityManagerEnabled的值为true，则SecurityManager与线程绑定
* 默认是false，即SecurityManager不与线程绑定，设置在WebEnvironment中

**2、**Shiro Filter中SecurityManager变量依赖注入的方式

Shiro Filter中成员变量SecurityManager实例化的方式有两种：
* 第一种方式是 SecurityManager 在 Spring 中创建，可以自定义设置SecurityManager的属性，然后依赖注入到Shiro Filter中（单例）
* 如果没有通过依赖注入的方式得到SecurityManager的实例，Shiro将自动创建默认的SecurityManager（每次调用过滤器都会创建一个默认SecurityManager，不推荐）

详见AbstractShiroFilter.java

**3、**★★★★在Web.xml中配置

	<filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <!-- 该值缺省为false,表示生命周期由SpringApplicationContext管理,设置为true则表示由ServletContainer管理 -->
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <!--使用[/*]匹配所有请求,保证所有的可控请求都经过Shiro的过滤-->
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

在applicationContext-shiro.xml中配置

	<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	...
	</bean>

则DelegatingFilterProxy类与ShiroFilterFactoryBean的关系是什么？

答：DelegatingFilterProxy是一个代理类，具体的操作交给内部的Filter对象delegate去处理，这个delegate通过Spring容器的中的ShiroFilterFactoryBean的工厂方法getBean获取，返回对象类型是SpringShiroFilter，即代理的Filter对象为SpringShiroFilter

处理流程：
1、Tomcat扫描web.xml文件，到过滤器节点，遇到DelegatingFilterProxy的配置，生成一个DelegatingFilterProxy的实例，并调用其init方法

	<filter>   
	    <filter-name>shiroFilter</filter-name>  
	    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
	    <init-param>  
	         <param-name>targetFilterLifecycle</param-name>   
	         <param-value>true</param-value>   
	    </init-param>   
	</filter> 

2、init方法在其父类GenericFilterBean中实现，init方法中调用initFilterBean方法，其在子类DelegatingFilterProxy中改写，代理对象通过语句 this.delegate = initDelegate(wac) 获得
3、查看initDelegate方法，有如下语句：Filter delegate = wac.getBean(getTargetBeanName(), Filter.class)，代理对象是从Spring容器中根据id获得，id即为过滤器的名称shiroFilter，你就知道为什么web.xml中过滤器的名称和application-shiro.xml中的bean id要一致了

	<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	...
	</bean>

4、在Spring容器中查找id为shiroFilter，类型为Filter的对象，即为ShiroFilterFactoryBean。但ShiroFilterFactoryBean实现了工厂类，上面的delegate真正的对象是通过它的getObject()获取的

	public Object getObject() throws Exception {
        if (instance == null) {
            instance = createInstance();
        }
        return instance;
    }

5、真正创建对象的方法在createInstance中，**可见代理的过滤器是SpringShiroFilter**

	protected AbstractShiroFilter createInstance() throws Exception {
		...
		return new SpringShiroFilter((WebSecurityManager) securityManager, chainResolver);
	}

6、SpringShiroFilter: ShiroFilterFactoryBean的内部类，继承AbstractShiroFilter

    private static final class SpringShiroFilter extends AbstractShiroFilter {

        protected SpringShiroFilter(WebSecurityManager webSecurityManager, FilterChainResolver resolver) {
            super();
            if (webSecurityManager == null) {
                throw new IllegalArgumentException("WebSecurityManager property cannot be null.");
            }
            setSecurityManager(webSecurityManager);
            if (resolver != null) {
                setFilterChainResolver(resolver);
            }
        }
    }

7、每次URL请求，经过代理过滤器DelegatingFilterProxy时，调用SpringShiroFilter实例的doFilter方法

DelegatingFilterProxy.java

<pre>
public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
		throws ServletException, IOException {

	// Lazily initialize the delegate if necessary.
	<font color='red'>Filter delegateToUse = this.delegate;</font>
	if (delegateToUse == null) {
		synchronized (this.delegateMonitor) {
			if (this.delegate == null) {
				WebApplicationContext wac = findWebApplicationContext();
				if (wac == null) {
					throw new IllegalStateException("No WebApplicationContext found: no ContextLoaderListener registered?");
				}
				this.delegate = initDelegate(wac);
			}
			delegateToUse = this.delegate;
		}
	}

	// Let the delegate perform the actual doFilter operation.
	<font color='red'>invokeDelegate(delegateToUse, request, response, filterChain);</font>
}
</pre>	

**总结：**

1、Tomcat创建代理对象，初始化代理对象的时候，代理对象通过bean工厂获得被代理对象，并做成其成员对象；每当来一个客户端请求，代理对象的doFilter方法，代理对象调用被代理对象的doFilter方法

2、DelegatingFilterProxy的主要作用就是一个代理模式的应用,可以把 servlet 容器中的filter同spring容器中的bean关联起来[既可以代理Spring Security的过滤器，也可以代理Shiro中的过滤器]


**4、** 路径匹配规则

1、路径为空，不进行匹配默认全部通过
2、请求url与资源权限路径列表从上到下进行匹配，如果某一个路径匹配，进行权限验证，根据返回结果通过还是拒绝

### 会话管理

#### 会话

所谓会话，即用户访问应用时保持的连接关系，**在多次交互中应用能够识别出当前访问的用户是谁**，且可以在多次交互中保存一些数据。如访问一些网站时登录成功后，网站可以记住用户，且在退出之前都可以识别当前用户是谁。

	public interface Session {
	
	    /**
		 * 获取当前会话的唯一标识。
	     */
	    Serializable getId();
	
	    /* 获得属性
	     */
	    Object getAttribute(Object key) throws InvalidSessionException;
	
	    /* 设置属性
	     */
	    void setAttribute(Object key, Object value) throws InvalidSessionException;
	
	    /* 删除属性
	     */
	    Object removeAttribute(Object key) throws InvalidSessionException;
	
		...
	}


#### 会话存储/持久化★★★★★

Shiro 提供 SessionDAO 用于会话的 CRUD，即 DAO（Data Access Object）模式实现：

	//如 DefaultSessionManager 在创建完 session 后会调用该方法；如保存到关系数据库/文件
	系统/NoSQL 数据库；即可以实现会话的持久化；返回会话 ID ；主要此处返回的
	ID.equals(session.getId())；
	Serializable create(Session session); 
	//根据会话 ID 获取会话
	Session readSession(Serializable sessionId) throws UnknownSessionException; 
	//更新会话；如更新会话最后访问时间/停止会话/设置超时时间/设置移除属性等会调用
	void update(Session session) throws UnknownSessionException; 
	//删除会话；当会话过期/会话停止（如用户退出时）会调用
	void delete(Session session); 
	//获取当前所有活跃用户，如果用户量多此方法影响性能

Shiro 内嵌了如下 SessionDAO 实现：

![](http://i.imgur.com/GCOP4Bg.png)

AbstractSessionDAO 提供了 SessionDAO 的基础实现， 如生成会话 ID 等； CachingSessionDAO
提供了对开发者透明的会话缓存的功能，只需要设置相应的 CacheManager 即可；MemorySessionDAO 直接在内存中进行会话维护； 而 EnterpriseCacheSessionDAO 提供了缓存功能的会话维护，**默认情况下使用 MapCache 实现，内部使用 ConcurrentHashMap 保存缓存的会话**。

可以通过如下配置设置 SessionDAO：

	sessionDAO=org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO 
	sessionManager.sessionDAO=$sessionDAO 

Shiro 提供了使用 Ehcache 进行会话存储，Ehcache 可以配合 TerraCotta 实现容器无关的分
布式集群——98页

#### 会话管理器

* 会话管理器管理着应用中所有 Subject 的会话的创建、维护、删除、失效、验证等工作。是Shiro 的核心组件
* 顶层组件 SecurityManager 直接继承了 SessionManager，且提供了SessionsSecurityManager 实 现直接把会话管理**委托**给相应的 SessionManager 


	public abstract class SessionsSecurityManager extends AuthorizingSecurityManager {
	    private SessionManager sessionManager;
		public Session start(SessionContext context) throws AuthorizationException {
       		return this.sessionManager.start(context);
    	}

	    public Session getSession(SessionKey key) throws SessionException {
	        return this.sessionManager.getSession(key);
	    }
		...
	}


![](http://i.imgur.com/WT5FGEk.png)

Shiro 提供了三个默认实现：
* DefaultSessionManager：DefaultSecurityManager 使用的默认实现，用于 JavaSE 环境；
* ServletContainerSessionManager： DefaultWebSecurityManager 使用的默认实现，用于 Web环境，其直接使用 Servlet 容器的会话；
* <font color='red'>**DefaultWebSessionManager**</font>： 用于 Web 环境的实现 ， 可以替代ServletContainerSessionManager，自己维护着会话，直接废弃了 Servlet 容器的会话管理。

**替换默认的会话管理器**

**在 Servlet 容器中，默认使用 JSESSIONID Cookie 维护会话**，且会话默认是跟容器绑定的；在某些情况下可能需要使用自己的会话机制，此时我们可以使用 DefaultWebSessionManager来维护会话：

	#sessionIdCookie 是 sessionManager 创建会话 Cookie 的模板
	sessionIdCookie=org.apache.shiro.web.servlet.SimpleCookie 
	sessionManager=org.apache.shiro.web.session.mgt.DefaultWebSessionManager 
	#sessionIdCookie.name：设置 Cookie 名字，默认为 JSESSIONID
	sessionIdCookie.name=sid 
	#sessionIdCookie.domain：设置 Cookie 的域名，默认空，即当前访问的域名
	#sessionIdCookie.domain=sishuok.com 
	#sessionIdCookie.path：设置 Cookie 的路径，默认空，即存储在域名根下
	#sessionIdCookie.path= 
	#sessionIdCookie.maxAge：设置 Cookie 的过期时间，秒为单位，默认-1 表示关闭浏览器时过期 Cookie
	sessionIdCookie.maxAge=1800 
	#sessionIdCookie.httpOnly如果设置为 true，则客户端不会暴露给客户端脚本代码，使用HttpOnlycookie 有助于减少某些类型的跨站点脚本攻击； 此特性需要实现了 Servlet2.5 MR6及以上版本的规范的 Servlet 容器支持
	sessionIdCookie.httpOnly=true 
	sessionManager.sessionIdCookie=$sessionIdCookie 
	#sessionManager.sessionIdCookieEnabled：是否启用/禁用 Session Id Cookie，默认是启用的；如果禁用后将不会设置 Session Id Cookie， 即默认使用了 Servlet 容器的 JSESSIONID， 且通过 URL 重写（URL 中的“;JSESSIONID=id”部分）保存 Session Id。
	sessionManager.sessionIdCookieEnabled=true 
	securityManager.sessionManager=$sessionManager 

#### 会话监听器

会话监听器用于监听会话创建、过期及停止事件

#### 会话验证

Shiro 提供了会话验证调度器，用于定期的验证会话是否已过期，如果过期将停止会话；**出于性能考虑， 一般情况下都是获取会话时来验证会话是否过期并停止会话的**； 但是如在 web 环境中，如果用户不主动退出是不知道会话是否过期的，因此需要定期的检测会话是否过期，Shiro 提供了会话验证调度器 SessionValidationScheduler 来做这件事情。

#### Session源代码

![](http://i.imgur.com/oU7r8WX.png)

Session是个状态性的数据上下文，可以理解为每个用户都有一个特定数据库，该数据库存储着每个用户自己的数据，在shiro里，它是和Subject绑定在一起的，通常用户通过Subject.getSession来获取使用。它在系统内会存活一段时间为用户提供客户端浏览器和应用服务器通讯的一些功能。以下是一些关于Session的使用场景。 

1、用户登陆成功后，应用服务器产生个Session，且返回该Session的唯一标识符ID给客户端保存（可通过cookie，或者uri参数等形式）。这样用户访问需要验证后的URL时，应用服务器就能识别。

**注意：shiro在用户第一次通过浏览器第一次访问应用服务器时，就会产生Session了，且返回SessionID，后面如果用户再登陆的话，则会在Session里存储该用户已经登陆的。 **

2、像网上商城，用户通常都会浏览自己喜欢商品，满意后则添加到购物车，由于Session相当于个人的小数据库，此时可以利用Session存储用户添加到购物车的商品，等用户选择完去付款时，此时从Session中获取该用户购物车的商品进行计费、生成订单付费即可。

Shiro实现了自己的Session，即由shiro来管理Session的生命周期。可以为任意的应用提供Session支持。

**具体：**http://blog.csdn.net/wojiaolinaaa/article/details/48312299

1、Session

	public interface Session {
		//返回Session的标识符
	    Serializable getId();
	
		//根据key获取Session存储的值
	    Object getAttribute(Object key) throws InvalidSessionException;
	
		//设置值和key到session中
	    void setAttribute(Object key, Object value) throws InvalidSessionException;
	
		//根据key删除值
	    Object removeAttribute(Object key) throws InvalidSessionException;
	
		...
	}

2、SimpleSession

	//SimpleSession实现了ValidatingSession和Serializable接口，支持验证session操作和序列化
	public class SimpleSession implements ValidatingSession, Serializable {
	
		private transient Map<Object, Object> attributes;
		//sessonID，用于保持客户端浏览器和服务端Session存储容器之间的标识
	    private transient Serializable id;
	
		public Serializable getId() {
	        return this.id;
	    }
	
		public Object getAttribute(Object key) {
	        Map<Object, Object> attributes = getAttributes();
	        if (attributes == null) {
	            return null;
	        }
	        return attributes.get(key);
	    }
	
		public void setAttribute(Object key, Object value) {
	        if (value == null) {
	            removeAttribute(key);
	        } else {
	            getAttributesLazy().put(key, value);
	        }
	    }
	
	    public Object removeAttribute(Object key) {
	        Map<Object, Object> attributes = getAttributes();
	        if (attributes == null) {
	            return null;
	        } else {
	            return attributes.remove(key);
	        }
	    }
	}

3、DelegatingSession

服务端的代理的Session，**该DelegatingSession 只是保存了真正的底层Session（SimpleSession）的key，然后根据该key来查找到SimpleSession再代理它的操作**。Subject.getSession()获取的就是该DelegatingSession，也许是为了不让用户破坏底层Session的一些特性吧

	public class DelegatingSession implements Session, Serializable {
		private final SessionKey key;
		//用于根据SessionKey来操作真正的底层Session
	    private final transient NativeSessionManager sessionManager;
		
		public Serializable getId() {
	        return key.getSessionId();
	    }
	
		public Object getAttribute(Object attributeKey) throws InvalidSessionException {
	        return sessionManager.getAttribute(this.key, attributeKey);
	    }
	
	    public void setAttribute(Object attributeKey, Object value) throws InvalidSessionException {
	        if (value == null) {
	            removeAttribute(attributeKey);
	        } else {
	            sessionManager.setAttribute(this.key, attributeKey, value);
	        }
	    }
	
	    public Object removeAttribute(Object attributeKey) throws InvalidSessionException {
	        return sessionManager.removeAttribute(this.key, attributeKey);
	    }
	}

4、ProxiedSession

**该类主要作用是代理真正的Session**，为子类提供可重写方法。如：不可调用代理某个方法，如果调用则抛出异常

	public class ProxiedSession implements Session {
	
		//真正的Session
	    protected final Session delegate;
	
		public Object getAttribute(Object key) throws InvalidSessionException {
	        return delegate.getAttribute(key);
	    }
	
	
	    public void setAttribute(Object key, Object value) throws InvalidSessionException {
	        delegate.setAttribute(key, value);
	    }
	
	
	    public Object removeAttribute(Object key) throws InvalidSessionException {
	        return delegate.removeAttribute(key);
	    }
	}

5、ImmutableSession

ImmutableProxiedSession 继承与ProxiedSession ，该类主要作用是返回一个不可修改Session的代理Session,对于修改的方法都重写抛异常

	public class ImmutableProxiedSession extends ProxiedSession {
	
	}

6、StoppingAwareProxiedSession

StoppingAwareProxiedSession 主要是增强代理Session的方法

	private class StoppingAwareProxiedSession extends ProxiedSession {
	
	}

7、HttpServletSession 

**该Session仅仅只是代理了servlet的HttpSession**。方便与ServletContainerSessionManager和shiro实现可易配置。把session交由servlet Container来控制生命周期。

#### SessionFactory源码分析

SessionManager根据SessionContext来创建出一个Session，默认是实现是SimpleSessionFactory

![](http://i.imgur.com/obPr9zS.png)

默认的SessionFactory SimpleSessionFactory的创建过程：

	public Session createSession(SessionContext initData) {
	    if (initData != null) {
	        String host = initData.getHost();
	        if (host != null) {
	            return new SimpleSession(host);
	        }
	    }
	    return new SimpleSession();
	}

如果SessionContext有host信息，就传递给Session，然后就是直接new一个Session接口的实现SimpleSession

#### SessionDAO

SessionDAO接口，即对Session进行增删改查

	public interface SessionDAO {
	    Serializable create(Session session);
	    Session readSession(Serializable sessionId) throws UnknownSessionException;
	    void update(Session session) throws UnknownSessionException;
	    void delete(Session session);
	    Collection<Session> getActiveSessions();
	}

* create:为Session分配Session-ID，并保存Session
* readSession:提供Session-ID取出Session

**SessionDAO 接口继承关系如下：**

![](http://i.imgur.com/OJePhsn.png)

**AbstractSessionDAO**：有一个重要的属性SessionIdGenerator，它负责给Session创建sessionId，SessionIdGenerator接口如下：

	public interface SessionIdGenerator {
	    Serializable generateId(Session session);
	}

很简单，参数为Session，返回sessionId。SessionIdGenerator 的实现有两个JavaUuidSessionIdGenerator、RandomSessionIdGenerator。而AbstractSessionDAO默认采用的是JavaUuidSessionIdGenerator，如下：

	public AbstractSessionDAO() {
		this.sessionIdGenerator = new JavaUuidSessionIdGenerator();
	}

AbstractSessionDAO实现了接口create方法，具体创建Session，存储，传回Session-ID方法由子类实现：

	public Serializable create(Session session) {
        Serializable sessionId = doCreate(session);
        verifySessionId(sessionId);
        return sessionId;
    }

	abstract Serializable doCreate(Session session)；

AbstractSessionDAO实现了接口readSession方法，具体根据Session-ID从缓存中获取Session的方法由子类实现：

	public Session readSession(Serializable sessionId) throws UnknownSessionException {
        Session s = doReadSession(sessionId);
        if (s == null) {
            throw new UnknownSessionException("There is no session with id [" + sessionId + "]");
        }
        return s;
    }

    protected abstract Session doReadSession(Serializable sessionId);


**MemorySessionDAO**：继承了AbstractSessionDAO，它把Session存储在一个ConcurrentMap<Serializable, Session> sessions集合中，key为sessionId，value为Session

	//Session存储在ConcurrentMap集合中
    private ConcurrentMap<Serializable, Session> sessions;
    public MemorySessionDAO() {
        this.sessions = new ConcurrentHashMap<Serializable, Session>();
    }

	//重写父类创建Session的方法
    protected Serializable doCreate(Session session) {
		//调用父类的获取Session-ID
        Serializable sessionId = generateSessionId(session);
		//为Session设置Session-ID
        assignSessionId(session, sessionId);
		//存储Session
        storeSession(sessionId, session);
        return sessionId;
    }

    protected Session storeSession(Serializable id, Session session) {
        if (id == null) {
            throw new NullPointerException("id argument cannot be null.");
        }
        return sessions.putIfAbsent(id, session);
    }

	//重写父类读取Session的方法，从Map中根据键值取出Session
	protected Session doReadSession(Serializable sessionId) {
        return sessions.get(sessionId);
    }

**CachingSessionDAO**：主要配合在别的地方存储session，维护了一个成员变量cacheManager，默认的activeSessionsCacheName名为shiro-activeSessionCache

	private CacheManager cacheManager;

扩展了AbstractSessionDAO的create方法，在创建Session后，放入缓存

	public Serializable create(Session session) {
	        Serializable sessionId = super.create(session);
	        cache(session, sessionId);
	        return sessionId;
	    }

扩展了AbstractSessionDAO的readSession方法，先从缓存中获取，如果不行，再去数据库获取

    public Session readSession(Serializable sessionId) throws UnknownSessionException {
        Session s = getCachedSession(sessionId);
        if (s == null) {
            s = super.readSession(sessionId);
        }
        return s;
    }

**EnterpriseCacheSessionDAO**：doCreate操作完成了为Session分配Session-ID

    protected Serializable doCreate(Session session) {
        Serializable sessionId = generateSessionId(session);
        assignSessionId(session, sessionId);
        return sessionId;
    }

doReadSession操作不执行任何功能

	protected Session doReadSession(Serializable sessionId) {
        return null; //should never execute because this implementation relies on parent class to access cache, which
        //is where all sessions reside - it is the cache implementation that determines if the
        //cache is memory only or disk-persistent, etc.
    }

如果使用默认的EnterpriseCacheSessionDAO，那么设置的缓存管理器就是内存，与MemorySessionDAO无任何差异，如果使用了外部的缓存管理器，如Ehcache，则将缓存保存到指定的缓存中

    public EnterpriseCacheSessionDAO() {
        setCacheManager(new AbstractCacheManager() {
            @Override
            protected Cache<Serializable, Session> createCache(String name) throws CacheException {
                return new MapCache<Serializable, Session>(name, new ConcurrentHashMap<Serializable, Session>());
            }
        });
    }

注意：这些缓存不支持持久化，如果要保存缓存进入数据库，需要自定义SessionDAO的doCreate()和doReadSession()方法，见张开涛的跟我学Shiro

#### SessionManager源代码解析

http://www.cnblogs.com/Kavlez/p/4135857.html

正如其名，sessionManager用于为应用中的Subject管理session，比如创建、删除、失效或者验证等，和Shiro中的其他核心组件一样，他由SecurityManager维护。

Shiro为SessionManager提供了3个实现类(顺便也整理一下与SecurityManager实现类的关系)。

* DefaultSessionManager
* DefaultWebSessionManager
* ServletContainerSessionManager

其中ServletContainerSessionManager只适用于servlet容器中，如果需要支持多种客户端访问，则应该使用DefaultWebSessionManager。

SessionManager的接口关系：

![](http://i.imgur.com/GfAv20z.png)

#### ThreadContext、SubjectContext（类似）

http://www.codeweblog.com/shiro%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%BA%8C-subject%E5%92%8Csession/

##### ThreadContext（**类似于数据库，共享Subject、SecurityManager**）

ThreadLocal模式线程内**共享Subject、SecurityManager**，ThreadLocal模式只能共享一个变量，由于需求要保存多个对象，保存这个变量类型为Map

	public abstract class ThreadContext {
	
		public static final String SECURITY_MANAGER_KEY = ThreadContext.class.getName() + "_SECURITY_MANAGER_KEY";
	
		public static final String SUBJECT_KEY = ThreadContext.class.getName() + "_SUBJECT_KEY";
	
		private static final ThreadLocal<Map<Object, Object>> resources = new InheritableThreadLocalMap<Map<Object, Object>>();
		.....
	}

绑定SecurityManager：如果SecurityManager不为空，向线程局部变量的map中，以键存入SECURITY_MANAGER_KEY，值securityManager存入；

    public static void bind(SecurityManager securityManager) {
        if (securityManager != null) {
            put(SECURITY_MANAGER_KEY, securityManager);
        }
    }

    public static void put(Object key, Object value) {
        ensureResourcesInitialized();
        resources.get().put(key, value);
    }

	//如果map没有初始化，新建一个HashMap
    private static void ensureResourcesInitialized(){
        if (resources.get() == null){
           resources.set(new HashMap<Object, Object>());
        }
    }

取出SecurityManager：取出线程局部变量中的map，根据资源获得SecurityManager

    public static SecurityManager getSecurityManager() {
        return (SecurityManager) get(SECURITY_MANAGER_KEY);
    }

    public static Object get(Object key) {
        Object value = getValue(key);
        return value;
    }

    private static Object getValue(Object key) {
        Map<Object, Object> perThreadResources = resources.get();
        return perThreadResources != null ? perThreadResources.get(key) : null;
    }

设置、取出Subject同理：

    public static void bind(Subject subject) {
        if (subject != null) {
            put(SUBJECT_KEY, subject);
        }
    }

    public static Subject getSubject() {
        return (Subject) get(SUBJECT_KEY);
    }

##### SubjectContext

于ThreadContext的设计类似

#### Subject.getSession(boolean create)

整体三大步骤，先创建一个SessionContext ，然后传递SessionContext给securityMa但我们可以看下如何创建Session的nager来创建Session，最后是装饰Session,由于创建Session过程内容比较多，先说说装饰Session。

	public class DelegatingSubject implements Subject {
	
		public Session getSession(boolean create) {		
		    if (this.session == null && create) {
				//①创建SessionContext
		        SessionContext sessionContext = createSessionContext();
				//②SecurityManager调用SessionManager创建Session
		        Session session = this.securityManager.start(sessionContext);
				//③装饰Session
		        this.session = decorate(session);
		    }
		    return this.session;		
		}
		
		//创建SessionContext
	    protected SessionContext createSessionContext() {
	        SessionContext sessionContext = new DefaultSessionContext();
	        if (StringUtils.hasText(host)) {
	            sessionContext.setHost(host);
	        }
	        return sessionContext;
	    }

		protected Session decorate(Session session) {
	        if (session == null) {
	            throw new IllegalArgumentException("session cannot be null");
	        }
	        return new StoppingAwareProxiedSession(session, this);
	    }
	}

装饰Session就是将Session和DelegatingSubject封装起来，返回的Session包含Subject和Session的实例，其中owner表示拥有该Session的Subject，delegate为代理的Session，类型为DelegatingSession

	private class StoppingAwareProxiedSession extends ProxiedSession {
        private final DelegatingSubject owner;

        private StoppingAwareProxiedSession(Session target, DelegatingSubject owningSubject) {
            super(target);
            owner = owningSubject;
        }
    }

Subject调用SessionManager创建Session的具体过程可以参见上节：Session、SessionFactory、SessionDAO、SessionManager

#### SecurityUtils创建Subject

SecurityUtils总体如下，SecurityUtils类中维护一个securityManager变量，它需要由用户自己创建SecurityManager，然后set进入SecurityUtils中进行存储，在应用程序中共享SecurityManager

	public abstract class SecurityUtils {
	
		private static SecurityManager securityManager;
	
		public static Subject getSubject() {
	        Subject subject = ThreadContext.getSubject();
	        if (subject == null) {
	            subject = (new Subject.Builder()).buildSubject();
	            ThreadContext.bind(subject);
	        }
	        return subject;
	    }
	
	    public static void setSecurityManager(SecurityManager securityManager) {
	        SecurityUtils.securityManager = securityManager;
	    }
	
	    public static SecurityManager getSecurityManager() throws UnavailableSecurityManagerException {
	        SecurityManager securityManager = ThreadContext.getSecurityManager();
	        if (securityManager == null) {
	            securityManager = SecurityUtils.securityManager;
	        }
	        return securityManager;
	    }
	}

第一次使用SecurityUtils.getSubject()来获取Subject：

	public static Subject getSubject() {
        Subject subject = ThreadContext.getSubject();
        if (subject == null) {
            subject = (new Subject.Builder()).buildSubject();
            ThreadContext.bind(subject);
        }
        return subject;
    }

首先使用ThreadLocal模式来获取，若没有则使用

	Subject subject = (new Subject.Builder()).buildSubject();

创建一个并绑定到当前线程（<font color='red'>**Subject与线程绑定**</font>）

此时创建使用的是Subject内部类Builder来创建的，Builder会创建一个SubjectContext接口的实例DefaultSubjectContext，最终会委托securityManager来根据SubjectContext信息来创建一个Subject

	public interface Subject {
	
		.....
	
		public static class Builder {
		
		    private final SubjectContext subjectContext;
		    private final SecurityManager securityManager;

			public Builder() {
		        this(SecurityUtils.getSecurityManager());
		    }

			//Builder会创建一个SubjectContext接口的实例DefaultSubjectContext
			public Builder(SecurityManager securityManager) {
	            if (securityManager == null) {
	                throw new NullPointerException("SecurityManager method argument cannot be null.");
	            }
	            this.securityManager = securityManager;
	            this.subjectContext = newSubjectContextInstance();
	            if (this.subjectContext == null) {
	                throw new IllegalStateException("Subject instance returned from 'newSubjectContextInstance' " +
	                        "cannot be null.");
	            }
	            this.subjectContext.setSecurityManager(securityManager);
	        }

			protected SubjectContext newSubjectContextInstance() {
	            return new DefaultSubjectContext();
	        }
		
			//委托securityManager来根据SubjectContext信息来创建一个Subject
			public Subject buildSubject() {
		        return this.securityManager.createSubject(this.subjectContext);
		    }
		
		}
	
	}

##### ★★★★★在DefaultSecurityManager的createSubject

下面详细说下该过程，**在DefaultSecurityManager的createSubject**方法中：

	public Subject createSubject(SubjectContext subjectContext) {
	        SubjectContext context = copy(subjectContext);
	
	        context = ensureSecurityManager(context);
	
	        context = resolveSession(context);
	
	        context = resolvePrincipals(context);
	
	        Subject subject = doCreateSubject(context);
	
	        save(subject);
	
	        return subject;
	    }

首先就是复制SubjectContext，即新建一个SubjectContext实例，与原来的对象具有相同的属性值，SubjectContext的详细介绍见下一节

对于context，把能获取到的参数都凑齐，SecurityManager、Session、Principal，以Session为例，整个过程如下;

	public class DefaultSecurityManager extends SessionsSecurityManager {

		//①以获取Session为例
		protected SubjectContext resolveSession(SubjectContext context) {
			//②尝试获取context的map中获取Session
	        if (context.resolveSession() != null) {
	            log.debug("Context already contains a session.  Returning.");
	            return context;
	        }
	        try {
	            //Context couldn't resolve it directly, let's see if we can since we have direct access to
	            //the session manager:
	
				//③若没有再尝试获取sessionId
	            Session session = resolveContextSession(context);
	            if (session != null) {
	                context.setSession(session);
	            }
	        } catch (InvalidSessionException e) {
	            log.debug("Resolved SubjectContext context session is invalid.  Ignoring and creating an anonymous " +
	                    "(session-less) Subject instance.", e);
	        }
	        return context;
	    }
	
	    protected Session resolveContextSession(SubjectContext context) throws InvalidSessionException {
	        SessionKey key = getSessionKey(context);
	        if (key != null) {
	            return getSession(key);
	        }
	        return null;
	    }

		protected SessionKey getSessionKey(SubjectContext context) {
	        Serializable sessionId = context.getSessionId();
	        if (sessionId != null) {
	            return new DefaultSessionKey(sessionId);
	        }
	        return null;
	    }
	
	}

首先调用SubjectContext的resolveSession方法，尝试获取context的map中获取Session：

	1、首先获取与SubjectContext中Session
	2、如果没找到，尝试获取SubjectContext存储的Subject，如果存在的话，调用Subject的getSession方法获取Session（不新建Session，没有返回null）

若没有再尝试获取sessionId,若果有了sessionId则构建成一个DefaultSessionKey来获取对应的Session。

注意：第一次调用SecurityUtil.getSubject()创建Subject的时候不新建Session

SubjectContext的参数填充完整后，进行实际的Subject创建工作，调用DefaultSecurityManager的doCreateSubject方法：请按照1，2，3，4的顺序查看

<pre>
public class DefaultSecurityManager extends SessionsSecurityManager {

    <font color='red'>protected SubjectDAO subjectDAO;</font>
    <font color='red'>protected SubjectFactory subjectFactory;</font>

	//④SubjectDAO、SubjectFactory默认值
    public DefaultSecurityManager() {
        super();
        <font color='red'>this.subjectFactory = new DefaultSubjectFactory();</font>
        <font color='red'>this.subjectDAO = new DefaultSubjectDAO();</font>
    }

	//①创建Subject
	public Subject createSubject(SubjectContext subjectContext) {
		//SubjectContext参数的获取
        SubjectContext context = copy(subjectContext);
        context = ensureSecurityManager(context);
        context = resolveSession(context);
        context = resolvePrincipals(context);
		//②具体创建Subject
        <font color='red'>Subject subject = doCreateSubject(context);</font>
        save(subject);
        return subject;
    }
	
	//③
	//通过SubjectFactory工厂接口来创建Subject的，而DefaultSecurityManager默认使用的SubjectFactory是DefaultSubjectFactory：
	protected Subject doCreateSubject(SubjectContext context) {
        return getSubjectFactory().createSubject(context);
    }
}
</pre>

继续看DefaultSubjectFactory是怎么创建Subject的：

	public Subject createSubject(SubjectContext context) {
        SecurityManager securityManager = context.resolveSecurityManager();
        Session session = context.resolveSession();
        boolean sessionCreationEnabled = context.isSessionCreationEnabled();
        PrincipalCollection principals = context.resolvePrincipals();
        boolean authenticated = context.resolveAuthenticated();
        String host = context.resolveHost();

        return new DelegatingSubject(principals, authenticated, host, session, sessionCreationEnabled, securityManager);
    }

仍然就是从SubjectContext获取这些属性，传递给新建的Subject实例：DelegatingSubject，也没什么好说的，到此为止创建一个Subject实例DelegatingSubject，维护了Principal、authenticated、host、session，此时除了host全为空，并返回；

	public class DelegatingSubject implements Subject {
	
	    private static final Logger log = LoggerFactory.getLogger(DelegatingSubject.class);
	
	    private static final String RUN_AS_PRINCIPALS_SESSION_KEY =
	            DelegatingSubject.class.getName() + ".RUN_AS_PRINCIPALS_SESSION_KEY";
	
	    protected PrincipalCollection principals;
	    protected boolean authenticated;
	    protected String host;
	    protected Session session;
	
	}


创建完成之后，就需要将刚创建的Subject保存起来

来看下save方法：

	protected void save(Subject subject) {
        this.subjectDAO.save(subject);
    }

可以看到又是使用另一个模块来完成的即SubjectDAO，SubjectDAO接口的默认实现为DefaultSubjectDAO()，具体的实现类DefaultSubjectDAO是如何来保存的：

	public Subject save(Subject subject) {
        if (isSessionStorageEnabled(subject)) {
            saveToSession(subject);
        } else {
            log.trace("Session storage of subject state for Subject [{}] has been disabled: identity and " +
                    "authentication state are expected to be initialized on every request or invocation.", subject);
        }

        return subject;
    }

首先就是判断isSessionStorageEnabled，是否要存储该Subject的session到
DefaultSubjectDAO：有一个重要属性SessionStorageEvaluator，它是用来决定一个Subject的Session来记录Subject的状态，接口如下

	public interface SessionStorageEvaluator {
	    boolean isSessionStorageEnabled(Subject subject);
	}

其实现为DefaultSessionStorageEvaluator：

	public class DefaultSessionStorageEvaluator implements SessionStorageEvaluator {
	
	    private boolean sessionStorageEnabled = true;
	
	    public boolean isSessionStorageEnabled(Subject subject) {
	        return (subject != null && subject.getSession(false) != null) || isSessionStorageEnabled();
	    }
	}

决定策略就是通过DefaultSessionStorageEvaluator 的sessionStorageEnabled的true或false 和subject是否有Session对象来决定的。如果允许存储Subject的Session的话，下面就说 **具体的存储过程**：

	protected void saveToSession(Subject subject) {
        mergePrincipals(subject);
        mergeAuthenticationState(subject);
    }

	protected void mergePrincipals(Subject subject) {
        //merge PrincipalCollection state:

        PrincipalCollection currentPrincipals = null;

        //SHIRO-380: added if/else block - need to retain original (source) principals
        //This technique (reflection) is only temporary - a proper long term solution needs to be found,
        //but this technique allowed an immediate fix that is API point-version forwards and backwards compatible
        //
        //A more comprehensive review / cleaning of runAs should be performed for Shiro 1.3 / 2.0 +
        if (subject.isRunAs() && subject instanceof DelegatingSubject) {
            try {
                Field field = DelegatingSubject.class.getDeclaredField("principals");
                field.setAccessible(true);
                currentPrincipals = (PrincipalCollection)field.get(subject);
            } catch (Exception e) {
                throw new IllegalStateException("Unable to access DelegatingSubject principals property.", e);
            }
        }
        if (currentPrincipals == null || currentPrincipals.isEmpty()) {
            currentPrincipals = subject.getPrincipals();
        }

        Session session = subject.getSession(false);

        if (session == null) {
           //只有当Session为空，并且currentPrincipals不为空的时候才会去创建Session
           //Subject subject = SecurityUtils.getSubject()此时两者都是为空的，
           //不会去创建Session
            if (!CollectionUtils.isEmpty(currentPrincipals)) {
                session = subject.getSession();
                session.setAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY, currentPrincipals);
            }
            //otherwise no session and no principals - nothing to save
        } else {
            PrincipalCollection existingPrincipals =
                    (PrincipalCollection) session.getAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY);

            if (CollectionUtils.isEmpty(currentPrincipals)) {
                if (!CollectionUtils.isEmpty(existingPrincipals)) {
                    session.removeAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY);
                }
                //otherwise both are null or empty - no need to update the session
            } else {
                if (!currentPrincipals.equals(existingPrincipals)) {
                    session.setAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY, currentPrincipals);
                }
                //otherwise they're the same - no need to update the session
            }
        }
    }

上面有我们关心的重点，当subject.getSession(false)获取的Session为空时（它不会去创建Session），此时就需要去创建Session，subject.getSession()则默认调用的是subject.getSession(true),则会进行Session的创建，创建过程上文已详细说明了。

在第一次创建Subject的时候

	Subject subject = SecurityUtils.getSubject();

虽然Session为空，但此时还没有用户身份信息，也不会去创建Session

**所以创建的DelegatingSubject实例，成员变量Principal、Session全是空**

#### Subject.login(token)

案例中的**subject.login(token)**该过程则会去创建Session，具体看下过程：

首先DelegatingSubject调用login方法会委托给SecurityManager的securityManager.login(this, token)方法

    public void login(AuthenticationToken token) throws AuthenticationException {
        clearRunAsIdentitiesInternal();

		//SecurityManager完成实际的登录操作
        Subject subject = securityManager.login(this, token);

        PrincipalCollection principals;

        String host = null;

        if (subject instanceof DelegatingSubject) {
            DelegatingSubject delegating = (DelegatingSubject) subject;
            principals = delegating.principals;
            host = delegating.host;
        } else {
            principals = subject.getPrincipals();
        }

        if (principals == null || principals.isEmpty()) {
            String msg = "Principals returned from securityManager.login( token ) returned a null or " +
                    "empty value.  This value must be non null and populated with one or more elements.";
            throw new IllegalStateException(msg);
        }
        this.principals = principals;
        this.authenticated = true;
        if (token instanceof HostAuthenticationToken) {
            host = ((HostAuthenticationToken) token).getHost();
        }
        if (host != null) {
            this.host = host;
        }
        Session session = subject.getSession(false);
        if (session != null) {
            this.session = decorate(session);
        } else {
            this.session = null;
        }
    }

SecurityManager的login方法分为两步：1、对用户名、密码验证 2、创建Subject

	public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
		//对用户名、密码验证
        AuthenticationInfo info;
        try {
            info = authenticate(token);
        } catch (AuthenticationException ae) {
            try {
                onFailedLogin(token, ae, subject);
            } catch (Exception e) {
                if (log.isInfoEnabled()) {
                    log.info("onFailedLogin method threw an " +
                            "exception.  Logging and propagating original AuthenticationException.", e);
                }
            }
            throw ae; //propagate
        }
         //在该过程会进行Session的创建
        Subject loggedIn = createSubject(token, info, subject);

        onSuccessfulLogin(token, info, loggedIn);

        return loggedIn;
    }

对于验证过程这里不再说明，重点还是在验证通过后，<font color='red'>**会创建一个新的Subject，同时Subject中包含认证信息和Session；**</font>

新建SubjectContext，并设置了认证信息

	protected Subject createSubject(AuthenticationToken token, AuthenticationInfo info, Subject existing) {
	        SubjectContext context = createSubjectContext();
	        context.setAuthenticated(true);
	        context.setAuthenticationToken(token);
	        context.setAuthenticationInfo(info);
	        if (existing != null) {
	            context.setSubject(existing);
	        }
			//根据SubjectContext创建Subject，之前详细讲解过
	        return createSubject(context);
	    }

	protected SubjectContext createSubjectContext() {
        return new DefaultSubjectContext();
    }

<font color='red'>**在系统刚启动时，创建Subject时，参数SubjectContext中的Session、Principal都为空，不创建Session；但在调用Subject.login有了认证成功的AuthenticationInfo信息，调用SubjectContext在resolvePrincipals便可以获取用户信息**</font>

再次调用createSubject(context）创建Subject，此时resolvePrincipals(context)中，可以获取到刚才认证传入的AuthenticationInfo，并取出Principal。**在SubjectContext中Principal不为空，但Session为空**
    
	public Subject createSubject(SubjectContext subjectContext) {

        SubjectContext context = copy(subjectContext);
        context = ensureSecurityManager(context);
        context = resolveSession(context);
        context = resolvePrincipals(context);
        Subject subject = doCreateSubject(context);
        save(subject);

        return subject;
    }

<font color='red'>**PrincipalCollection不为空了，在<font color='blue'>save(subject)</font>的时候会得到session为空，同时PrincipalCollection不为空，则会执行Session的创建。**</font>也就是说在认证通过后，会执行Session的创建，返回的Session为Subject和Session的封装体，并在Session中存入Principal

<font color='blue'>**调用subject.getSession()，回返回一个Subject和Session的封装体，并且设置该Session为Subject的成员变量**</font>

	protected void mergePrincipals(Subject subject) {
        PrincipalCollection currentPrincipals = null;
        if (subject.isRunAs() && subject instanceof DelegatingSubject) {
            try {
                Field field = DelegatingSubject.class.getDeclaredField("principals");
                field.setAccessible(true);
                currentPrincipals = (PrincipalCollection)field.get(subject);
            } catch (Exception e) {
                throw new IllegalStateException("Unable to access DelegatingSubject principals property.", e);
            }
        }
        if (currentPrincipals == null || currentPrincipals.isEmpty()) {
            currentPrincipals = subject.getPrincipals();
        }

        Session session = subject.getSession(false);

        if (session == null) {
            if (!CollectionUtils.isEmpty(currentPrincipals)) {
				//【此处会执行！！！】Principal不为空会创建一个Session
				//调用subject.getSession()，回返回一个Subject和Session的共同体，并且设置该Session为Subject的成员变量
                session = subject.getSession();

				//Session中存入Principal
                session.setAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY, currentPrincipals);
            }
            //otherwise no session and no principals - nothing to save
        } else {
            PrincipalCollection existingPrincipals =
                    (PrincipalCollection) session.getAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY);

            if (CollectionUtils.isEmpty(currentPrincipals)) {
                if (!CollectionUtils.isEmpty(existingPrincipals)) {
                    session.removeAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY);
                }
                //otherwise both are null or empty - no need to update the session
            } else {
                if (!currentPrincipals.equals(existingPrincipals)) {
                    session.setAttribute(DefaultSubjectContext.PRINCIPALS_SESSION_KEY, currentPrincipals);
                }
                //otherwise they're the same - no need to update the session
            }
        }
    }

委托SecurityManager执行登录操作完毕，回到DelegatingSubject的login函数，**返回了新建的Subject**

Session创建完成之后会进行一次装饰，即用新建一个StoppingAwareProxiedSession对象，其中的delegate为session（也为StoppingAwareProxiedSession类型），owner为Subject，然后又进行如下操作：

<pre>	
public class DelegatingSubject implements Subject {

	public void login(AuthenticationToken token) throws AuthenticationException {
        clearRunAsIdentitiesInternal();
        //★★★★★这里的Subject则是经过认证后创建的并且也含有刚才创建的session
        <font color='red'>Subject subject = securityManager.login(this, token);</font>

        PrincipalCollection principals;

        String host = null;

        if (subject instanceof DelegatingSubject) {
            DelegatingSubject delegating = (DelegatingSubject) subject;
            //we have to do this in case there are assumed identities - we don't want to lose the 'real' principals:
            principals = delegating.principals;
            host = delegating.host;
        } else {
            principals = subject.getPrincipals();
        }

        if (principals == null || principals.isEmpty()) {
            String msg = "Principals returned from securityManager.login( token ) returned a null or " +
                    "empty value.  This value must be non null and populated with one or more elements.";
            throw new IllegalStateException(msg);
        }
		<font color='red'>//内部Subject的principal、session、authenticated复制到外部的Subject</font>
        this.principals = principals;
        this.authenticated = true;
        if (token instanceof HostAuthenticationToken) {
            host = ((HostAuthenticationToken) token).getHost();
        }
        if (host != null) {
            this.host = host;
        }
        Session session = subject.getSession(false);
        if (session != null) {
            //在这里可以看到又进行了一次装饰
            this.session = decorate(session);
        } else {
            this.session = null;
        }
    }
}
</pre>

subject 创建出来之后，暂且叫内部subject，就是**把认证通过的内部subject的信息和session复制给我们外界使用的subject.login(token)的subject中（principals、host、authenticated、Session）**，这个subject暂且叫外部subject，看下session的赋值，又进行了一次装饰，这次装饰则把session(类型为StoppingAwareProxiedSession，**即是内部subject和session的合体)和外部subject绑定到一起**

#### 从Cookie、URL、request中根据Token获取Session

1、客户端请求到来，调用AbstractShiroFilter中的doFilterInternal方法，创建Subject

	final Subject subject = createSubject(request, response);

2、调用WebSubject.Builder().buildWebSubject方法

    protected WebSubject createSubject(ServletRequest request, ServletResponse response) {
        return new WebSubject.Builder(getSecurityManager(), request, response).buildWebSubject();
    }

3、创建WebSubject与默认的Subject稍微有差别，但还是调用父类（即Subject接口的Builder中的BuildSubject方法）

	public WebSubject buildWebSubject() {
            Subject subject = super.buildSubject();
            if (!(subject instanceof WebSubject)) {
                String msg = "Subject implementation returned from the SecurityManager was not a " +
                        WebSubject.class.getName() + " implementation.  Please ensure a Web-enabled SecurityManager " +
                        "has been configured and made available to this builder.";
                throw new IllegalStateException(msg);
            }
            return (WebSubject) subject;
        }

4、父类的buildSubject方法，将创建Subject的方法委托为SecurityManager

	public Subject buildSubject() {
        return this.securityManager.createSubject(this.subjectContext);
    }

5、DefaultSecurityManager的createSubject

    public Subject createSubject(SubjectContext subjectContext) {
        SubjectContext context = copy(subjectContext);
        context = ensureSecurityManager(context);
        context = resolveSession(context);
        context = resolvePrincipals(context);
        Subject subject = doCreateSubject(context);
        save(subject);
        return subject;
    }

6、resolveSession(context)解析Session，context.resolveSession()返回的是null，调用Session session = resolveContextSession(context)尝试获取Session

    protected SubjectContext resolveSession(SubjectContext context) {
		//①
        if (context.resolveSession() != null) {
            log.debug("Context already contains a session.  Returning.");
            return context;
        }
        try {
            //Context couldn't resolve it directly, let's see if we can since we have direct access to 
            //the session manager:
			//②
            Session session = resolveContextSession(context);
            if (session != null) {
                context.setSession(session);
            }
        } catch (InvalidSessionException e) {
            log.debug("Resolved SubjectContext context session is invalid.  Ignoring and creating an anonymous " +
                    "(session-less) Subject instance.", e);
        }
        return context;
    }

7、具体看resolveContextSession，调用getSessionKey(context)

    protected Session resolveContextSession(SubjectContext context) throws InvalidSessionException {
        SessionKey key = getSessionKey(context);
        if (key != null) {
            return getSession(key);
        }
        return null;
    }

8、DefaultWebSecurityManager重写getSessionKey(SubjectContext context)方法

    protected SessionKey getSessionKey(SubjectContext context) {
        if (WebUtils.isWeb(context)) {
            Serializable sessionId = context.getSessionId();
            ServletRequest request = WebUtils.getRequest(context);
            ServletResponse response = WebUtils.getResponse(context);
            return new WebSessionKey(sessionId, request, response);
        } else {
            return super.getSessionKey(context);

        }
    }

返回WebSessionKey实例，但WebSessionKey中的sessionId为null

9、回到第7步SessionKey key = getSessionKey(context)，此时key不为空，调用SessionSecurityManager的getSession方法（接口方法）

    public Session getSession(SessionKey key) throws SessionException {
        return this.sessionManager.getSession(key);
    }

10、调用AbstractNativeSessionManager中的getSession

    public Session getSession(SessionKey key) throws SessionException {
        Session session = lookupSession(key);
        return session != null ? createExposedSession(session, key) : null;
    }

11、进入到lookupSession

    private Session lookupSession(SessionKey key) throws SessionException {
        if (key == null) {
            throw new NullPointerException("SessionKey argument cannot be null.");
        }
        return doGetSession(key);
    }

12、doGetSession被子类AbstractValidatingSessionManager重写

    protected final Session doGetSession(final SessionKey key) throws InvalidSessionException {
        enableSessionValidationIfNecessary();

        log.trace("Attempting to retrieve session with key {}", key);

        Session s = retrieveSession(key);
        if (s != null) {
            validate(s, key);
        }
        return s;
    }

13、retrieveSession也是个抽象方法，被子类DefaultSessionManager重写，

    protected Session retrieveSession(SessionKey sessionKey) throws UnknownSessionException {
        Serializable sessionId = getSessionId(sessionKey);
        if (sessionId == null) {
            log.debug("Unable to resolve session ID from SessionKey [{}].  Returning null to indicate a " +
                    "session could not be found.", sessionKey);
            return null;
        }
        Session s = retrieveSessionFromDataSource(sessionId);
        if (s == null) {
            //session ID was provided, meaning one is expected to be found, but we couldn't find one:
            String msg = "Could not find session with ID [" + sessionId + "]";
            throw new UnknownSessionException(msg);
        }
        return s;
    }

14、getSessionId被子类DefaultWebSessionManager重写

    public Serializable getSessionId(SessionKey key) {
        Serializable id = super.getSessionId(key);
        if (id == null && WebUtils.isWeb(key)) {
            ServletRequest request = WebUtils.getRequest(key);
            ServletResponse response = WebUtils.getResponse(key);
            id = getSessionId(request, response);
        }
        return id;
    }

由于父类的getSessionId返回null，执行getSessionId(request,response)

    protected Serializable getSessionId(SessionKey sessionKey) {
        return sessionKey.getSessionId();
    }

15、DefaultWebSessionManager的getSessionId

    protected Serializable getSessionId(ServletRequest request, ServletResponse response) {
        return getReferencedSessionId(request, response);
    }

16、★★★★★★★getReferencedSessionId先解析cookie，在解析URL，最后解析

    private Serializable getReferencedSessionId(ServletRequest request, ServletResponse response) {

        String id = getSessionIdCookieValue(request, response);
        if (id != null) {
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE,
                    ShiroHttpServletRequest.COOKIE_SESSION_ID_SOURCE);
        } else {
            //not in a cookie, or cookie is disabled - try the request URI as a fallback (i.e. due to URL rewriting):

            //try the URI path segment parameters first:
            id = getUriPathSegmentParamValue(request, ShiroHttpSession.DEFAULT_SESSION_ID_NAME);

            if (id == null) {
                //not a URI path segment parameter, try the query parameters:
                String name = getSessionIdName();
                id = request.getParameter(name);
                if (id == null) {
                    //try lowercase:
                    id = request.getParameter(name.toLowerCase());
                }
            }
            if (id != null) {
                request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE,
                        ShiroHttpServletRequest.URL_SESSION_ID_SOURCE);
            }
        }
        if (id != null) {
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID, id);
            //automatically mark it valid here.  If it is invalid, the
            //onUnknownSession method below will be invoked and we'll remove the attribute at that time.
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_IS_VALID, Boolean.TRUE);
        }

        // always set rewrite flag - SHIRO-361
        request.setAttribute(ShiroHttpServletRequest.SESSION_ID_URL_REWRITING_ENABLED, isSessionIdUrlRewritingEnabled());

        return id;
    }

#### SubjectContext、MapContext、Map

接口设计:

![](http://i.imgur.com/5BjziW0.png)

**MapContext**：实现了Map接口，**内部拥有一个类型为HashMap的backingMap属性，大部分方法都由HashMap来实现，然后仅仅更改某些行为**，**MapContext没有选择去继承HashMap，而是使用了组合的方式，更加容易去扩展**，如backingMap的类型不一定非要选择HashMap，可以换成其他的Map实现，一旦MapContext选择继承HashMap，如果想对其他的Map类型进行同样的功能增强的话，就需要另写一个类来继承它然后改变一些方法实现，这样的话就会有很多重复代码。这也是设计模式所强调的少用继承多用组合

	public class MapContext implements Map<String, Object>, Serializable {
	
	    private static final long serialVersionUID = 5373399119017820322L;
	
	    private final Map<String, Object> backingMap;
	
	    public MapContext() {
	        this.backingMap = new HashMap<String, Object>();
	    }
	
	    public MapContext(Map<String, Object> map) {
	        this();
	        if (!CollectionUtils.isEmpty(map)) {
	            this.backingMap.putAll(map);
	        }
	    }
	  //略
	}

**SubjectContext**接口继承了Map<String, Object>，然后加入了几个重要的SecurityManager、SessionId、Subject、PrincipalCollection、Session、boolean authenticated、boolean sessionCreationEnabled、Host、AuthenticationToken、AuthenticationInfo等众多信息。

MapContext中增加了getTypedValue方法，便于获得参数化的类型：

	protected <E> E getTypedValue(String key, Class<E> type) {
	        E found = null;
	        Object o = backingMap.get(key);
	        if (o != null) {
	            if (!type.isAssignableFrom(o.getClass())) {
	                String msg = "Invalid object found in SubjectContext Map under key [" + key + "].  Expected type " +
	                        "was [" + type.getName() + "], but the object under that key is of type " +
	                        "[" + o.getClass().getName() + "].";
	                throw new IllegalArgumentException(msg);
	            }
	            found = (E) o;
	        }
	        return found;
	    }

可以把Map当做公有基本类，然后SubjectContext在此基础上扩展，类似于公有方法和新添加的方法；Map定义了这个类的功能是存储，SubjectContext在此基础上增加了SecurityManager、Subject、Session的存储

讨论1：首先是SubjectContext为什么要去实现Map<String, Object>？

实现对数据的存储有设计方式有两种方式：
1、继承数据结构，如map，自然就可以调用Map的put/get方法对数据进行存储
2、组合方式，即类中存放一个存储型数据结构，如map，调用成员变量的put/get方法实现存储

ThreadContext采用采用的是组合方式，SubjectContext采用的是继承方式

讨论2：SubjectContext接口的作用？

SubjectContext提供了常用的get、set方法，还提供了一个resolve方法，以SecurityManager为例：

	SecurityManager getSecurityManager();
	void setSecurityManager(SecurityManager securityManager);
	SecurityManager resolveSecurityManager();

这些get、set方法则用于常用的设置和获取，而resolve则表示先调用getSecurityManager，如果获取不到，则使用其他途径来获取，如DefaultSubjectContext的实现：

	public SecurityManager resolveSecurityManager() {
        SecurityManager securityManager = getSecurityManager();
        if (securityManager == null) {
            if (log.isDebugEnabled()) {
                log.debug("No SecurityManager available in subject context map.  " +
                        "Falling back to SecurityUtils.getSecurityManager() lookup.");
            }
            try {
                securityManager = SecurityUtils.getSecurityManager();
            } catch (UnavailableSecurityManagerException e) {
                if (log.isDebugEnabled()) {
                    log.debug("No SecurityManager available via SecurityUtils.  Heuristics exhausted.", e);
                }
            }
        }
        return securityManager;
    }

如果getSecurityManager获取不到，则使用SecurityUtils工具来获取。
再如resolvePrincipals

	public PrincipalCollection resolvePrincipals() {
        PrincipalCollection principals = getPrincipals();

        if (CollectionUtils.isEmpty(principals)) {
            //check to see if they were just authenticated:
            AuthenticationInfo info = getAuthenticationInfo();
            if (info != null) {
                principals = info.getPrincipals();
            }
        }

        if (CollectionUtils.isEmpty(principals)) {
            Subject subject = getSubject();
            if (subject != null) {
                principals = subject.getPrincipals();
            }
        }

        if (CollectionUtils.isEmpty(principals)) {
            //try the session:
            Session session = resolveSession();
            if (session != null) {
                principals = (PrincipalCollection) session.getAttribute(PRINCIPALS_SESSION_KEY);
            }
        }

        return principals;
    }

普通的getPrincipals()获取不到，尝试使用其他属性来获取。

#### SubjectFactory

![](http://i.imgur.com/i1mAQCJ.png)

DefaultSubjectFactory: 根据SubjectContext，解析出ecurityManager、Session、PrincipalCollection等信息，生成一个代理Subject：DelegatingSubject

    public Subject createSubject(SubjectContext context) {
        SecurityManager securityManager = context.resolveSecurityManager();
        Session session = context.resolveSession();
        boolean sessionCreationEnabled = context.isSessionCreationEnabled();
        PrincipalCollection principals = context.resolvePrincipals();
        boolean authenticated = context.resolveAuthenticated();
        String host = context.resolveHost();

        return new DelegatingSubject(principals, authenticated, host, session, sessionCreationEnabled, securityManager);
    }

#### Subject

![](http://i.imgur.com/Tdbme0T.png)

#### ★★★会话管理总结

1、服务器刚启动时，创建Subject，其Principal、Session为空
2、用户登录，调用Subject.login获取用户信息，创建Session，在Session中存入Principal信息，更新Subject中的Session、Principal
3、下次用户携带Token登录，恢复Session，取出Principal，建立带状态信息的Subject

### 缓存机制

#### 环境

导入依赖

	<!--shiro与ehcache整合-->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-ehcache</artifactId>
        <version>${shiro.version}</version> <!--1.3.2-->
    </dependency>
    <!--ehcache,纯Java开源缓存框架-->
    <dependency>
        <groupId>net.sf.ehcache</groupId>
        <artifactId>ehcache</artifactId>
        <version>2.10.1</version>
    </dependency>

ehcache.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
	         updateCheck="false">
	
	    <diskStore path="java.io.tmpdir"/>
	
	    <!--
	    Mandatory Default Cache configuration. These settings will be applied to caches
	    created programmtically using CacheManager.add(String cacheName)
	    -->
	    <!--
	       name:缓存名称。
	       maxElementsInMemory：缓存最大个数。
	       eternal:对象是否永久有效，一但设置了，timeout将不起作用。
	       timeToIdleSeconds：设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
	       timeToLiveSeconds：设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
	       overflowToDisk：当内存中对象数量达到maxElementsInMemory时，Ehcache将会对象写到磁盘中。
	       diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
	       maxElementsOnDisk：硬盘最大缓存个数。
	       diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
	       diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
	       memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
	       clearOnFlush：内存数量最大时是否清除。
	    -->
	    <defaultCache
	            maxElementsInMemory="10000"
	            eternal="false"
	            timeToIdleSeconds="120"
	            timeToLiveSeconds="120"
	            overflowToDisk="true"
	            maxElementsOnDisk="10000000"
	            diskPersistent="false"
	            diskExpiryThreadIntervalSeconds="120"
	            memoryStoreEvictionPolicy="LRU"
	    />
	
	    <cache name="shiro"
	           maxEntriesLocalHeap="2000"
	           eternal="false"
	           timeToIdleSeconds="600"
	           timeToLiveSeconds="600"
	           overflowToDisk="false"
	           statistics="true">
	    </cache>
	</ehcache>

### OAuth2

根据应用场景的不同，目前实现开放授权的方法分为两种：一种是使用OAuth协议；另一种是使用IAM服务

* OAuth协议主要适用于针对个人用户对资源的开放授权，比如Google的用户Alice允许别的应用程序（如Facebook）访问他的联系人列表
* IAM它的特点是"预先授权"或"离线授权"，客户端通过REST API去访问资源，资源所有者可以预先知道第三方应用所需的资源请求，一次授权之后，很少会更改。**IAM服务一般在云计算服务中使用**，如阿里云计算服务

#### OAuth 角色

资源拥有者（resource owner）：能 **授权** 访问受保护资源的一个实体，可以是一个人，我们称之为最终用户，如新浪微博用户、zhangsan

资源服务器（resource server）：存储受保护资源，客户端通过 access token 请求资源，资源服务器响应受保护资源给客户端，存储着用户 zhangsan 的微博等信息

授权服务器（authorization server）：成功验证资源拥有者并获取授权之后，授权服务器颁发授权令牌（Access Token）给客户端

客户端（client）：如新浪微博客户端、微格等第三方应用，也可以是它自己的官方应用，其本身不存储资源，而是资源拥有者授权通过后，使用它的授权（授权令牌）访问受保护资源，然后客户端把相应的数据展示出来/提交到服务器。"客户端"术语不代表任何特定实现（如应用运行在一台服务器、桌面、手机或其他设备）

#### OAuth2 协议流程

![](http://i.imgur.com/z8GVJj9.png)

1、客户端从 **资源拥有者** 那请求授权。授权请求可以直接发给资源拥有者，或间接的通过授
权服务器这种中介，后者更可取。（我们常会看到跳出弹框，是否允许第三方授权，需要用户决定是否同意）
2、客户端收到一个 **授权许可**，代表资源服务器提供的授权
3、客户端使用它自己的私有证书及授权许可到 **授权服务器** 验证
4、如果验证成功，则下发一个 **访问令牌**
5、客户端使用访问令牌向 **资源服务器** 请求受保护资源
6、资源服务器会验证访问令牌的有效性，如果成功则下发受保护资源

#### 无状态 Web 应用集成

在一些环境中，可能需要把 Web 应用做成无状态的，即服务器端无状态，就是说 **服务器端不会存储像会话这种东西，而是每次请求时带上相应的用户名进行登录**。如一些 REST 风格的 API，如果不使用 OAuth2 协议，就可以使用如 REST+HMAC 认证进行访问。

HMAC（Hash-based  Message  Authentication  Code）：基于散列的消息认证码，使用一个密钥和一个消息作为输入，生成它们的消息摘要。注意该密钥只有客户端和服务端知道，其他第三方是不知道的。

访问时使用该消息摘要进行传播，服务端然后对该消息摘要进行验证。如果只传递用户名+密码的消息摘要， 一旦被别人捕获可能会重复使用该摘要进行认证。解决办法如：

1、每次客户端申请一个 Token，然后使用该 Token 进行加密，而该 Token 是一次性的，即
只能用一次；有点类似于 OAuth2 的 Token 机制，但是简单些；
2、客户端每次生成一个唯一的 Token，然后使用该 Token 加密，这样服务器端记录下这些
Token，如果之前用过就认为是非法请求

## 权限实例

http://www.360doc.com/content/14/0529/10/11298474_381933566.shtml#

### 权限概述

有一个用户管理系统，注册的用户分为**normal用户，manager用户，admin用户**，有**系统管理、用户管理、角色管理**三类操作，我们规定：

* admin 可以访问 `/admin/**`
* manager 可以访问 `/admin/user/**`
* normal 可以访问 `/admin/role/**`

用户 admin 拥有 admin、manager、normal 三种角色，可以访问`/admin/**`，`/admin/user/**`,`/admin/role/**`的资源

用户 zhangsan 拥有 manager、normal 两种角色，可以访问 `/admin/user/**`，`/admin/role/**`

用户 lisi 拥有 normal 角色，可以访问 `/admin/role/**`


**权限是应用程序的一些基本操作，角色是权限的集合**

我们采用下面的逻辑创建权限表结构（不是绝对的，根据需要修改）

***用户与角色关系：多对多***
一个用户可以有多种角色（normal,manager,admin等等）
一个角色可以有多个用户（user1,user2,user3等等）

***角色与权限关系：多对多***
一个角色可以有多个权限（save,update,delete,query等等）
一个权限可以属于多个角色

![](http://i.imgur.com/O5fa7S2.png)

### 数据库设计

我们创建五张表：

tb_user用户表：设置了3个用户

	+----+-----------+----------------------------------
	| id | user_name | password     
	+----+-----------+----------------------------------      
	|  1 | admin     | 086bad3456e2653dc05e32a77000ce87   
	|  2 | zhangsan  | 39dd5dcf4b69e917f7a32233462c55d4    
	|  3 | lisi      | a259a0625727a416130cdb03e6e9dfb6
	+----+-----------+----------------------------------  
        
tb_role角色表：设置3个角色

	+----+-----------+-------------+
	| id | role_name | description |
	+----+-----------+-------------+
	|  1 | admin     | 系统管理员  |
	|  2 | manager   | 系统顾问    |
	|  3 | normal    | 系统用户    |
	+----+-----------+-------------+-

**tb_user_role用户角色表**：

	+----+---------+---------+
	| id | user_id | role_id |
	+----+---------+---------+
	|  1 | 1       | 1       |
	|  2 | 1       | 2       |
	|  3 | 1       | 3       |
	|  4 | 2       | 2       |
	|  5 | 2       | 3       |
	|  6 | 3       | 3       |
	+----+---------+---------+

tb_resources权限表：

	+----+---------------+------------+----------------+
	| id | resource_name | permission | url            |
	+----+---------------+------------+----------------+
	|  1 | 系统管理      | NULL       | /admin/**      |
	|  2 | 用户管理      | NULL       | /admin/user/** |
	|  3 | 角色管理      | NULL       | /admin/role/** |
	+----+---------------+------------+----------------+   

select * from tb_role_resource角色权限表：

	+----+---------+-------------+
	| id | role_id | resource_id |
	+----+---------+-------------+
	|  1 |       1 |           1 |
	|  2 |       2 |           2 |
	|  3 |       3 |           3 |
	+----+---------+-------------+
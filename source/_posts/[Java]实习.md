---
title: 实习

date: 2017-01-03 11:52:00

categories:
- Java

tags:
- Java
---

## 系统总体架构

![](http://i.imgur.com/pKPNhhK.png)
源文件：https://note.youdao.com/yws/res/8735/WEBRESOURCEb9e23c0741530726b0124ee5d3bfd729

1、一个基于Thrift的微服务需要创建三个模块（最终打包成三个Jar包），分别是Core、Server和Adapter

* core：定义接口。用接口描述语言定义并创建服务
* server：Thrift的服务端。依赖core模块，实现core中定义的接口，**主要的业务逻辑部分，分为Service层、Dao层、缓存和数据库**。微服务就是进行分模块，把各模块放到不同的位置提供服务
* adapter：Thrift的客户端。依赖core模块，**其他模块如需调用Serve的接口，需引入此jar包**

以论坛项目为例，整个论坛项目分为四个模块：

* bp-forum-core:数据定义和服务创建
* bp-forum-server:服务实现
* bp-forum-adapter:客户端，与服务器交互
* bp-forum-wap:页面显示，通过调用adapter，获取服务

对于每一个微服务都有其各自的core、server、adapter模块

注意：虽然Server和Adapter在同一个工程下，但在实际运行环境中，一般Server运行在A主机，而B主机上的应用程序需要调用A主机上的Server，需要引入Adapter依赖，实现远程服务调用

2、前端数据显示：Freemarker——模板+数据=输出网页

## Thrift

http://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/#ibm-pcon
目前流行的服务调用方式有很多种，例如基于 SOAP 消息格式的 Web Service，基于 JSON 消息格式的 RESTful 服务等。其中所用到的数据传输方式包括 XML，JSON 等，然而 XML 相对体积太大，传输效率低，JSON 体积较小，新颖，但还不够完善。本文将介绍由 Facebook 开发的远程服务调用框架 Apache Thrift，它采用接口描述语言定义并创建服务，支持可扩展的跨语言服务开发，所包含的代码生成引擎可以在多种语言中，如 C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk 等创建高效的、无缝的服务，其传输数据采用二进制格式，相对 XML 和 JSON 体积更小，对于高并发、大数据量和多语言的环境更有优势

## Guava Cache

http://www.cnblogs.com/parryyang/p/5777019.html

当某些值会被多次调用时，将数据缓存在内存中提升速度

Guava Cache是一个全内存的本地缓存实现，它提供了线程安全的实现机制。整体上来说Guava cache 是本地缓存的不二之选，简单易用，性能好。

### Guava Cache与ConcurrentMap区别

最基本的区别是ConcurrentMap会一直保存所添加的元素，直到显式的移除；Guava Cache为了限制内存的占用，通常都是设定为自动回收元素

### 创建方式

* CacheLoader
* Callable

相同点：从缓存中取key X的值，如果该值已经缓存过了，则返回缓存中的值，如果没有缓存过，可以通过某个方法来获取这个值。

不同点：

* cacheloader的定义比较宽泛， 是**针对整个cache定义的**，可以认为是**统一的根据key值获取value的方法**
* callable的方式较为灵活，允许你在get的时候，针对特定key值指定获取方式

1、CacheLoader

    public class AppkeyInfoLocalCache {
    
        /*在创建cache时指定所有key获取value的方法！！！*/
        static LoadingCache<String, AppkeyInfoBasic> cache = CacheBuilder.newBuilder().refreshAfterWrite(3, TimeUnit.HOURS)// 给定时间内没有被读/写访问，则回收。
                .expireAfterAccess(APIConstants.TOKEN_VALID_TIME, TimeUnit.HOURS)// 缓存过期时间和redis缓存时长一样
                .maximumSize(1000).// 设置缓存个数
                build(new CacheLoader<String, AppkeyInfoBasic>() {
                    @Override
                    /** 当本地缓存命没有中时，调用load方法获取结果并将结果缓存 **/
                    public AppkeyInfoBasic load(String appKey) throws DaoException {
                        return getAppkeyInfo(appKey);
                    }
    
                    /** 数据库进行查询 **/
                    private AppkeyInfoBasic getAppkeyInfo(String appKey) throws DaoException {
                        //从数据库取值
                        return ((AppkeyInfoMapper) SpringContextHolder.getBean("appkeyInfoMapper"))
                                .selectAppkeyInfoByAppKey(appKey);
                    }
                });
    
    }
    
2、Callable

    @Test
    public void testcallableCache() throws Exception{
        Cache<String, String> cache = CacheBuilder.newBuilder().maximumSize(1000).build();  
        
        //设置键jerry取值方式
        String resultVal = cache.get("jerry", new Callable<String>() {  
            public String call() {  
                String strProValue="hello "+"jerry"+"!";                
                return strProValue;
            }  
        });  
        System.out.println("jerry value : " + resultVal);
        
        //设置键peida取值方式
        resultVal = cache.get("peida", new Callable<String>() {  
            public String call() {  
                String strProValue="hello "+"peida"+"!";                
                return strProValue;
            }  
        });  
        System.out.println("peida value : " + resultVal);  
    }
    
## Mavan多模块

http://juvenshun.iteye.com/blog/305865?page=2#comments

所有用Maven管理的真实的项目都应该是分模块的，每个模块都对应着一个pom.xml。它们之间通过继承和聚合相互关联

### 简单的Maven模块结构

一个简单的Maven模块结构是这样的：
 
    app-parent
        |-- pom.xml (pom)
        |
        |-- app-util
        |        |-- pom.xml (jar)
        |
        |-- app-dao
        |        |-- pom.xml (jar)
        |
        |-- app-service
        |        |-- pom.xml (jar)
        |
        |-- app-web
                 |-- pom.xml (war)   
 
上述简单示意图中，有一个父项目(app-parent)聚合很多子项目（app-util, app-dao, app-service, app-web）。每个项目，不管是父子，**都含有一个pom.xml文件**。而且要注意的是，小括号中标出了每个项目的打包类型。**父项目是pom,也只能是pom**。**子项目有jar，或者war。**

### 多模块的好处

用项目层次的划分替代包层次的划分能给我们带来如下好处：
* **方便重用**，如果你有一个新的swing项目需要用到app-dao和app-service，添加对它们的依赖即可，你不再需要去依赖一个WAR。而有些模块，如app-util，完全可以渐渐进化成公司的一份基础工具类库，供所有项目使用。这是模块化最重要的一个目的。
* 由于你现在划分了模块，每个模块的配置都在各自的pom.xml里，不用再到一个混乱的纷繁复杂的总的POM中寻找自己的配置。
* 如果你只是在app-dao上工作，**你不再需要build整个项目**，只要在app-dao目录运行mvn命令进行build即可，这样可以节省时间，尤其是当项目越来越复杂，build越来越耗时后。
* 某些模块，如app-util被所有人依赖，但你不想给所有人修改，现在你完全可以从这个项目结构出来，做成另外一个项目，svn只给特定的人访问，但仍提供jar给别人使用
* 

## RPC

GRPC
Thrift

## Spring

### 获取ApplicationContext

需要将ApplicationContext保存至某个静态类中提供接口获取，那必须要在某一个地方将ApplicationContext设置进来。无法直接获取，因为获取ApplicationContext是需要上下文条件限制的。在Web环境下可以直接通过ServletContext获取。如果Bean对象本身被IoC容器所管理可直接通过注解获取。

方法一：非spring管理的类中要获得applicationcontext

    //获取ServletConetxt实例
    HttpServletRequest   httprequest = (HttpServletRequest)ServletActionContext.getRequest(); 
    ServletContext sc = httprequest.getSession().getServletContext(); 
    //使用 WebApplicationContextUtils.getWebApplicationContext(ServletContext)
    ApplicationContext  wac = WebApplicationContextUtils.getWebApplicationContext(sc); 

方法二：新建一个ApplicationContextHolder类，保存ApplicationContext实例

    @Autowired
    private ApplicationContext applicationContext;

方法三：在自己定义的bean中获得 applicationcontext，可以让该bean实现**applicationcontextaware**接口，记得把这个bean 配置在spring的配置文件里

    public class MyApplicationContext implements ApplicationContextAware { 
    
        private ApplicationContext applicationContext; 
        
        public void setApplicationContext(ApplicationContext ac) 
        throws BeansException { 
            this.applicationContext = ac; 
        } 
        
        public  ApplicationContext  getApplicationContext(){ 
            return applicationContext; 
        } 
    } 
    
spring 的配置如下 

    <bean  id="myapplicationcontext"  class="com.my.utils.MyApplicationContext"> 
    </bean> 

通过上面2种方式都可以获取到ApplicationContext如果楼主需要将ApplicationContext保存至某个静态类中提供接口获取，那必须要在某一个地方将ApplicationContext设置进来。无法直接获取。因为获取ApplicationContext是需要上下文条件限制的。在Web环境下可以直接通过ServletContext获取。如果Bean对象本身被IoC容器所管理可直接通过注解获取。

## 翻墙工具

ShadowSocket
lantern

## 全局gitignore
$ git config --global core.excludesFile /c/gitignore
$ git config --list

## 测试环境

cd /opt/nuc_git/bp-salt-deploy/
服务器ip：10.16.5.231
账号：root
密码：LXRoxqIqV4

---
title: RedisTemplate 使用及基本原理

date: 2017-05-06 11:24:00

categories:
- Redis

tags:
- Redis

---

## RedisTemplate 配置

在 Spring 配置文件中，配置的对象有：

* 连接池配置对象
* Jedis连接工厂，提供 getConnection 方法从连接池中获取连接
* RedisTemplate，Redis操作类

思路：1、首先必须有 Redis 操作类 2、然后为了提高性能，加入 Redis 连接池，调用连接池的 getConnection 方法获得连接

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
        http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd"
       default-lazy-init="true">

    <context:component-scan base-package="com.mk.util.redis"/>

    <!-- 连接池配置 -->
    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <!-- 连接池中最大空闲的连接数 -->
        <property name="maxIdle" value="${jedis.maxIdle}"></property>
        <!-- 连接空闲的最小时间，达到此值后空闲连接将可能会被移除。负值(-1)表示不移除. -->
        <property name="minEvictableIdleTimeMillis" value="${jedis.minEvictableIdleTimeMillis}"></property>
        <!-- 对于“空闲链接”检测线程而言，每次检测的链接资源的个数。默认为3 -->
        <property name="numTestsPerEvictionRun" value="${jedis.numTestsPerEvictionRun}"></property>
        <!-- “空闲链接”检测线程，检测的周期，毫秒数。如果为负值，表示不运行“检测线程”。默认为-1. -->
        <property name="timeBetweenEvictionRunsMillis" value="${jedis.timeBetweenEvictionRunsMillis}"></property>
    </bean>

    <!-- Spring提供的Redis连接工厂 -->
    <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory" destroy-method="destroy">
        <!-- 连接池配置 -->
        <property name="poolConfig" ref="jedisPoolConfig"></property>
        <!-- Redis服务主机 -->
        <property name="hostName" value="${redis.hostName}"></property>
        <!-- Redis服务端口号 -->
        <property name="port" value="${redis.port}"></property>
        <!-- 连超时设置 -->
        <property name="timeout" value="${redis.timeout}"></property>
        <!-- 是否使用连接池 -->
        <property name="usePool" value="${redis.usePool}"></property>
        <!-- Redis服务连接密码 -->
        <!--<property name="password" value="${redis.password}"></property>-->
    </bean>

    <!-- Spring提供的访问Redis类 -->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <!-- Redis连接工厂 -->
        <property name="connectionFactory" ref="jedisConnectionFactory"></property>
        <property name="keySerializer">
            <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
        </property>
        <!-- JdkSerializationRedisSerializer支持对所有实现了Serializable的类进行序列化 -->
        <property name="valueSerializer">
            <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"/>
        </property>
    </bean>


    <!--下面是自己编写的监听器，用来实现发布订阅功能的，对于基础的redis使用不需要-->
    <bean id="redisMessageListener" class="com.mk.util.redis.listener.RedisMessageListener"/>
    <bean  class="org.springframework.data.redis.listener.RedisMessageListenerContainer"
          destroy-method="destroy">
        <property name="connectionFactory" ref="jedisConnectionFactory"/>
        <property name="taskExecutor">
            <bean class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
                <property name="corePoolSize" value="5"/>
            </bean>
        </property>
        <property name="messageListeners">
            <map>
                <entry key-ref="redisMessageListener">
                    <!--这里配置频道信息-->
                    <bean class="org.springframework.data.redis.listener.ChannelTopic">
                        <constructor-arg value="notify"/>
                    </bean>
                </entry>
            </map>

        </property>
    </bean>
</beans>
```

## Redis操作工具类

提供了set/get方法，删除缓存，缓存定时删除方法

```
public class RedisUtils {
    @Autowired
    private RedisTemplate<String,Object> redisTemplate;

    public void set(String key, Object val) {
        redisTemplate.boundValueOps(key).set(val);
    }

    public Object get(String key) {
        return redisTemplate.boundValueOps(key).get();
    }

    public void delete(String key) {
        redisTemplate.delete(key);
    }

    public void expire(String key, long timeout){
        //单位为秒
        redisTemplate.expire(key, timeout, TimeUnit.SECONDS);
    }

    /**
     * 发布消息
     *
     * @param channel 发布的频道，需要在redis配置文件中进行配置
     * @param message
     */
    public void sendMessage(Serializable message) {
        sendMessage("notify",message);
    }
    public void sendMessage(String channel, Serializable message) {
        redisTemplate.convertAndSend(channel,message);
    }
}
```

## RedisTemplate源码

1、获得连接
2、调用action.doRedis
3、连接创建动态代理
4、释放连接

序列化：
String channerl => byte[]
Object message

## Redis 订阅和发布

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息
Redis 客户端可以订阅任意数量的频道。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![](http://i.imgur.com/7SNznwa.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![](http://i.imgur.com/RZt3oFl.png)

### redis 命令

1、创建频道名 redisChat

	redis 127.0.0.1:6379> SUBSCRIBE redisChat
	
	Reading messages... (press Ctrl-C to quit)
	1) "subscribe"
	2) "redisChat"
	3) (integer) 1

2、发送消息

现在，我们先重新开启个 redis 客户端，然后在同一个频道 redisChat 发布两次消息，订阅者就能接收到消息。

	redis 127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
	
	(integer) 1
	
	redis 127.0.0.1:6379> PUBLISH redisChat "Learn redis by runoob.com"
	
	(integer) 1
	
	# 订阅者的客户端会显示如下消息
	1) "message"
	2) "redisChat"
	3) "Redis is a great caching technique"
	1) "message"
	2) "redisChat"
	3) "Learn redis by runoob.com"

### RedisTemplate 发布订阅

![](http://i.imgur.com/LnxjzkL.png)

1、发送sub指令

Spring启动时加载配置文中配置的一个个bean，**RedisMessageListenerContainer中配置事件监听器及当事件来临时做出的响应**

    <bean id="redisMessageListener" class="com.mk.util.redis.listener.RedisMessageListener"/>
    <bean  class="org.springframework.data.redis.listener.RedisMessageListenerContainer"
          destroy-method="destroy">
        <property name="connectionFactory" ref="jedisConnectionFactory"/>
        <property name="taskExecutor">
            <bean class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
                <property name="corePoolSize" value="5"/>
            </bean>
        </property>
        <property name="messageListeners">
            <map>
                <!--可以配置多个订阅者(listener)，这里只配置一个-->
                <entry key-ref="redisMessageListener">
                    <!--这里配置频道信息-->
                    <bean class="org.springframework.data.redis.listener.ChannelTopic">
                        <constructor-arg value="notify"/>
                    </bean>
                </entry>
            </map>

        </property>
    </bean>

其中自定义的 RedisMessageListener 实现 MessageListener 接口，这个由程序员实现，当某个特定消息来临时，该执行哪个操作

```
public class RedisMessageListener implements MessageListener {

    @Override
    public void onMessage(Message message, byte[] bytes) {
        logger.info("=============调用redis监听器onMessage====================");
    }
}
```

**加载 RedisMessageListenerContainer 类时，会调用setMessageListeners方法注入监听器**；

除了注入监听器，这个方法还完成了：

* **开启一个客户端与Redis通信的线程**（不然程序整个程序就阻塞了，所以要开新线程）
* 与Redis服务器建立长连接
* 向Redis服务器发送SUBSCRIBE命令
* 循环监听Redis服务器向客户端发送的消息，如果没有消息则阻塞线程；如果接收到一个消息，分派消息
* **启动一个新的线程，执行消息处理方法**

整个执行过程的大致源码过程（删除无关代码）：

RedisMessageListenerContainer：

1、Spring框架注入属性时，调用setMessageListeners方法

	public void setMessageListeners(Map<? extends MessageListener, Collection<? extends Topic>> listeners) {
		initMapping(listeners);
	}

2、启动监听

	private void initMapping(Map<? extends MessageListener, Collection<? extends Topic>> listeners) {
		//在发布者和订阅者中添加监听器
		if (!CollectionUtils.isEmpty(listeners)) {
			for (Map.Entry<? extends MessageListener, Collection<? extends Topic>> entry : listeners.entrySet()) {
				addListener(entry.getKey(), entry.getValue());
			}
		}

		//启动监听
		if (initialized) {
			start();
		}
	}

3、start中关键代码是lazyListen

	public void start() {
		lazyListen();
	}

4、lazyListen中使用线程池开启一个线程用来客户端与redis服务器通信，并进行消息分派

	private void lazyListen() {
		subscriptionExecutor.execute(subscriptionTask);
	}

5、执行SubscriptionTask的run方法

	private class SubscriptionTask implements SchedulingAwareRunnable {
		public void run() {
					SubscriptionPresentCondition subscriptionPresent = eventuallyPerformSubscription();
		}
	}

6、调用connection的subscribe方法
	
	private SubscriptionPresentCondition eventuallyPerformSubscription() {
	
		connection.subscribe(new DispatchMessageListener(), unwrap(channelMapping.keySet()));
	
	}

执行JedisConnection类的方法：

7、调用JedisConnection的subscribe方法，将Spring中配置MessageListener参数传入

	public void subscribe(MessageListener listener, byte[]... channels) {
	BinaryJedisPubSub jedisPubSub = new JedisMessageListener(listener);
		jedis.subscribe(jedisPubSub, channels);
	}

执行BinaryJedis类的方法：

8、上层传入的MessageListener被封装在BinaryJedisPubSub中，执行jedisPubSub的proceed方法

	public void subscribe(BinaryJedisPubSub jedisPubSub, byte[]... channels) {
		jedisPubSub.proceed(client, channels);
	}

执行BinaryJedisPubSub类的方法：

9、**核心代码**

* 调用client的subscribe方法进行订阅，包括打开Socket与Redis Server建立长连接，并发送SUBSCRIBE命令
* process为消息分派

	public void proceed(Client client, byte[]... channels) {
		this.client = client;
		client.subscribe(channels);
		client.flush();
		process(client);
	}

10、消息分派

do——while的结构

其中：List<Object> reply = client.getRawObjectMultiBulkReply();从底层读取服务器向客户端发送的消息，如果没有消息则阻塞，否则进行消息分派

```
private void process(Client client) {
	do {
		List<Object> reply = client.getRawObjectMultiBulkReply();
		final Object firstObj = reply.get(0);
		if (!(firstObj instanceof byte[])) {
			throw new JedisException("Unknown message type: " + firstObj);
		}
		final byte[] resp = (byte[]) firstObj;
		if (Arrays.equals(SUBSCRIBE.raw, resp)) {
			subscribedChannels = ((Long) reply.get(2)).intValue();
			final byte[] bchannel = (byte[]) reply.get(1);
			onSubscribe(bchannel, subscribedChannels);
		} else if (Arrays.equals(UNSUBSCRIBE.raw, resp)) {
			subscribedChannels = ((Long) reply.get(2)).intValue();
			final byte[] bchannel = (byte[]) reply.get(1);
			onUnsubscribe(bchannel, subscribedChannels);
		} else if (Arrays.equals(MESSAGE.raw, resp)) {
			final byte[] bchannel = (byte[]) reply.get(1);
			final byte[] bmesg = (byte[]) reply.get(2);
			onMessage(bchannel, bmesg);
		} else if (Arrays.equals(PMESSAGE.raw, resp)) {
			final byte[] bpattern = (byte[]) reply.get(1);
			final byte[] bchannel = (byte[]) reply.get(2);
			final byte[] bmesg = (byte[]) reply.get(3);
			onPMessage(bpattern, bchannel, bmesg);
		} else if (Arrays.equals(PSUBSCRIBE.raw, resp)) {
			subscribedChannels = ((Long) reply.get(2)).intValue();
			final byte[] bpattern = (byte[]) reply.get(1);
			onPSubscribe(bpattern, subscribedChannels);
		} else if (Arrays.equals(PUNSUBSCRIBE.raw, resp)) {
			subscribedChannels = ((Long) reply.get(2)).intValue();
			final byte[] bpattern = (byte[]) reply.get(1);
			onPUnsubscribe(bpattern, subscribedChannels);
		} else {
			throw new JedisException("Unknown message type: " + firstObj);
		}
	} while (isSubscribed());
}
```

参考：
jedis的publish/subscribe[转]含有redis源码解析
http://www.cnblogs.com/wxh04/p/4301291.html

菜鸟教程 Redis 发布订阅
http://www.runoob.com/redis/redis-pub-sub.html
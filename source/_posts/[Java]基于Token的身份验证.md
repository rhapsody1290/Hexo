---
title: 基于 Token 的身份验证

date: 2016-10-11 10:50:00

categories:
- Java

tags:
- Token

---

## 为什么要身份验证

可能你会说，不是有登录接口吗，输入用户名、密码，身份验证通过就可以跳到下一个页面，很简单啊！

但是，HTTP 是一种没有状态的协议，也就是它并不知道用户是否已经登录验证过。这里我们把用户看成是客户端（可以是浏览器、Android、IOS等），客户端使用用户名还有密码通过了身份验证，不过下回这个客户端再发送请求访问其他服务器接口的时候，还得再验证一下，<font color='red'>**否则的话用户就可以跳过登录页面，直接访问其他服务器接口，登录就失去了意义**</font>

## 传统身份验证方法

Cookie+Session的存在主要是为了解决HTTP这一无状态协议下服务器如何识别用户的问题。当用户请求登录的时候，如果验证通过，<font color='red'>**在服务端创建一个session**</font>，session中记录一下登录的用户信息，<font color='red'>**然后把这个session的 sessionid 号发送给客户端**</font>，客户端收到以后把这个 sessionid 号存储在 Cookie 里，<font color='red'>**下次这个用户再向服务端发送请求的时候带着这个 Cookie**</font> ，这样服务端会根据 Cookie 里的sessionid恢复session，<font color='red'>**看看session中是否存有用户信息**</font>，如果是，说明用户已经通过了身份验证，就把用户请求的数据返回给客户端

上面说的就是 Session，我们需要在服务端存储为登录的用户生成的 Session ，这些 Session 可能会存储在内存，磁盘，或者数据库里。我们可能需要在服务端创建一个守护程序定期的去清理过期的 Session

## 基于 Token 的身份验证方法

使用基于 Token 的身份验证方法，**在服务端不需要存储用户的登录记录**。大概的流程是这样的：

1、客户端使用用户名和密码请求登录
2、服务端收到请求，去验证用户名与密码
3、验证成功后，服务端会签发一个 Token，再把这个 Token 发送给客户端
4、客户端收到 Token 以后可以把它存储起来，比如放在 Cookie 里或者 Local Storage 里
5、客户端每次向服务端请求资源的时候需要带着服务端签发的 Token
6、服务端收到请求，然后去验证客户端请求里面带着的 Token，如果验证成功，就向客户端返回请求的数据

比起传统的身份验证方法，Token 扩展性更强，也更安全点，非常适合用在 Web 应用或者移动应用上

## JWT

实施 Token 验证的方法挺多的，还有一些标准方法，比如 JWT，读作：jot，表示：JSON Web Tokens 。JWT 标准的 Token 有三个部分：

* header
* payload
* signature

**中间用点分隔开**，并且都会使用 Base64 编码，所以真正的 Token 看起来像这样：

<pre>
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9<font color='red' style='font-weight:bold'>.</font>eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ<font color='red' style='font-weight:bold'>.</font>SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc
</pre>

### header

**header** 部分主要是两部分内容，一个是 Token 的类型，另一个是使用的算法，比如下面类型就是 JWT，使用的算法是 HS256。

	{
		"typ": "JWT",
		"alg": "HS256"
	}

上面的内容要用 Base64 的形式编码一下，所以就变成这样：

	eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
	
### Payload

Payload 里面是 Token 的具体内容，这些内容里面有一些是标准字段，你也可以添加其它需要的内容。下面是标准字段：

* iss：Issuer，发行者
* sub：Subject，主题
* aud：Audience，观众
* exp：Expiration time，过期时间
* nbf：Not before
* iat：Issued at，发行时间
* jti：JWT ID

比如下面这个 Payload ，用到了 iss 发行人，还有 exp 过期时间。另外还有两个自定义的字段，一个是 name ，还有一个是 admin 。

	{
	    "iss": "ninghao.net",
	    "exp": "1438955445",
	    "name": "wanghao",
	    "admin": true
	}

使用 Base64 编码以后就变成了这个样子：

	eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ

### Signature

JWT 的最后一部分是 Signature ，这部分内容有三个部分，先是用 Base64 编码的 header.payload ，再用加密算法加密一下，加密的时候要放进去一个 Secret ，这个相当于是一个密码，**这个密码秘密地存储在服务端**。

* header
* payload
* secret


	var encodedString = base64UrlEncode(header) + "." + base64UrlEncode(payload); 
	HMACSHA256(encodedString, 'secret');

处理完成以后看起来像这样：

	SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc

最后这个在服务端生成并且要发送给客户端的 Token 看起来像这样：

	eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJuaW5naGFvLm5ldCIsImV4cCI6IjE0Mzg5NTU0NDUiLCJuYW1lIjoid2FuZ2hhbyIsImFkbWluIjp0cnVlfQ.SwyHTEx_RQppr97g4J5lKXtabJecpejuef8AqKYMAJc

客户端收到这个 Token 以后把它存储下来，下回向服务端发送请求的时候就带着这个 Token 。服务端收到这个 Token ，然后进行验证，通过以后就会返回给客户端想要的资源

## JWT库下载

官网下载
https://jwt.io/#libraries-io

### JJWT 

#### 依赖

	<dependency>
	    <groupId>io.jsonwebtoken</groupId>
	    <artifactId>jjwt</artifactId>
	    <version>0.7.0</version>
	</dependency>

#### 生成token

	import io.jsonwebtoken.Jwts;
	import io.jsonwebtoken.SignatureAlgorithm;
	import io.jsonwebtoken.impl.crypto.MacProvider;
	import java.security.Key;
	
	// We need a signing key, so we'll create one just for this example. Usually the key would be read from your application configuration instead.
	Key key = MacProvider.generateKey();
	
	String compactJws = Jwts.builder()
	  .setSubject("Joe")
	  .signWith(SignatureAlgorithm.HS512, key)
	  .compact();

#### 校验token

1、判断JWT是否有效
2、JWT中个数据是否正确，比如判断Subject是否为Joe

	try {
	
	    Jwts.parser().setSigningKey(key).parseClaimsJws(compactJws);	
	    //OK, we can trust this JWT
	
	} catch (SignatureException e) {
	
	    //don't trust the JWT!
	}


## 参考文件

基于 Token 的身份验证(http://ninghao.net/blog/2834)




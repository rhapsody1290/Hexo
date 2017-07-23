---
title: 消息转换机制

date: 2017-04-11 22:06:00

categories:
- Spring MVC

tags:
- 消息转换

---

源码分析：
http://www.cnblogs.com/fangjian0423/p/springMVC-xml-json-convert.html


HttpMessageConverter接口就是Spring提供的http消息转换接口

![](http://i.imgur.com/UXOcuk9.png)

1、对Http请求报文进行抽象，通过HttpInputMessage的getBody()方法获得输入流，通过HttpOutputMessag的getBody()方法获得输出流
2、消息转换器的最高层次的接口抽象：HttpMessageConverter
3、根据@RequestBody注解选择适当的HttpMessageConverter实现类来将请求参数解析到string变量中


开启注解驱动：`<mvc:annotation-driven/>`后实例化了RequestMappingHandlerMapping，RequestMappingHandlerAdapter等诸多类。

RequestMappingHandlerMapping处理请求映射的，处理@RequestMapping跟请求地址之间的关系。

RequestMappingHandlerAdapter是请求处理的适配器，将会处理注解和准备handler方法中的参数（RequestMappingHandlerAdapter中的HttpMessageConverter完成Http请求体中的数据到Handler参数的映射）




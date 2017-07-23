---
title: Fastjson使用指南.

date: 2017-03-01 10:09:00

categories:
- Java

tags:
- Fastjson

---

Fastjson API入口类是com.alibaba.fastjson.JSON，常用的序列化操作都可以在JSON类上的静态方法直接完成。

```
public static final Object parse(String text); // 把JSON文本parse为JSONObject或者JSONArray 
public static final JSONObject parseObject(String text)； // 把JSON文本parse成JSONObject    
public static final <T> T parseObject(String text, Class<T> clazz); // 把JSON文本parse为JavaBean 
public static final JSONArray parseArray(String text); // 把JSON文本parse成JSONArray 
public static final <T> List<T> parseArray(String text, Class<T> clazz); //把JSON文本parse成JavaBean集合 
public static final String toJSONString(Object object); // 将JavaBean序列化为JSON文本 
public static final String toJSONString(Object object, boolean prettyFormat); // 将JavaBean序列化为带格式的JSON文本 
public static final Object toJSON(Object javaObject); 将JavaBean转换为JSONObject或者JSONArray。
```

Demo

```
//1、反序列化，JSON字符串->对象
String recvObject = "{\"name\":\"qm\",\"password\":\"123\"}";
String recvArray = "[\"qm\",\"tr\",\"hj\"]";

JSONObject jsonObject = JSON.parseObject(recvObject);
JSONArray jsonArray = JSON.parseArray(recvArray);

    //JSONObject、JSONArray操作
System.out.println(jsonObject.getString("name") + " " + jsonObject.getString("password"));
System.out.println(jsonArray.getString(0) + " " + jsonArray.getString(1) + " " + jsonArray.getString(2));

//2、序列化，对象(数组，List，Map，bean等)->JSON字符串
String sendObject = JSON.toJSONString(jsonObject);
String sendArray = JSON.toJSONString(jsonArray);
```

更详细的Fastjson使用方法

http://www.cnblogs.com/Jie-Jack/p/3758046.html
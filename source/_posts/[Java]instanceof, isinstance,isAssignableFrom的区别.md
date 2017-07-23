---
title: instanceof, isinstance,isAssignableFrom的区别

date: 2016-11-24 21:38:00

categories:
- Java

tags:
- Java

---

## 类-实例

instanceof运算符只被用于对象引用变量，检查左边的被测试对象是不是右边类或接口的实例化。如果被测对象是null值，则测试结果总是false。

形象地：自身实例或子类实例 instanceof 自身类返回true 
例： String s=new String("javaisland"); 
	System.out.println(s instanceof String); //true 

## Class-实例

Class类的isInstance(Object obj)方法，obj是被测试的对象，如果obj是调用这个方法的class或接口 的实例，则返回true。这个方法是instanceof运算符的动态等价。 
形象地：自身类.class.isInstance(自身实例或子类实例)返回true 
例：String s=new String("javaisland"); 
	System.out.println(String.class.isInstance(s)); //true 

## Class-Class

Class类的isAssignableFrom(Class cls)方法，如果调用这个方法的class或接口 与 参数cls表示的类或接口相同，或者是参数cls表示的类或接口的父类，则返回true。 
形象地：自身类.class.isAssignableFrom(自身类或子类.class)返回true 
例：System.out.println(ArrayList.class.isAssignableFrom(Object.class));  //false 
	System.out.println(Object.class.isAssignableFrom(ArrayList.class));  //true

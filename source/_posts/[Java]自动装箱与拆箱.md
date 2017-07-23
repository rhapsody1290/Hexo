---
title: 自动装箱与拆箱（张孝祥补充）

date: 2016-09-06 17:07:00

categories:
- Java

tags:
- Java

---


## Integer与int比较

* Integer.valueof() 返回的是Integer的对象
* Integer.parseInt() 返回的是一个int的值
* new Integer.valueof().intValue();返回的也是一个int的值

### 不同类型比较

	public static void main(String[] args) {  	  
		String a = "400";  
		String b = "400";  
		//int和Integer比较，Integer自动拆箱，结果true
		System.out.println(Integer.parseInt(a) == Integer.valueOf(b));
	}  

### 相同类型比较

***单字节（-128-127）的Integer*** 比较是直接作为基本类型比较，否则是对象比较

	Integer c = 100; 
	Integer d = 100; 
	Integer c1 = 200; 
	Integer d1 = 200; 
	System.out.println(c == d); //为true 
	System.out.println(c1 == d1);//为false 


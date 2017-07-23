---
title: list转string[]

date: 2016-09-30 00:00:00

categories:
- Java

tags:
- Java

---
## 方法一：单个元素转换

	//ArrayList
	ArrayList<String> array = new ArrayList<String>();
	array.add("aaa");
	array.add("bbb");
	array.add("ccc");
	//转化成Object数组
	Object[] objs = array.toArray();
	//新建数组
	String[] strings = new String[array.size()];
	//每个元素类型转换
	int i = 0;
	for(Object obj : objs){
		if(obj instanceof String){
			strings[i++] = (String)obj;
		}
	}
	for(String s : strings){
		System.out.println(s);
	}


## 方法二：整个转

使用List的toArray(T[])方法，传入一个与list容量相等的字符串数组

	//ArrayList
	ArrayList<String> array = new ArrayList<String>();
	array.add("aaa");
	array.add("bbb");
	array.add("ccc");
	
	String[] strings = new String[array.size()];
	array.toArray(strings);
	for(String s : strings){
		System.out.println(s);
	}

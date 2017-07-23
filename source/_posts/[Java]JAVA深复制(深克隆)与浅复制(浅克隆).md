---
title: JAVA深复制(深克隆)与浅复制(浅克隆)

date: 2016-07-29 00:00:00

categories:
- Java

tags:
- Java
- 深克隆与浅克隆

---

参考http://www.cnblogs.com/yxnchinahlj/archive/2010/09/20/1831615.html

## 浅复制与深复制概念

### *浅复制*

　　被复制的对象的成员变量与原来的对象都有相同的值，但引用其他对象的成员变量仍指向原来的对象，修改引用对象的值，会同时影响原对象与复制对象。换言之，**浅复制仅仅复制所考虑的对象，不复制它引用的对象**。
### *深复制*
　　被复制的对象的所有成员变量与原来对象都有相同的值，而且引用其他对象的成员变量将指向被复制过的新对象。换言之，**深复制把要复制的对象及所引用的对象都复制了一遍。**

## java.lang.Object类的clone()方法是浅复制

```
public class test{
	public static void main(String[] args){
		professor p = new professor(40);
		student s = new student("student", p);
		student new_s = (student)s.clone();
		
		new_s.p.age = 18;
		System.out.println(s);
		System.out.println(new_s);
	}
}

class student implements Cloneable{
	String name;
	professor p;
	public student(String name, professor p){
		this.name = name;
		this.p = p;
	}

	public Object clone(){
		Object o = null;
		try{
			o = super.clone();
		}catch(Exception e){
			e.printStackTrace();
		}
		
		return o;
	}
	@Override
	public String toString(){
		return name + " " + p.age;
	}
}
class professor{
	int age;
	public professor(int age){
		this.age = age;
	}
}
```

结果：

    student 18
    student 18

因为复制对象中成员变量引用的对象professor不变，当改变新复制对象的成员变量引用值，对原对象也会有影响。
![这里写图片描述](http://img.blog.csdn.net/20160504171513907)

## 实现深层次克隆方法1

　　在复制对象时，对引用的对象也进行复制
```
public class test{
	public static void main(String[] args){
		professor p = new professor(40);
		student s = new student("student", p);
		student new_s = (student)s.clone();
		
		new_s.p.age = 18;
		System.out.println(s);
		System.out.println(new_s);
	}
}

class student implements Cloneable{
	String name;
	professor p;
	public student(String name, professor p){
		this.name = name;
		this.p = p;
	}

	public Object clone(){
		Object o = null;
		try{
			o = super.clone();
			((student)o).p = (professor)this.p.clone();//核心
		}catch(Exception e){
			e.printStackTrace();
		}
		
		return o;
	}
	@Override
	public String toString(){
		return name + " " + p.age;
	}
}
class professor implements Cloneable{
	int age;
	public professor(int age){
		this.age = age;
	}
	public Object clone(){
		Object o = null; 
		try{
			o = super.clone();
		}catch(Exception e){
			e.printStackTrace();
		}
		return o;
	}
}
```

## 实现深层次克隆方法2

　　利用串行化来做深复制，在Java语言里深复制一个对象，常常可以先使对象实现Serializable接口，然后把对象（实际上只是对象的一个拷贝）写到一个流里，再从流里读出来，便可以重建对象。
```
import java.io.*;
public class test{
	public static void main(String[] args) throws Exception{
		professor p = new professor(40);
		student s = new student("student", p);
		student new_s = (student)s.deepClone();

		new_s.p.age = 18;
		System.out.println(s);
		System.out.println(new_s);
	}
}
class student implements Serializable{
	String name;
	professor p;
	public student(String name, professor p){
		this.name = name;
		this.p = p;
	}
	public Object deepClone() throws Exception{
		//将对象写到流里
		ByteArrayOutputStream bo = new ByteArrayOutputStream();
		ObjectOutputStream oo = new ObjectOutputStream(bo);
		oo.writeObject(this);
		//从流里读出来
		ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
		ObjectInputStream oi = new ObjectInputStream(bi);
		return oi.readObject();
	}
	@Override
	public String toString(){
		return name + " " + p.age;
	}
}
class professor implements Serializable{
	int age;
	public professor(int age){
		this.age = age;
	}
}
```
结果：

    student 40
    student 18
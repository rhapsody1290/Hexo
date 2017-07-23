---
title: Java中关于父子类之间this关键字的一个问题

date: 2017-03-02 17:50:00

categories:
- Java

tags:
- this

---
父类方法中使用this，那么这个this指的是谁？
http://blog.csdn.net/sinat_31311947/article/details/50619467

java中关于父子类之间this关键字的一个问题
https://my.oschina.net/mlongbo/blog/90047

Parent类：

```
public class Parent {
	public Parent(){
		System.out.println(this.getClass());
	}
	public void info(){
		System.out.println(this);
	}
}
```

Child类：

```
public class Child extends Parent {
}
```

测试类：

```
public class Test {
	public static void main(String[] args) {
		Parent p = new Parent();
		p.info();
		System.out.println();
		Child c = new Child();
		c.info();
	}
}
```

Child类继承Parent类，在Parent类的info方法中会输出对象的内存地址。

	class demo.test.Parent
	demo.test.Parent@544a5ab2
	
	class demo.test.Child
	demo.test.Child@2e6e1408



结论：

* this指的是当前对象
* 在new子类的时候，<font color='red'>**会先调用父类的构造方法。但是只会new一个子类对象，不会new父类对象**</font>，所以**在继承关系中只有子类一个对象，this也就指的是子类对象**
* 在继承关系中，this指的是子类，或者说谁调用的方法就是指谁。多重继承也是如此。
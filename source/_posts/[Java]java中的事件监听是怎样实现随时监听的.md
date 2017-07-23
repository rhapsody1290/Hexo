---
title: java中的事件监听是怎样实现随时监听的

date: 2016-07-29 00:00:00

categories:
- Java

tags:
- Java
- 事件监听

---
### github

https://github.com/rhapsody1290/monitor

### 事件监听机制
![这里写图片描述](http://img.blog.csdn.net/20160420164724788)
　　Java中的事件监听是整个Java消息传递的基础和关键。牵涉到三类对象：事件源（Event Source）、事件（Event）、事件监听器（Event Listener）。
　　● 事件源是事件发生的场所，通常就是各个组件，它可以是一个按钮，编辑框等。
　　● 事件监听者负责监听事件源所发生的事件，并对各种事件做出相应的响应。
　　● 事件是描述事件源状态改变的对象。
　　具体实现呢，可以看看Button的源码。可能不好看得懂。那好我们仿照侯捷先生的作法，来模拟一个这样的事件传递： 

### 定义一个自己的事件
将事件源中value的最新值告知监听器

	public class MyEvent {
		private int value;
	
		public int getValue() {
			return value;
		}
	
		public void setValue(int value) {
			this.value = value;
		}
	}

### 做一个监听器接口 Listener
当外部响应触发事件源上的事件时，产生一个事件对象，该事件对象会作为参数传递给监听器的事件处理方法

	public interface Listener {
		public void valueChanged(MyEvent e);
	}

 
### 做一个事件发生者
* 当事件源中的value值发生改变时，会促发事件
* 监听器在事件源上注册，事件源会保存该监听器，在事件触发时调用监听器的事件处理方法


	public class MySource {
		private int value;
		private Vector<Listener> listeners = new Vector<Listener>();
		/**
		 * 添加监听器
		 * @param listener
		 */
		public void addListener(Listener listener){
			listeners.add(listener);
		}
		
		public void setValue(int value){
			this.value = value;
			//发送消息
			MyEvent e = new MyEvent();
			e.setValue(value);
			for(int i = 0; i < listeners.size(); i++){
				listeners.get(i).valueChanged(e);
			}
		}
	
	}

### 注册监听器

* 如果想监听事件源中value值改变，就在事件源那儿注册一下监听器，然后写消息处理代码就可以了，一般使用匿名内部类的方式定义监听器
* 这样，当MySource的value真的改变时，就会触发响应


	public class Main {
	
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			MySource mySource = new MySource();
			mySource.addListener(new Listener() {
				public void valueChange(MyEvent e) {
				    System.out.println("值改变了：" + e.getValue());
				}
			});
			mySource.setValue(1);
		}
	
	}

	#结果
	value changed to:10

Java中AWT/Swing的事件传递的实现，现在版本于上述有所不同，但应该都是这个原理。

### 总结[图解]★★★★★★

![](http://i.imgur.com/JU0I5Nn.png)

#### 建议开发顺序：
* 先编写事件源，事件源中有监听器集合Vector<Listener> listeners和增加监听器方法addListener
* 在触发事件源的方法上如setValue，产生事件、并调用监听器方法，将事件以参数传入监听器方法
* 创建事件类和监听器类
* 测试，创建事件源——添加监听器——触发事件

#### 思考
***监听器本质***

* 当调用setValue时，使得value属性的值发生变化，产生事件并调用监听器中对应该属性值改变时做出的处理；而事件不触发时不会产生这个响应，这就起到***对一个属性值监控*** 的作用
* 使用监听器有什么好处呢？我们可以直接在setValue这个函数中做出响应啊！但是如果直接在serValue中写，这个响应处理不能由程序员自己控制，这是写死的。而采用监听器方法时，通过重写Listener中的事件处理函数，***程序员可以自己编写事件处理函数***
* <font color='red'>函数调用顺序是：外界动作调用setValue方法，在这个而方法中去调用监听器中对应的事件处理函数</font>

### 参考文献
[1]. http://www.jcodecraeer.com/a/chengxusheji/java/2012/0822/371.html

---
title: 10、全排列（DFS+剪枝）

date: 2017-06-28 09:17:00

categories:
- 数据结构与算法

tags:
- 算法
- 数据结构

---

## 题目：全排列

例如ABC，全排列的结果是ABC、ACB、BAC、BCA、CAB、CBA

## 递归穷举

1、穷举，遍历出所有结果。因为对于长度为n的字符串，**需要n层for循环，代码没法写**，所以需要利用递归实现

	for(int i = 0; i < str.length(); i++){
		for(int i = 0; i < str.length(); i++){
			for(int i = 0; i < str.length(); i++){
				.....长度为n的字符串需要写n个for，代码没法写了
			}
		}
	}



写成递归的形式：

	public void DFS(int step) {
		
		//递归结束
	    if(step >= str.length()){
	        return;
	    }
	
	    for (int i = 0; i < str.length(); i++) {
			System.out.println(str.charAt(i));
	        DFS(step + 1);//进去都是for循环，和多个嵌套for一样，要注意结束点
	    }
	}

遍历结果：

a、a、a、b、c、b、a、b、c、c、a、b、c
b、a、a、b、c、b、a、b、c、c、a、b、c
c、a、a、b、c、b、a、b、c、c、a、b、c

参照画的图，结果一致：

![](http://i.imgur.com/9IPlcrP.jpg)

## 记录路径

还是以循环的形式考虑，这个很容易理解

	List<Integer> path = new LinkedList<Integer>();
	for (int i = 0; i < str.length(); i++) {
	    path.add(i);
	    for (int j = 0; j < str.length(); j++) {
	        path.add(j);
	        for (int k = 0; k < str.length(); k++) {
	            path.add(k);
	            //输出
	            System.out.println(path);
	            path.remove(path.size() - 1);
	        }
	        path.remove(path.size() - 1);
	    }
	    path.remove(path.size() - 1);
	}

输出结果：
	
	[0, 0, 0]
	[0, 0, 1]
	[0, 0, 2]
	[0, 1, 0]
	[0, 1, 1]
	[0, 1, 2]
	[0, 2, 0]
	[0, 2, 1]
	[0, 2, 2]
	[1, 0, 0]
	[1, 0, 1]
	[1, 0, 2]
	[1, 1, 0]
	[1, 1, 1]
	[1, 1, 2]
	[1, 2, 0]
	[1, 2, 1]
	[1, 2, 2]
	[2, 0, 0]
	[2, 0, 1]
	[2, 0, 2]
	[2, 1, 0]
	[2, 1, 1]
	[2, 1, 2]
	[2, 2, 0]
	[2, 2, 1]
	[2, 2, 2]

现在改成递归的方式应该很容易了吧，**你知道哪里应该add，哪里应该remove了**

	public class Permutation {
	
	    String str;
	    List<Integer> path = new LinkedList<Integer>();
	
	    public void DFS(int step) {
	
	        if (step >= str.length()) {
	            return;
	        }
	
	        for (int i = 0; i < str.length(); i++) {
	            path.add(i);
	            if(path.size() == str.length()){
	                System.out.println(path);
	            }
	            DFS(step + 1);
	            path.remove(path.size() - 1);
	        }
	    }
	
	    public static void main(String[] args) {
	        String str = "abc";
	        Permutation s = new Permutation();
	        s.str = str;
	        s.DFS(0);
	    }
	
	}

有个问题：输出是放在DFS（step + 1）前还是后面？

	if(path.size() == str.length()){
	    System.out.println(path);
	}

如果到达**最后一层**循环，这个DFS（step+1）相当于**空函数！！**，所以**输出放在DFS(step + 1)前或者后都一样**

	for (int i = 0; i < str.length(); i++) {
	    //System.out.println(str.charAt(i));
	    path.add(i);
	    if(path.size() == str.length()){
	        System.out.println(path);
	    }
	    DFS(step + 1);
	    path.remove(path.size() - 1);
	}

## 剪枝

![](http://i.imgur.com/C15NY78.jpg)

红色描绘的路径是符合条件的全排列，观察后发现：

* **访问的路径节点之前已经存在**，那么就需要剪枝
* 路径刚才我们已经保存了，所以可以做到

加入节点前需要判断这个节点是否已经存在

	if(path.contains(i)){
	    continue;
	}

## 完整代码

递归完整版

	public class Permutation {
	
	    String str;
	    List<Integer> path = new LinkedList<Integer>();
	
	    public void DFS(int step) {
	
	        if (step >= str.length()) {
	            return;
	        }
	
	        for (int i = 0; i < str.length(); i++) {
	            if(path.contains(i)){
	                continue;
	            }
	            path.add(i);
	            //输出
	            if(path.size() == str.length()){
	                //System.out.println(path);
	                for(Integer t : path){
	                    System.out.print(str.charAt(t) + " ");
	                }
	                System.out.println();
	            }
	            DFS(step + 1);
	            path.remove(path.size() - 1);
	        }
	    }
	
	    public static void main(String[] args) {
	        String str = "abc";
	        Permutation s = new Permutation();
	        s.str = str;
	        s.DFS(0);
	    }
	
	}

循环完整版

	List<Integer> path = new LinkedList<Integer>();
	    for (int i = 0; i < str.length(); i++) {
	        if(path.contains(i)){
	            continue;
	        }
	        path.add(i);
	        for (int j = 0; j < str.length(); j++) {
	            if(path.contains(j)){
	                continue;
	            }
	            path.add(j);
	            for (int k = 0; k < str.length(); k++) {
	                if(path.contains(k)){
	                    continue;
	                }
	
	                path.add(k);
	                //输出
	                //System.out.println(path);
	                for(Integer t : path){
	                    System.out.print(str.charAt(t) + " ");
	                }
	                System.out.println();
	
	                path.remove(path.size() - 1);
	            }
	            path.remove(path.size() - 1);
	        }
	        path.remove(path.size() - 1);
	    }
	}

## 如果有重复

如果输入aac，将会输出

	a a c 
	a c a 
	a a c 
	a c a 
	c a a 
	c a a 

有重复，为什么？为了区分用a1，a2表示

下图为按照未重复的时候的结果进行遍历、剪枝，打钩的即为需要保留的元素，与程序运行的一致

![](http://i.imgur.com/vfH34t6.jpg)

下图中圈出来的路径需要被剪去，思考一下有什么特征？

![](http://i.imgur.com/jv61CWg.jpg)

	a2、a1、c
	a2、c、a1
	c、a2、a1

**如果之前路径有相同的元素（这里是判断值相等，刚才是判断下标是否相等），而且当前下标比之前的小，则剪去**

代码如下：

	//重复元素
	boolean flag = false;
	for(Integer t : path){
	    if(str.charAt(t) == str.charAt(i) && i < t){
	        flag = true;
	        break;
	    }
	}
	if(flag){
	    continue;
	}


完整代码：

	public class Permutation {
	
	    String str;
	    List<Integer> path = new LinkedList<Integer>();
	
	    public void DFS(int step) {
	
	        if (step >= str.length()) {
	            return;
	        }
	
	        for (int i = 0; i < str.length(); i++) {
	            if(path.contains(i)){
	                continue;
	            }
	
	            //重复元素
	            boolean flag = false;
	            for(Integer t : path){
	                if(str.charAt(t) == str.charAt(i) && i < t){
	                    flag = true;
	                    break;
	                }
	            }
	            if(flag){
	                continue;
	            }
	
	            path.add(i);
	            //输出
	            if(path.size() == str.length()){
	                //System.out.println(path);
	                for(Integer t : path){
	                    System.out.print(str.charAt(t) + " ");
	                }
	                System.out.println();
	            }
	            DFS(step + 1);
	            path.remove(path.size() - 1);
	        }
	    }
	
	    public static void main(String[] args) {
	        String str = "aaaa";
	        Permutation s = new Permutation();
	        s.str = str;
	        s.DFS(0);
	
	    }
	
	}
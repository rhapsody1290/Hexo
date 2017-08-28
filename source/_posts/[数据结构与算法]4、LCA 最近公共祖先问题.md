---
title: 4、LCA 最近公共祖先问题

date: 2017-05-26 10:03:00

categories:
- 数据结构与算法

tags:
- LCA

---

## 问题描述

最近公共祖先问题可以将树从下往上看，两个目标节点到树的根节点组成的两条链表，求两条链表第一次相遇的地方

如下图所示，求3，4的公共祖先，则目标节点到根的链表分别是：3——>1、4——>2——>1，两条链表第一次相遇的地方是1，那么最近公共祖先就是1

![](http://i.imgur.com/KLZozPV.png)

另外，4 和 5 的最近公共祖先是2，5和3的最近公共祖先是1，2和1的最近公共祖先是1

## 转化成求两个链表的第一个交点

思考：

1、深度优先搜索模板

注意递归结束条件

	public void DFS(TreeNode root) {
	    if (root == null) {
	        return;
	    }
	    DFS(root.left);
	    DFS(root.right);
	}

2、增加路径

* 第一次访问节点的时候将元素添加到栈中
* 访问完左右节点后，将该元素弹出
* 什么时候输出结果？访问到叶子节点时输出完整路径root.left=null&&root.right= null
* 输出结果放在什么位置？可以是第一次访问节点、访问左节点后、访问右节点后，一般在第一次访问时，在将元素放入栈之后输出


	public void DFS(TreeNode root, List<Integer> path, int key) {
	    if (root == null) {
	        return;
	    }
	    path.add(root.val);
	    if (root.left == null && root.right == null) {
	        System.out.println(path);
	    }
	    DFS(root.left, path, key);
	    DFS(root.right, path, key);
	    path.remove(path.size() - 1);
	}

3、如果找到路径后，停止搜索，输出路径

* 返回值：是否找到目标节点true/false
* 如果左边没有找到（返回false），继续搜索右边；如果左边找到了（返回true），不需要搜索右边。**停止递归的关键是：如果找到元素后，不再进行递归深入**

	boolean find = DFS(root.left, path, key) || DFS(root.right, path, key);

* 本来只要return find即可，但是对于true/false，对于true，不需要pop（）路径，直接return true；如果是false，则需要pop（）当前元素
* 为什么add不用根据返回值控制？只要递归不深入，不会调用add方法


	import java.util.ArrayList;
	import java.util.List;
	
	/**
	 * 树的结构
	 * 1
	 * /\
	 * 2 3
	 * / \ /\
	 * 4 5 6 7
	 */
	public class Solution {
	
	
	    public boolean DFS(TreeNode root, List<Integer> path, int key) {
	        if (root == null) {
	            return false;
	        }
	        path.add(root.val);
	        //如果找到目标节点后就不需要向下搜索，并且需要一个状态值向上传递，然每层循环都停止搜索
	        if (root.val == key) {
	            return true;
	        }
	        //左边必须搜索，左边如果搜索到了，则右边可以不搜索（关键）
	        boolean find = DFS(root.left, path, key) || DFS(root.right, path, key);
	        //如果找到了直接返回
	        if (find) {
	            return true;
	        } else {//没找到，需要将当前节点弹出，并且返回false
	            path.remove(path.size() - 1);
	            return false;
	        }
	    }
	
	    public static void main(String[] args) {
	        TreeNode t4 = new TreeNode(4, null, null);
	        TreeNode t5 = new TreeNode(5, null, null);
	        TreeNode t6 = new TreeNode(6, null, null);
	        TreeNode t7 = new TreeNode(7, null, null);
	        TreeNode t2 = new TreeNode(2, t4, t5);
	        TreeNode t3 = new TreeNode(3, t6, t7);
	        TreeNode root = new TreeNode(1, t2, t3);
	        Solution s = new Solution();
	        List<Integer> path = new ArrayList<Integer>();
	        s.DFS(root, path, 3);
	        System.out.println(path);
	    }
	
	}

<br/>
<font color='red'>**关键点：**</font>

* 找到之后返回true
* 如果左边找到了，不需要找右边
* 如果已经找到了，不需要remove路径；没找到需要移出路径

4、最近公共祖先

	import java.util.ArrayList;
	import java.util.List;
	
	/**
	 * 树的结构
	 * 1
	 * /\
	 * 2 3
	 * / \ /\
	 * 4 5 6 7
	 */
	public class Solution {
	
	    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
	        if (root == null || p == null || q == null) {
	            return null;
	        }
	
	        List<TreeNode> path1 = new ArrayList<TreeNode>();
	        List<TreeNode> path2 = new ArrayList<TreeNode>();
	
	        boolean find1 = DFS(root, path1, p);
	        boolean find2 = DFS(root, path2, q);
	
	        TreeNode result = null;
	        if (find1 && find2) {
	            int n = Math.min(path1.size(), path2.size());
	            for (int i = 0; i < n; i++) {
	                if (path1.get(i).val == path2.get(i).val) {
	                    result = path1.get(i);
	                }else{
	                    break;
	                }
	            }
	            return result;
	        } else {
	            return null;
	        }
	    }
	
	    //找路径
	    public boolean DFS(TreeNode root, List<TreeNode> path, TreeNode key) {
	        if (root == null) {
	            return false;
	        }
	        path.add(root);
	        if (root == key) {
	            return true;
	        }
	        boolean find = DFS(root.left, path, key) || DFS(root.right, path, key);
	        if (find) {
	            return true;
	        } else {
	            path.remove(path.size() - 1);
	            return false;
	        }
	
	    }
	
	    public static void main(String[] args) {
	        TreeNode t4 = new TreeNode(4, null, null);
	        TreeNode t5 = new TreeNode(5, null, null);
	        TreeNode t6 = new TreeNode(6, null, null);
	        TreeNode t7 = new TreeNode(7, null, null);
	        TreeNode t2 = new TreeNode(2, t4, t5);
	        TreeNode t3 = new TreeNode(3, t6, t7);
	        TreeNode root = new TreeNode(1, t2, t3);
	        Solution s = new Solution();
	        System.out.println(s.lowestCommonAncestor(root,t2,t4));
	    }
	
	}

## Tarjan(离线)算法

### 原理（最浅显易懂的描述）

![](http://i.imgur.com/BXMZV63.png)

用集合的角度去思考 LCA 问题：

10 与 1，2，5，6 的 LCA 都为 1
10 与 3，7 的 LCA 都为 3
10 与 8，9，11 的 LCA 都为 8
10 与 10，12 的 LCA 都为 12

Tarjan 算法可以批量查询，假设有求5，10的 LCA，3、10的LCA，9、10的LCA

当 DFS 遍历到节点 10 时，查看查询语句中是否有与10相关的：
如果求 5、10 的 LCA，只要求当前5所在的集合的祖先就行了，结果就是8
如果求 3、10 的 LCA，只要求3所在集合的祖先就行了，结果是3
如果求 9、10 的 LCA，只要求9所在结合的祖先就行了，结果是8

那么问题的关键就是 <font color='red'>**集合怎么表示，集合如何构建？**</font>

### 并查集

可以发现，当 **一个数A** 与 **集合中的元素B** 求 LCA 时，<font color='red'>结果都是指向同一个祖先</font>，我们将设置一个数据结构将各个节点**组成集合**，并且能够根据集合中的点**访问到祖先**，可以使用 **并查集** 满足上述要求 。并查集维护了**当前节点与父节点之间的关系，最终可以获得祖先节点**

**并查集是当前节点与father节点的映射关系**，是一个Key-Value的结构，可以用数组的下标和数组的值作为存储结构，按照上图左子树举个例子：

![](http://i.imgur.com/uP9FUsk.png)

初始化的时候每个节点的父节点都是自己，如果**当前节点的father就是自己本身**，那么他就是集合的根节点

并查集查集合根节点：

    public int find(int x) {
        int r = x;
        while(father[r] != r){
            r = father[r];
        }
        return r;
    }

并查集基本介绍完毕，现在可以用并查集来表示集合了，并且找到集合的根节点，接下来的问题，**哪些节点可以归为同一个集合？**

### 算法描述（思路）

![](http://i.imgur.com/BXMZV63.png)

还是之前那张图，对于新搜索到的一个结点u，先创建由u构成的集合，再对u的每颗子树进行搜索，每搜索完一棵子树，将子节点归到集合u中

1.任选一个点为根节点，从根节点开始。
2.遍历该点u所有子节点v，并标记这些子节点v已被访问过。
3.若是v还有子节点，返回2，否则下一步。
4.合并v到u上。
5.寻找与当前点u有询问关系的点v。
6.若是v已经被访问过了，则可以确认u和v的最近公共祖先为v被合并到的父亲节点a。

伪代码

	Tarjan(u)//marge和find为并查集合并函数和查找函数
	{
	    for each(u,v)    //访问所有u子节点v
	    {
	        Tarjan(v);        //继续往下遍历
	        marge(u,v);    //合并v到u上
	        标记v被访问过;
	    }
	    for each(u,e)    //访问所有和u有询问关系的e
	    {
	        如果e被访问过;
	        u,e的最近公共祖先为find(e);
	    }
	}

### 算法模拟

http://www.cnblogs.com/JVxie/p/4854719.html

### 代码实现

## OJ

http://www.cnblogs.com/JVxie/p/4854719.html





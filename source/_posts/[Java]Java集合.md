---
title: Java集合

date: 2016-07-29 00:00:00

categories:
- Java

tags:
- Java
- Java集合

---

## 集合框架--使用

![这里写图片描述](http://img.blog.csdn.net/20160416165357245)
### *开发常用*
　　●　在实际开发过程中，比较常用的是Vector、Stack、ArrayList、LinkedList、HashMap、HashTable
　　●　集合类基本上在util包中
### *集合分类*
　　Java集合类主要有以下几种
　 　●　List结构的集合类：ArrayList、LinkedList、Vector、Stack
　 　●　Map结构的集合类：HashMap、HashTable
　 　●　Set结构的集合类：HashSet、TreeSet
　 　●　Queue结构的集合：Queue接口

## List结构的集合类

### *ArrayList类的使用(无同步性，线程不安全)*

* 基于动态数组的数据结构，随机访问优于LinkedList
```
public class ArrayList

1. 往list中插入元素 
    ● boolean add(E e) 元素添加到列表的尾部 
    ● void    add(int index, E element) 元素插入到列表的指定位置
2. 返回列表中的元素数 
    ● int size() 返回列表中的元素数 
3. 如果此列表中没有元素，则返回 true 
    ● boolean isEmpty() 
4. 返回此列表中指定位置上的元素 
    ● E get(int index) 
5. 用指定的元素替代此列表中指定位置上的元素,返回值是以前位于该位上的元素 
    ● E set(int index, E element) 
6. 移除列表上的元素 
    ● E remove(int index) 移除此列表中指定位置上的元素 
    ● boolean remove(Object o) 移除此列表中首次出现的指定元素（如果存在） 
    ● protected void removeRange(int fromIndex, int toIndex) 移除列表中索引在 fromIndex（包括）和 toIndex（不包括）之间的所有元素。 
    ● void clear() 移除列表中所有元素
7. 查找元素 
    ● contains(Object o) 判断list中是否含有指定元素 
    ● int indexOf(Object o) 返回list中元素第一次出现的位置,若没有则返回-1 
    ● int lastIndexOf(Object o) 返回arraylist中元素最后一次出现的位置,若没有则返回-1
8. 返回包含此列表中所有元素的数组（按顺序），相当于数组API和collection API的桥梁,返回一个object的数组 
    ● Object[] toArray()
```

### *LinkedList类的使用*

* 基于链表的数据结构，新增add和删除remove操作优于ArrayList
```
1. 将指定元素插入此列表的开头
    ● void addFirst(E e) 
2. 将指定元素添加到此列表的结尾
    ● void addLast(E e)  
```

### *Vector同理(线程安全具有同步性)*
### *Stack*
```
1. 测试堆栈是否为空 
    boolean empty() 
2. 查看堆栈顶部的对象，但不从堆栈中移除它 
    E peek() 
3. 移除堆栈顶部的对象，并作为此函数的值返回该对象
    E pop() 
4. 把项压入堆栈顶部
    E push(E item) 
5. 返回对象在堆栈中的位置，以 1 为基数  
    int search(Object o) 
```
### ArrayList和Vector的区别
```
ArrayList和Vector的区别
    ArrayList与Vector都是java的集合类，都可以用来存放java对象，这是他们的相同点，但是他们也有区别：
1、同步性
    Vector是线程同步的。这个类中的一些方法保证了Vector中的对象是线程安全的。而ArrayList则是线程异步的，因此ArrayList中的对象并不是线程安全的。因为同步的要求会影响执行的效率，所以如果你不需要线程安全的集合那么使用ArrayList是一个很好的选择，这样可以避免由于同步带来的不必要的性能开销。
2、数据增长
    从内部实现机制来讲ArrayList和Vector都是使用数组(Array)来控制集合中的对象。当你向这两种类型中增加元素的时候，如果元素的数目超出了内部数组目前的长度它们都需要扩展内部数组的长度，Vector缺省情况下自动增长原来一倍的数组长度，ArrayList是原来的50%，所以最后你获得的这个集合所占的空间总是比你实际需要的要大。所以如果你要在集合中保存大量的数据那么使用Vector有一些优势，因为你可以通过设置集合的初始化大小来避免不必要的资源开销。
```

## Map结构的集合类

### *HashMap（线程不同步），HashTable（线程同步）集合类*

```
void clear() 
      从此映射中移除所有映射关系。 
V get(Object key) 
      返回指定键所映射的值；如果对于该键来说，此映射不包含任何映射关系，则返回 null。 
boolean isEmpty() 
      如果此映射不包含键-值映射关系，则返回 true。 
V put(K key, V value) 
      在此映射中关联指定值与指定键。 
V remove(Object key) 
      从此映射中移除指定键的映射关系（如果存在）。 
int size() 
      返回此映射中的键-值映射关系数。 
```
### *Map遍历*
    1. Map.Entry<String, String>的数据类型为键值对。
    
    2. map.entrySet(),返回类型为Set<Entry<String, String>> ，即map中非空元素组成的键值对Set集合，可以用图例（key1,value1）,(key2,value2)...(keyn,valuen)表示。
    
    3. Map.Entry<String, String>类型分别通过getKey()、getValue()方法取出键和值。
    
    4. 可以通过map.keySet()方法取出全部键的Set集合，从而通过map.get(Object key)方法取出值，即方法一。
    
    5. 可以通过map.value()得到Collection<String>集合，得到所有值。即方法四
    
    6、方法二中使用迭代器对数据遍历，类型是Map.Entry<String, String>，即仍旧是键值对，集合是map.entrySet().iterator()，比方法三麻烦。

```Java
public static void main(String[] args) {

  Map<String, String> map = new HashMap<String, String>();
  map.put("1", "value1");
  map.put("2", "value2");
  map.put("3", "value3");
  
  //第一种：普遍使用，二次取值
  System.out.println("通过Map.keySet遍历key和value：");
  for (String key : map.keySet()) {
   System.out.println("key= "+ key + " and value= " + map.get(key));
  }
  
  //第二种
  System.out.println("通过Map.entrySet使用iterator遍历key和value：");
  Iterator<Map.Entry<String, String>> it = map.entrySet().iterator();
  while (it.hasNext()) {
   Map.Entry<String, String> entry = it.next();
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }
  
  //第三种：推荐，尤其是容量大时
  System.out.println("通过Map.entrySet遍历key和value");
  for (Map.Entry<String, String> entry : map.entrySet()) {
   System.out.println("key= " + entry.getKey() + " and value= " + entry.getValue());
  }

  //第四种
  System.out.println("通过Map.values()遍历所有的value，但不能遍历key");
  for (String v : map.values()) {
   System.out.println("value= " + v);
  }
 }
```

### *HashMap和Hashtable集合类的区别*

    HashMap与Hashtable都是java的集合类，都可以用来存放java对象，这是他们的相同点，但是他们也有区别。
    1、历史原因
        Hashtable是基于陈旧的Dictionary类的，HashMap是java 1.2引进的Map接口的一个实现。
    2、同步性
        Hashtable是线程同步的。这个类中的一些方法保证了Hashtable中的对象是线程安全的。而HashMap则是线程异步的，因此HashMap中的对象并不是线程安全的。因为同步的要求会影响执行的效率，所以如果你不需要线程安全的集合那么使用HashMap是一个很好的选择，这样可以避免由于同步带来的不必要的性能开销，从而提高效率。
    3、值
        HashMap可以让你将空值作为一个表的条目的key或value但是Hashtable是不能放入空值的(null)
    
## List、Map集合框架的选择★（常用）★★★★

    如何选用集合类？
    1、要求线程安全，使用Vector、Hashtable
    2、不要求线程安全，使用ArrayList,LinkedList,HashMap
    3、要求key和value键值，则使用HashMap,Hashtable
    4、数据量很大，又要线程安全，则使用Vector

## Set结构的集合类

### *用法*

```
#添加
add(Object)

#遍历
Iterator<String> iterator = hashSet.iterator();
while(iterator.hasNext()){
	System.out.println(iterator.next());
}

#删除元素
删除一个元素 remove(Object)
删除所有元素 clear()

#包含
contain(Object)
```

### *HashSet与TreeSet区别*

* HashSet随机存储对象，不能保证先插入的元素出现在前面；TreeSet中元素默认按照元素自然序进行排列，使用compareTo()方法自定义排序
* HashSet内部依靠HashMap，而TreeSet依靠TreeMap
* HashSet可以存储空对象，但TreeSet不允许
* 性能，HashSet对基本操作add，remove，size有常数的时间复杂度，TreeSet有log(n)的时间复杂度
* TreeSet比起HashSet具有更多丰富的功能，例如pollFirst(),pollLast(),first(),last(),celling(),lower()等
* 比较，HashSet使用equals()方法进行比较，而TreeSet使用compareTo()方法获得排列顺序

### *HashSet与TreeSet相同点*

* 只能存储独一无二的元素，不允许任何重复的元素
* 不是线程安全
* Clone方法有相同的实现技术

## Queue结构的集合







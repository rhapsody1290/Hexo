---
title: HashMap

date: 2017-05-10 21:50:00

categories:
- JDK源码阅读

tags:
- JDK

---

## 存储结构

![](http://i.imgur.com/nYCSoQg.png)

从图中可以看出 HashMap 的底层就是一个数组结构，数组中的每一项又是一个链表

```
transient Entry<K,V>[] table;  
static class Entry<K,V> implements Map.Entry<K,V> {  
	final K key;     //键
	V value;         //值
	Entry<K,V> next; //指向下一个节点  
	int hash;        //散列值
	
	Entry(int h, K k, V v, Entry<K,V> n) {  
		value = v;  
		next = n;  
		key = k;  
		hash = h;  
	}  
}  
```

HashMap有一个属性是Entry的数组table。Entry就是table数组中的元素，Map.Entry保存一个键值对
和这个键值对持有指向下一个键值对的引用，如此就构成了链表了

## 构造方法

一些属性：

* capacity：数组的长度，默认是16
* loadFactor：HashMap元素的个数除以数组的长度，当元素个数大于threshold时，需要扩容。默认负载因子是0.75
* threashold：数组的长度乘以负载因子，当元素大于threshold时，需要扩容。默认threshold = 16*0.75 = 12
* size：HashMap中元素的个数

```
/**根据指定容量和负载因子构造HashMap*/   
public HashMap(int initialCapacity, float loadFactor) {  
    if (initialCapacity < 0)  
        throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity);  
	/**
	 * 数组最大容量是 static final int MAXIMUM_CAPACITY = 1 << 30，即32位最大2的整数次
	 * 如果传入初始容量太大，真实设置的的最大
	 */
    if (initialCapacity > MAXIMUM_CAPACITY)  
        initialCapacity = MAXIMUM_CAPACITY;  
    if (loadFactor <= 0 || Float.isNaN(loadFactor))  
        throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
	
	/**
	 * 找到大于给出的初始容量的2的乘方
	 */
    // Find a power of 2 >= initialCapacity  
    int capacity = 1;  
    while (capacity < initialCapacity)  
        capacity <<= 1;  

	/**
	 * 负载因子是链表的长度除以数组的长度，当链表的长度大于threshold时，需要扩容
	 */
    this.loadFactor = loadFactor;  
    threshold = (int)Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);  
	
	/**
	 * 初始化 Entry 数组
	 */
    table = new Entry[capacity];  
	
	/**
	 * 空函数，子类可扩展
	 */
    init();  
}  

/**根据指定的容量和默认的负载因子构造HashMap*/  
public HashMap(int initialCapacity) {  
    this(initialCapacity, DEFAULT_LOAD_FACTOR);  
}  

//默认的空的构造器  
public HashMap() {  
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);  
}  
/**通过指定一个Map对象进行构造*/  
public HashMap(Map<? extends K, ? extends V> m) {  
    this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,  
                  DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);  
    putAllForCreate(m);  
}  
```

## PUT方法

当存入的key是null的时候将调用putForNUllKey方法，暂时将这段逻辑放一边，看key不为null的情况。先调用了hash(int h)方法获取了一个hash值。
<br/>

<font color='red'>**遍历table[i]上的元素，如果存在键相等，则替换它的值。否则以头插法的方式插入链表**</font>

之后判断size是否到达了需要扩充table数组容量的界限并让size自增1，如果达到了则调用resize(int capacity)方法将数组容量拓展为原来的两倍。

<pre>
public V put(K key, V value) {  
    // HashMap允许存放null键和null值。  
    // 当key为null时，调用putForNullKey方法，将value放置在数组第一个位置。  
    if (key == null)  
        return putForNullKey(value);  
    <font color='red'>// 根据key的keyCode重新计算hash值。 </font> 
    int hash = hash(key);//注意这里的实现是jdk1.7和以前的版本有区别的  
    <font color='red'>// 搜索指定hash值在对应table中的索引。</font>  
    int i = indexFor(hash, table.length);  
    // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素。如果键的Hash值一样，而且equal相等  
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {  
        Object k;  
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {  
            V oldValue = e.value;  
            e.value = value;  
            e.recordAccess(this);  
            return oldValue;  
        }  
    }  
    // 如果i索引处的Entry为null，表明此处还没有Entry。  
    modCount++;  
    // 将key、value添加到i索引处。  
    addEntry(hash, key, value, i);  
    return null;  
}  

/**
 * <font color='red'>采用头插法，即使头结点是空也没有关系！！</font>
 * 数组上的元素为刚插入的节点
 */
void addEntry(int hash, K key, V value, int bucketIndex) {
	Entry<K,V> e = table[bucketIndex];
	table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
	if (size++ >= threshold)
		resize(2 * table.length);
}
</pre>

**为什么要进行两次Hash？**

防止质量较差的哈希函数带来过多的冲突（碰撞）问题。Java中int值占4个字节，即32位。根据这32位值进行移位、异或运算得到一个值。

**如何确定元素在数组的位置？**

	static int indexFor(int h, int length) {
		return h & (length-1);     
	}

这样可以保证结果的最大值是length-1，而且均匀分布在数组中，异或的操作效率比取余的效率高

**HashMap中如何判断两个键相等？**

两个对象的HashCode相等， 而且equals相等。所以如果使用对象的作为键，需要重写HashCode和equals方法

String已经重写了HashCode，String中的数据结构是char[] s，Hash 值的计算方法是 s[0]*31^(n-1) + s[1]*31^(n-2) + ..... + s[0]*31^0

可以采用迭代的方法计算：h = h * 31 + s[i]

	int h = 0;
	for(int i = 0; i < n; i++){
		h = h * 31 + s[i];
	}

**链表扩容**

当链表内元素大于threashold时，新建一个新的数组，数组长度变成原来的两倍，并将原来的元素复制到新数组上

<pre>
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    // 这个if块表明，如果容量已经到达允许的最大值，即MAXIMUN_CAPACITY，则不再拓展容量，而将装载拓展的界限值设为计算机允许的最大值。
    // 不会再触发resize方法，而是不断的向map中添加内容，即table数组中的链表可以不断变长，但数组长度不再改变
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }
    // 创建新数组，容量为指定的容量
    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable);
    table = newTable;
    // 设置下一次需要调整数组大小的界限
    threshold = (int)(newCapacity * loadFactor);
}
</pre>

将old数组上的元素复制到新数组上的操作在transfer上完成，主要步骤是：

* 遍历原来的Entry数组
* 遍历链表
* 将链表元素依次按照插入到新的数组中(头插法)，<font color='red'>**链表反置**</font>

<pre>
void transfer(Entry[] newTable) {
    // 保留原数组的引用到src中，
    Entry[] src = table;
    // 新容量使新数组的长度
    int newCapacity = newTable.length;
	// 遍历原数组
    for (int j = 0; j < src.length; j++) {
        // 获取元素e
        Entry<K,V> e = src[j];
        if (e != null) {
            // 将原数组中的元素置为null
            src[j] = null;
            // 遍历原数组中j位置指向的链表
            do {
                Entry<K,V> next = e.next;
                // 根据新的容量计算e在新数组中的位置
                int i = indexFor(e.hash, newCapacity);
                // 将e插入到newTable[i]指向的链表的头部
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            } while (e != null);
        }
    }
}
</pre>

**HashMap允许键null、值null，怎么处理？**

从源码可以看到，key为null的对象放在数组**索引0的位置**。**put的方式和put普通元素一样**，首先遍历链表，是否有相同key的元素，如果有则替换并返回原来的值。否则头插入的方式将元素插入链表

<pre>
private V putForNullKey(V value) {
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(0, null, value, 0);
    return null;
}
</pre>

## GET方法

```
public V get(Object key) {
	/**
	 * 如果key为null，从table[0]所在的链表进行搜索，如果存在元素且元素的key为null，返回元素的值；如果之前没有传入key为null的元素，那么返回null
	 */
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
```

该方法分为key为null和不为null两块。先看不为null的情况。先获取key的hash值，之后通过hash值及table.length获取key对应的table数组的索引，遍历索引的链表，所找到key相同的元素，则返回元素的value，否者返回null。不为null的情况调用了getForNullKey()方法。

	private V getForNullKey() {
	    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
	        if (e.key == null)
	            return e.value;
	    }
	    return null;
	}



参考：http://www.cnblogs.com/hzmark/archive/2012/12/24/HashMap.html

## containsKey

containsKey(Object key)方法很简单，只是判断getEntry(key)的结果是否为null，是则返回false，否返回true

```
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}
```

getEntry(Object key)也没什么内容，只是根据key对应的hash值计算在table数组中的索引位置，然后遍历该链表判断是否存在相同的key值。

```
final Entry<K,V> getEntry(Object key) {
    int hash = (key == null) ? 0 : hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

## containsValue


判断一个value是否存在比判断key是否存在还要简单，就是遍历所有元素判断是否有相等的值。这里分为两种情况处理，value为null何不为null的情况，但内容差不多，只是判断相等的方式不同。

<pre>
public boolean containsValue(Object value) {
	if (value == null)
	        return containsNullValue();
	
	Entry[] tab = table;
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            <font color='red'>if (value.equals(e.value))</font>
                return true;
	return false;
}

private boolean containsNullValue() {
	Entry[] tab = table;
    for (int i = 0; i < tab.length ; i++)
        for (Entry e = tab[i] ; e != null ; e = e.next)
            <font color='red'>if (e.value == null)</font>
                return true;
	return false;
}
</pre>

## remove(Object key)

	public V remove(Object key) {
		Entry<K,V> e = removeEntryForKey(key);
		return (e == null ? null : e.value);
	}

看这个方法，removeEntryKey(key)的返回结果应该是被移除的元素，如果不存在这个元素则返回为null。remove方法根据removeEntryKey返回的结果e是否为null返回null或e.value。

<pre>
final Entry<K,V> removeEntryForKey(Object key) {
    int hash = (key == null) ? 0 : hash(key.hashCode());
    int i = indexFor(hash, table.length);
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
			<font color='red'>//临界情况，如果是第一个节点</font>
            if (prev == e)
                table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }

    return e;
}
</pre>

删除节点需要两个指针pre和e，next可以不需要吧？

## HashMap产生死锁的原因

调用 PUT 函数时，当 HashMap 中的元素大于 Threshold 时，会调用 Resize 函数进行扩容，数组长度增大一倍，整个过程如下：

1. 对索引数组中的元素遍历
2. 对链表上的每一个节点遍历：用 next 取得要转移那个元素的下一个，将 e 转移到新 Hash 表的头部，因为可能有元素，所以先将 e.next 指向新 Hash 表的第一个元素（如果是第一次就是 null)，这时候新 Hash 的第一个元素是 e，但是 Hash 指向的却是 e 没转移时候的第一个，所以需要将 Hash 表的第一个元素指向 e
3. 循环2，直到链表节点全部转移
4. 循环1，直到所有索引数组全部转移 

经过这几步，我们会发现**链表转移的时候是逆序的**。假如转移前链表顺序是1->2->3，那么转移后就会变成3->2->1。这时候就有点头绪了，死锁问题不就是因为1->2的同时2->1造成的吗？所以，**HashMap 的死锁问题就出在这个transfer()函数上。**

<pre>
public V put(K key, V value) {
	if (table == EMPTY_TABLE) {
	inflateTable(threshold);
	}
	if (key == null)
		return putForNullKey(value);
	int hash = hash(key);
	int i = indexFor(hash, table.length);
	//如果该 key 存在，就替换旧值
	for (Entry<K,V> e = table[i]; e != null; e = e.next) {
		Object k;
		if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
			V oldValue = e.value;
			e.value = value;
			e.recordAccess(this);
			return oldValue;
		}
	}

	modCount++;
	//如果没有这个 key，就插入一个新元素！跟进去看看
	addEntry(hash, key, value, i);
	return null;
}

void addEntry(int hash, K key, V value, int bucketIndex) {
	//查看当前的size是否超过了我们设定的阈值threshold，如果超过，需要resize
	if ((size >= threshold) && (null != table[bucketIndex])) {
		resize(2 * table.length);
		hash = (null != key) ? hash(key) : 0;
		bucketIndex = indexFor(hash, table.length);
	}
	createEntry(hash, key, value, bucketIndex);
}

//新建一个更大尺寸的hash表，把数据从老的Hash表中迁移到新的Hash表中。
void resize(int newCapacity) {
	Entry[] oldTable = table;
	int oldCapacity = oldTable.length;
	if (oldCapacity == MAXIMUM_CAPACITY) {
		threshold = Integer.MAX_VALUE;
		return;
	}
	//创建一个新的 Hash 表
	Entry[] newTable = new Entry[newCapacity];
	//转移！！！！跟进去
	transfer(newTable, initHashSeedAsNeeded(newCapacity));
	table = newTable;
	threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

<font color='red'>//高能预警！！！！重点全在这个函数中</font>
void transfer(Entry[] newTable, boolean rehash) {
	int newCapacity = newTable.length;
	for (Entry<K,V> e : table) {
		while(null != e) {
			Entry<K,V> next = e.next;
			if (rehash) {
				e.hash = null == e.key ? 0 : hash(e.key);
			}
			int i = indexFor(e.hash, newCapacity);
			e.next = newTable[i];
			newTable[i] = e;
			e = next;
		}
	}
}
</pre>

下面着重分析一下transfer方法，精简后得到如下代码★★★★★：

```
while(null != e) {
	//因为是单链表，如果要转移头指针，一定要保存下一个结点，不然转移后链表就丢了
	Entry<K,V> next = e.next;
	//e 要插入到链表的头部，所以要先用 e.next 指向新的 Hash 表第一个元素
	e.next = newTable[i];
	//将新 Hash 表的头指针指向 e
	newTable[i] = e;
	//转移 e 的下一个结点 
	e = next;
}
```

**死锁原因重现**

假设这里有两个线程同时执行了 put() 操作，并进入了 transfer() 环节，现在假设线程1的工作情况如下代码所示，而线程2完成了整个 transfer() 过程，所以就完成了 rehash。

<pre>
while(null != e) {
	<font color='red'>Entry<K,V> next = e.next; //线程1执行到这里被调度挂起了</font>
	e.next = newTable[i];
	newTable[i] = e;
	e = next;
}
</pre>

那么现在的状态：

![](http://i.imgur.com/8PFB2wW.png)

从上面的图我们可以看到，因为线程1的 e 指向了 key(3)，而 next 指向了 key(7)，在线程2 rehash 后，就指向了线程2 rehash 后的链表。

然后线程1被唤醒了，执行：

第一步 

	Entry<K,V> next = e.next;
	e.next = newTable[i];
	newTable[i] = e;
	e = next;

链表状态如下图：

![](http://i.imgur.com/GbZgrGm.png)

第二步

	Entry<K,V> next = e.next;
	e.next = newTable[i];
	newTable[i] = e;
	e = next;

链表状态如下：

![](http://i.imgur.com/AlWEfbc.jpg)

第三步

	Entry<K,V> next = e.next;
	e.next = newTable[i];
	newTable[i] = e;
	e = next;

![](http://i.imgur.com/Yk3AC54.jpg)

很明显，环形链表出现了！！当然，现在还没有事情，因为下一个节点是 null，所以transfer()就完成了，等put()的其余过程搞定后，HashMap 的底层实现就是线程1的新 Hash 表了。

**没错，put()过程虽然造成了环形链表，但是它没有发生错误。它静静的等待着get()这个冤大头的到来。**

现在程序被执行了一个hashMap.get(11)，这时候会调用getEntry()，这个函数就是去找对应索引的链表中有没有这个 key。然后。。。。悲剧了

因为 HashMap 为了性能考虑，没有使用锁机制。所以就是非线程安全的，而 ConcurrentHashMap 使用了锁机制，所以是线程安全的。**要并发就使用 ConcurrentHashMap**

参考：http://blog.csdn.net/luxia_24/article/details/52344367
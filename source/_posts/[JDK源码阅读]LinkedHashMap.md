---
title: LinkedHashMap

date: 2017-06-29 14:00:00

categories:
- JDK源码阅读

tags:
- Java

---
## 设计

1、Entry节点增加了after、before两个指针，组成双向链表，记录节点插入顺序

	private static class Entry<K,V> extends HashMap.Entry<K,V> {
		Entry<K,V> before, after;
	}

2、LinkedHashMap保存头节点指针，插入的方式都采用头插法，因为是双向链表，可以轻易插在头部和尾部

	public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
	{
	    private transient Entry<K,V> header;
	}

3、LinkHashMap有一个成员变量accessOrder，表示链表的保存顺序，默认是false，按照插入顺序保存。

	private final boolean accessOrder;

**注意：**因为插入时采用的是头插法，所以链表的存储顺序是逆序的，**即先插入的在尾部**，但是迭代器在遍历的时候从尾到头的顺序，所以不影响

如果想实现LRU的方式，按照访问顺序的方式存储节点，构造LinkedHashMap的时候accessOrder设为true

	public LinkedHashMap(int initialCapacity,
			 float loadFactor, boolean accessOrder) {
	    super(initialCapacity, loadFactor);
	    this.accessOrder = accessOrder;
	}

## 源码关键 

	//LinkedHashMap方法
	public V get(Object key) {
	    Entry<K,V> e = (Entry<K,V>)getEntry(key);
	    if (e == null)
	        return null;
	    e.recordAccess(this);
	    return e.value;
	}

    //HashMap方法
    public V put(K key, V value) {
		if (key == null)
	    return putForNullKey(value);
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
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
        addEntry(hash, key, value, i);
        return null;
    }

1、<font color='red'>**当调用get或者put方法的时候，如果K-V已经存在，会回调Entry.recordAccess()方法**</font>

当 accessOrder 为 true，即按LRU的策略存储，会先调用 remove 清除的当前首尾元素的指向关系，之后调用addBefore方法，**将当前元素加入header之前。**

	/**
	 * This method is invoked by the superclass whenever the value
	 * of a pre-existing entry is read by Map.get or modified by Map.set.
	 * If the enclosing Map is access-ordered, it moves the entry
	 * to the end of the list; otherwise, it does nothing. 
	 */
	void recordAccess(HashMap<K,V> m) {
	    LinkedHashMap<K,V> lm = (LinkedHashMap<K,V>)m;
	    if (lm.accessOrder) {
	        lm.modCount++;
	        remove();
	        addBefore(lm.header);
	    }
	}
	
	/**
	 * Remove this entry from the linked list.
	 */
	private void remove() {
	    before.after = after;
	    after.before = before;
	}
	
	/**                                             
	 * Insert this entry before the specified existing entry in the list.
	 */
	private void addBefore(Entry<K,V> existingEntry) {
	    after  = existingEntry;
	    before = existingEntry.before;
	    before.after = this;
	    after.before = this;
	}

2、put操作时，如果key不存在，当有新元素加入 Ma p的时候会调用 Entry 的 addEntry 方法，**会调用 removeEldestEntry 方法，这里就是实现LRU元素过期机制的地方**，默认的情况下removeEldestEntry方法只返回false表示元素永远不过期。 

<pre>
/**
 * This override alters behavior of superclass put method. It causes newly
 * allocated entry to get inserted at the end of the linked list and
 * removes the eldest entry if appropriate.
 */
void addEntry(int hash, K key, V value, int bucketIndex) {
    createEntry(hash, key, value, bucketIndex);

    // Remove eldest entry if instructed, else grow capacity if appropriate
    Entry<K,V> eldest = header.after;
    if (<font color='red'>removeEldestEntry(eldest)</font>) {
        removeEntryForKey(eldest.key);
    } else {
        if (size >= threshold) 
            resize(2 * table.length);
    }
}

/**
 * This override differs from addEntry in that it doesn't resize the
 * table or remove the eldest entry.
 */
void createEntry(int hash, K key, V value, int bucketIndex) {
    HashMap.Entry<K,V> old = table[bucketIndex];
	Entry<K,V> e = new Entry<K,V>(hash, key, value, old);
    table[bucketIndex] = e;
    e.addBefore(header);
    size++;
}

protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    <font color='red'>return false;</font>
}
</pre>

## 实现LRU

LRU缓存LinkedHashMap(inheritance)实现

采用inheritance方式实现比较简单，而且实现了Map接口，在多线程环境使用时可以使用 **Collections.synchronizedMap()**方法实现线程安全操作

1、构造函数中调用父类的构造函数，accessOrder设置为true
2、重写removeEldestEntry，当元素数量大于缓存最大数量时，删除

<pre>
import java.util.LinkedHashMap;
import java.util.Map;

public class LRUCache2<K, V> extends LinkedHashMap<K, V> {
    
	private final int MAX_CACHE_SIZE;

	public LRUCache2(int cacheSize) {
		super ((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, <font color='red'>true</font>);
		MAX_CACHE_SIZE = cacheSize;
	}

    @Override
	protected boolean removeEldestEntry(Map.Entry eldest) {
		<font color='red'>return size() > MAX_CACHE_SIZE;</font>
    }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<K, V> entry : entrySet()) {
            sb.append(String.format("%s:%s ", entry.getKey(), entry.getValue()));
        }       
		return sb.toString();
    }
}
</pre>

## 实现FIFO

1、只需重写removeEldestEntry就行，默认accessOrder就是false
2、重写removeEldestEntry方法，当缓存个数大于最大个数时，删除最先放进去的元素

	final int cacheSize = 5;
	LinkedHashMap<Integer, String> lru = new LinkedHashMap<Integer, String>() {
	    @Override   
		protected boolean removeEldestEntry(Map.Entry<Integer, String> eldest) {
	    	return size() > cacheSize;
	    }
	};


## 参考

LRU缓存实现(Java)

http://transcoder.tradaquan.com/from=1086k/bd_page_type=1/ssid=0/uid=0/pu=usm%401%2Csz%40320_1002%2Cta%40iphone_2_5.1_2_6.0/baiduid=99806F566512D4B5CD2F1444260EE0C9/w=0_10_/t=iphone/l=3/tc?ref=www_iphone&lid=7454783636244085594&order=1&fm=alop&h5ad=1&srd=1&dict=32&tj=www_normal_1_0_10_title&url_mf_score=4&vit=osres&m=8&cltj=cloud_title&asres=1&title=LRU%E7%BC%93%E5%AD%98%E5%AE%9E%E7%8E%B0%28Java%29-%E6%87%92%E6%83%B0%E7%9A%84%E8%82%A5%E5%85%94-%E5%8D%9A%E5%AE%A2%E5%9B%AD&w_qd=IlPT2AEptyoA_yitJU7mHiU5vhXPLqCyZhaDQ_&sec=22108&di=8c5570f1d6ae3a47&bdenc=1&nsrc=IlPT2AEptyoA_yixCFOxXnANedT62v3IEQGG_ytK1DK6mlrte4viZQRAXinhMXmYGlGwdoSOxBt8w83c_79j7QwTaP1s&clk_info=%7B%22srcid%22%3A%221599%22%2C%22tplname%22%3A%22www_normal%22%2C%22t%22%3A1498709801466%2C%22sig%22%3A%221646%22%2C%22xpath%22%3A%22div-a-h3%22%7D

基于LinkedHashMap实现LRU缓存调度算法原理及应用

http://woming66.iteye.com/blog/1284326
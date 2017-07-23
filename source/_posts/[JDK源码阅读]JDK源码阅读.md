---
title: JDK源码阅读

date: 2016-10-13 09:00:00

categories:
- JDK源码阅读

tags:
- JDK

---
## 文件IO

//类加载根目录
String parent = getClass().getClassLoader().getResource("").getPath();

### BufferedOutputStream

要介绍BufferedOutputStream，我们先了解一下OutputStream类 
抽象类OutputStream类有三个write方法

	public abstract void write(int b)
	public void write(byte b[])
	public void write(byte b[], int off, int len)

由上面我们可以看出第一个write方法是让子类覆盖的，而第二个人write（byte b[]）方法源代码如下

	public void write(byte b[]) throws IOException {
        write(b, 0, b.length);
    }

所以可见最后处理还是调用第三个方法write(byte b[],int off,int len)，该方法源码如下：
 
	public void write(byte b[], int off, int len) throws IOException {
        if (b == null) {
            throw new NullPointerException();
        } else if ((off < 0) || (off > b.length) || (len < 0) ||
                   ((off + len) > b.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }
        for (int i = 0 ; i < len ; i++) {
        //注意这儿，这儿其实调用前面的抽象方法write（int b）,同时还发生了自动转型
            write(b[off + i]);
        }
    }

我们先不看抽象方法是如何实现的，**也就是说OutputStream也具有缓存器功能，我们可以将要写入到流中的数据写到一个byte[] buf数组中，然后调用write(byte b[])或者write(byte b[], int off, int len)也可以**，那为什么还要BufferedInputStream类干什么呢，他们有什么区别呢。同时我们知道BufferedInputStream类中还有一个flush()方法，在OutputStream流中没有flush()方法，这又是为什么呢？flush()是不是必须的呢，接下来看一下BufferedOutputStream类；

首先，BufferedOutput将OutputStream类对象作为一个构造方法的参数的。 
首先看一下 BufferedOutputStream 类源代码

	public class BufferedOutputStream extends FilterOutputStream {
	
	    //这儿定义了一个byte[]数组，用来充当缓存器  
	    protected byte buf[]; 
	    //这个变量是重点，他就是用来记录当前缓存器中的字节数量的
	    protected int count; 
	    //我们初始化创建一个对象的时候给了这个buf这个数组8192个字节.
	    public BufferedOutputStream(OutputStream out) {
	        this(out, 8192);
	    }
	
	    public BufferedOutputStream(OutputStream out, int size) {
	        super(out);
	        if (size <= 0) {
	            throw new IllegalArgumentException("Buffer size <= 0");
	        }
	      // 这儿创建一个给定大小的数组对象来充当缓存器
	        buf = new byte[size];
	    }
	
	    public synchronized void write(int b) throws IOException {
	        if (count >= buf.length) {
	            flushBuffer();
	        }
	        buf[count++] = (byte)b;
	    }
	
		//该方法是重点    
		public synchronized void write(byte b[], int off, int len) throws IOException {
	        //如果传进来的数组长度大于buf 数组的长度，则直接调用OutputStream对象的write方法。
	        if (len >= buf.length) {
	
	            flushBuffer();
	            out.write(b, off, len);
	            return;
	        }
	        //验证BufferedOutputStream 类中buf剩下的空间能否装得下传进来的数组。如果不能则先将当前buf数组中数据写入底层io流中
	        if (len > buf.length - count) {
	            flushBuffer();
	        }
	        //该处是重点，如果在当前BufferedOutputStream 类中buf数组没有满，则将传进来的数组复制到当前类对象buf数组中，同时更新count的值。
	        System.arraycopy(b, off, buf, count, len);
	        count += len;
	   //调用flushBuffer方法也就是将不满8192个字节数组中的数据发送出去。同时将count置零。
	    private void flushBuffer() throws IOException {
	        if (count > 0) {
	            out.write(buf, 0, count);
	            count = 0;
	        }
	    }
	
	    //强制将buf数据中未满8192个字节的数据写入底层io中。
	    public synchronized void flush() throws IOException {
	        flushBuffer();
	        out.flush();
	    }
	}

结论： 

OutputStream的缓存器（数组）与BufferedOutputStream中类的缓存器（数组）本质是一样的，只是BufferedOutputStream类中将要写入到底层io流中的数据先 **凑个整**，然后再一起写入底层io流中，这样就大大**节省了io操作**，大大提高了io利用率，写一次io是很费资源的。这样也出现了一个问题，假设向硬盘中写入一个文件，文件最后数据比默认值8192个字节小，则BufferOutputStream就不会将这些数据写入底层io流中，造成文件缺失，因此就**需要在close()前调用flush（）方法，强制将还没有装满buf数组的数据写入底层io中**。同时也可以看出节点流是不用flush()方法的，而一般的处理流都会采用固定buf这种方式的，比如常用的PrintWriter里面其实操作的就是一个BufferedWriter对象，因此也需要调用flush（）方法来刷新，因为默认是不刷新的。

引用：http://m.blog.csdn.net/article/details?id=51355523

## Arrays

### copyOf

<pre>
<font color='red'>//复制原来的数组，长度为newLength</font>
public static <T> T[] copyOf(T[] original, int newLength) {
	return (T[]) copyOf(original, newLength, original.getClass());
}

public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
	<font color='red'>//将original复制到copy中，从originnal的0开始，copy的0开始，长度为newLength</font>
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
</pre>

### sort

<pre>
<font color='red'>//from包括，toIndex不包括</font>
public static void sort(int[] a, int fromIndex, int toIndex) {
    rangeCheck(a.length, fromIndex, toIndex);
	sort1(a, fromIndex, toIndex-fromIndex);
}
</pre>

### equals

<pre>
public static boolean equals(int[] a, int[] a2) {
    if (a==a2)
        return true;
    if (a==null || a2==null)
        return false;

    int length = a.length;
    if (a2.length != length)
        return false;

    for (int i=0; i<length; i++)
        if (a[i] != a2[i])
            return false;

    return true;
}
</pre>

### asList

<pre>
public static <T> List<T> asList(T... a) {
<font color='red'>//生成的是内部类的ArrayList</font>
return new ArrayList<T>(a);
}

<font color='red'>//内部类，add、remove没有重写（改动了数组的长度），get、set重写了（不改动数组的长度）</font>
private static class ArrayList<E> extends AbstractList<E>
implements RandomAccess, java.io.Serializable
{
	private static final long serialVersionUID = -2764017481108945198L;
	private final E[] a;
	
	ArrayList(E[] array) {
	    if (array==null)
	        throw new NullPointerException();
		a = array;
	}
	
	public int size() {
		return a.length;
	}
	
	public Object[] toArray() {
		return a.clone();
	}
	
	public <T> T[] toArray(T[] a) {
		int size = size();
		if (a.length < size)
			return Arrays.copyOf(this.a, size,
				     (Class<? extends T[]>) a.getClass());
		System.arraycopy(this.a, 0, a, 0, size);
		if (a.length > size)
			a[size] = null;
		return a;
	}
	
	public E get(int index) {
		return a[index];
	}
	
	public E set(int index, E element) {
		E oldValue = a[index];
		a[index] = element;
		return oldValue;
	}
	
	public int indexOf(Object o) {
		if (o==null) {
		    for (int i=0; i<a.length; i++)
		        if (a[i]==null)
		            return i;
		} else {
		    for (int i=0; i<a.length; i++)
		        if (o.equals(a[i]))
		            return i;
		}
		return -1;
	}
	
	public boolean contains(Object o) {
		return indexOf(o) != -1;
	}
}
</pre>

## 判断两个对象是否相等

### 判断两个对象是否相等代码

	public boolean equals(Object o) {
		if (this == o) return true;
		if (o == null || getClass() != o.getClass()) return false;

		Employee employee = (Employee) o;

		if (id != null ? !id.equals(employee.id) : employee.id != null) return false;
		if (name != null ? !name.equals(employee.name) : employee.name != null) return false;
		if (email != null ? !email.equals(employee.email) : employee.email != null) return false;
		if (hiredate != null ? !hiredate.equals(employee.hiredate) : employee.hiredate != null) return false;

		return true;
	}

其中Employee有如下属性：

	public class Employee {
		private Integer id;
		private String name;
		private String email;
		private Date hiredate;
	}

### 代码解析

判断两个对象是否相等的原则：***对象类型相同***，而且***属性值全部相等***

1、如果两个对象地址相等，两个对象一定相等

	if( this == o ) return true;

2、两个对象类型不相等，两个一定不相等

	if( getClass() != o.getClass() ) return false;
	养成一个好习惯，在使用对象前判断对象o是否为空
	考虑到如果比较对象o为空null，但当前对象this必然不为null，不相等，可以结合起来：
	if( o == null || getClass() != o.getClass() ) return false;

3、如果两个对象所有属性相等，两个对象才相等，否则判定为不相等

	类型强转:Employee employee = (Employee) o;
	如果属性不相等，return false
	if( !id.equals(employee.id) ) return false;
	同样，需要判断id是否为空。如果id为空，不能使用equals作为判断方法，如下
	if( id == null && employee.id != null || id != null && !id.equals(employee.id) ) return false;
	使用三目运算符让代码更加优雅：
	if( id != null ? !id.equals(employee.id) : employee.id != null) return false

## HashMap

### 存储结构

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

### 构造方法

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

### PUT方法

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

### GET方法

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

### containsKey

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

### containsValue


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

### remove(Object key)

	public V remove(Object key) {
		Entry<K,V> e = removeEntryForKey(key);
		return (e == null ? null : e.value);
	}

看这个方法，removeEntryKey(key)的返回结果应该是被移除的元素，如果不存在这个元素则返回为null。remove方法根据removeEntryKey返回的结果e是否为null返回null或e.value。

```
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
```

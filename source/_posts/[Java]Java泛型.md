---
title: Java泛型

date: 2016-09-30 00:00:00

categories:
- Java

tags:
- Java

---
## 什么是泛型

泛型本质是**参数化类型**，也就是说把操作的**数据类型**指定为一个**参数**。这种参数类型可以用在**类、接口和方法**的创建中，分别称为泛型类、泛型接口、泛型方法。 

Java语言引入泛型的好处是：
1、**安全简单**，在**编译的时候检查类型安全**，并且所有的**强制转换都是自动和隐式**。比如**在定义集合的时候指定具体的参数类型，这样无法加入指定类型以外的数据，避免了强制类型转换时出现对转换错误的问题**
2、能够复用算法，防止重复代码的编写

## 泛型的基本应用

* Jdk1.5的集合类希望你在定义集合时，***明确表示你要向集合中装哪种类型的数据***，无法加入指定类型以外的数据
* 如下例子，如果不给定参数类型，可以在集合类中加入任意类型的数据，但在取元素时必须由程序员强行转换数据类型，编译器不会报错，出现问题由程序员负责


	ArrayList collection1 = new ArrayList();
	collection1.add(1);
	collection1.add(1L);
	collection1.add("abc");

* 在指定参数后，泛型的基本用法如下，**对不是给定类型的数据，编译会报错**。另一个好处是取出数据后，**不用对数据进行强制转换**。


	ArrayList<String> collection2 = new ArrayList<String>();
	//collection2.add(1);
	//collection2.add(1L);
	collection2.add("abc");
	String string = collection2.get(0);

* **泛型是提供给javac编译器使用的**，可以限定集合中的输入类型，让编译器挡住源程序中的非法输入，编译器编译带类型说明的集合时会**去除掉"类型"信息**，使程序运行效率不受影响，对于参数化的泛型类型，getClass()方法的返回值和原始类型完全一样


	ArrayList<Integer> collection3 = new ArrayList<Integer>();
	System.out.println(collection1.getClass() == collection2.getClass());//True
	System.out.println(collection2.getClass() == collection3.getClass());//True


* 由于编译生成的字节码会去掉泛型的类型信息，只要能跳过编译器，就可以往某个泛型集合中加入其它类型的数据，例如，用**反射**得到集合，再调用其add方法即可



	ArrayList<Integer> collection3 = new ArrayList<Integer>();
	collection3.getClass().getMethod("add",Object.class).invoke(collection3, "abc");
	System.out.println(collection3.get(0));//abc

collection3的参数类型是Integer，但用反射的方法跳过编译器，还是能够加入String类型的数据

## 泛型的优点

1. 提高了我们程序的使用性，不需要在自己去做强转
2. 将原来在运行阶段处理的问题，放到了编译阶段
3. 提高安全性

## 泛型的术语

ArrayList<E>类定义和ArrayList<Integer>类引用中涉及如下术语：

* 整个称为 ArrayList<E>泛型类型
* ArrayList<E>中的E称为类型变量或**类型参数**
* 整个ArrayList<Integer>称为**参数化的类型**
* ArrayList<Integer>中的Integer称为类型参数的实际类型参数或实际类型参数
* ArrayList<Integer>中的<>念typeof
* ArrayList称为**原始类型**

### 参数化类型与原始类型的兼容性

* 参数化类型可以引用一个原始类型，编译报告警告，例如
	  Collection<String> c = new Vector();

* 原始类型可以引用一个参数化类型的对象，编译报告警告，例如
	  Collection c = new Vector<String>();

### 参数化类型不考虑类型参数的继承关系

不要认为String和Object有继承关系就不会报错

* Vector<String> v = new Vector<Object>();//错误
* Vector<Object> v = new Vector<String>();//错误

**思考**，下面的代码会报错误吗？

	Vector v1 = new Vector<String>();
	Vector<Object> v = v1;

答：不会，第一个语句，原始类型可以引用一个参数化类型的对象，第二个语句，参数化类型可以引用一个原始类型的对象，所以不会报错。

## 泛型的声明

**泛型可以声明在类上，可以声明在方法上，可以声明在接口上**。声明在类上的泛型，在整个类的范围内都可以使用。声明在方法上的泛型，只能在方法上使用。

***什么时候在方法上声明泛型？***

类上已经声明了泛型，但是在方法上我们不使用类上的泛型，而是自定义的一个，那么就可以在方法上声明，注意：在方法上声明时，**泛型要定义在方法的返回值前面。**

```
public class APP {
    public static void main(String[] args) {
		//sayHello的参数列表中使用了泛型，可以传入不同的数据类型
        String hello = sayHello("Hello");
        Integer integer = sayHello(1);
        Float aFloat = sayHello(1F);
        Double aDouble = sayHello(1D);
    }
	//泛型定义在返回值前面
    private static <T> T sayHello(T parameter) {
        System.out.println(parameter);
        return parameter;
    }
}
```

## 泛型中使用？通配符

定义一个printCollection函数，参数是一个集合，类型参数是？通配符，这样可以将任何类型参数的集合作为参数传入，如ArrayList<Integer>

	public static void printCollection(Collection<?> collection){
		//System.out.println(collection.add("abc"));//错误，因为它不知道自己未来匹配就一定是String
		System.out.println(collection.size());//没错，此方法与类型参数无关
		for(Object object : collection){
			System.out.println(object);
		}
	}

总结：**使用`？`通配符可以引用其它各种参数化的类型**，`？`通配符定义的变量主要用作引用，**只可以调用与参数化无关的方法**，不能调用与参数化有关的方法

## 泛型中的 ? 通配符的扩展

* <? extends E> 它是用来限定是E类型或是E的子类型.
* <? super E>  只能是E类型或E的父类型.

### 限定通配符的上边界

* 正确：Vector<? extends Number> x = new Vector<Integer>(); 
* 错误：Vector<? extends Number> x = new Vector<String>();

### 限定通配符的下边界

* 正确：Vector<? super Number> x = new Vector<Number>();
* 错误：Vector<? super Number> x = new Vector<Byte>();

## 泛型方法（自定义泛型）

**泛型类，是在实例化类的时候指明泛型的具体类型；泛型方法，是在调用方法的时候指明泛型的具体类型。**

<font color='red'>在返回值前加上&lt;T>表示新定义一个类型</font>，返回`x+y`有错误，因为不确定T类型是否有`+`这个方法，<font color='red'>**返回类型为传入类型的最大公共类型**</font>，比如传入Integer和Float，返回Numberic，传入Integer和String，返回Object

![](http://i.imgur.com/hZFc0o8.png)

**泛型的实际类型只能是引用类型，不能是基本类型**。`T`不能被基本类型替换。在案例`add(1，2)`中会自动装箱，而在swap案例中，**数组本身就是Object，不会装箱**。

```Java
private <T> void swap(T[] a, int i, int j ){
		T tmp = a[i];
		a[i] = a[j];
		a[j] = tmp;
}
```

```Java
swap(new String[]{"abc","xyz","asd"},1,2);//正确
//swap(new int[]{1,2,3,4,5},1,2);//报错
```

除了在应用泛型时可以使用extends限定符，在定义泛型时也可以使用extends限定符,可以用来指定多个边界，如 

```　　
//必须实现Serializable 和cloneable两个接口
<V extends Serializable & cloneable > void method(){}
```

普通方法、构造方法和静态方法中都可以使用泛型

也可以用类型变量表示异常，称为参数化的异常，可以用于方法的throws列表中，但是不能用于catch子句中。
例：用下面的代码说明对异常如何采用泛型：

```Java
private static <T extends Exception> sayHello() throws T
{
    try{
 
    }catch(Exception e){
        throw (T)e;
    }
}
``` 

在泛型中可以同时**有多个类型参数**，在定义它们的尖括号中用逗号分，例如：

```Java
public static <K,V> V getValue(K key) {
	return map.get(key); 
}
```

我们在开发中，如果在类上已经声明了泛型，但是我们在方法上不使用类上声明的泛型，我们可以自己在方法上声明泛型。

* 这个在方法上声明的泛型，只能在方法内使用。如果声明在方法上，必须声明在方法的返回值前面。 
* 泛型声明在类上，那么这个泛型可以在整个类内使用.

```Java
class Demo9 
{
    public static void main(String[] args) 
    {
        Student<String> s=new Student<String>();
        s.a="hello";
        System.out.println(s.print());
    }
}
```
```Java
//在类上定义泛型
//如果将泛型定义在类上，那么这个泛型可以在整个类内使用.
class Student<T>  
{
    T a;  //泛型做为成员属性
    public void show(){
        System.out.println(a);
    }
    public void show(T t){ //泛型作用在方法.
        System.out.println(t);
    }
    public T print(){
        return a;
    }
}
```

**泛型练习题**

自动将Object类型的对象转换成其他类型

```
public class Generic {

    public static void main(String[] args){
        Object o = "123";
        String s = convert(o);
        System.out.println(s);
    }

    public static <T> T convert(Object o){
        return (T) o;
    }
}
```

**注意：这里的T由返回类型决定!!!**

## 通过反射获取泛型的实际参数类型
```
Vector<Date> v1 = new Vector<Date>();
```

通过获取v1的字节码来获取Vector的实际参数类型是**不可能**，因为编译器在编译后会去掉类型信息。但在已知框架中的确有这个应用，它是怎么实现的呢？

通过变量是无法知道它的参数类型的，但是当把这个变量交给一个方法去使用的时候，方法可以获得它的参数列表，并且是泛型的形式。

```Java
public class GenericTest {
	public static void main(String[] args) throws Exception{
		//Vector<Date> v1 = new Vector<Date>();
		Method applyMethod = GenericTest.class.getMethod("applyVector", Vector.class);
		Type[] types = applyMethod.getGenericParameterTypes();
		ParameterizedType pType = (ParameterizedType)types[0];
		System.out.println(pType.getRawType());
		System.out.println(pType.getActualTypeArguments()[0]);
		
	}
	public static void applyVector(Vector<Date> v1){
		
	}
}
```
结果：
	class java.util.Vector
	class java.util.Date

## 泛型DAO

1、编写BaseDAO类，类中封装常用的DAO方法，如selectById

```
public abstract class BaseDAO<T> {
    private Class<T> entityClass;

    protected BaseDAO() {
        //得到子类的泛型父类
        Type type = getClass().getGenericSuperclass();
        //转化为泛型类型，返回此泛型类型的实际类型参数的Type对象的数组,数组里放的都是对应类型的Class
        Type[] trueType = ((ParameterizedType) type).getActualTypeArguments();
        this.entityClass = (Class<T>) trueType[0];
        System.out.println("传入泛型类型：" + entityClass.getCanonicalName());
    }

    protected T findById(Serializable id){
        //从数据库中查询，并返回
        T t = null;
        try {
            t = entityClass.newInstance();
            Field field_id = entityClass.getDeclaredField("id");
            field_id.setAccessible(true);
            field_id.set(t,id);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return t;
    }

}
```

当调用getClass()方法，**隐含的this指针是子类对象**（因为new的是子类，this对象指的是子类对象。虽然会执行父类的构造方法，但并没有构造一个父类的对象），获得父类

2、对于每一个用户自定义的DAO类，继承BaseDAO，并且传入Domain类作为类型参数。这样可以从父类BaseDAO中获得常用的单表增、删、改、查方法。如果用户需要一些复杂查询，需要自行添加

```
public class UserDAO extends BaseDAO<User> {
}
```

3、Domain对象对应数据库中的一张表，从数据库中查出的一条记录需要转变成一个Domain对象，方便Service中使用

```
public class User {
    private long id;
    private String name;
    private String password;

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

4、测试具体DAO的使用

```
public class APP {
	UserDAO userDAO = new UserDAO();
	User user = userDAO.findById(2);
	System.out.println(user);
}
```

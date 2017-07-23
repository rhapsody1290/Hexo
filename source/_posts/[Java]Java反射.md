---
title: Java反射

date: 2016-12-30 21:55:00

categories:
- Java

tags:
- Java

---

## 反射的基石->Class

Java类用于描述一类事物的共性，该类事物有什么属性，没有什么属性，至于这个属性的值是什么，则是由这个类的实例对象来确定，不同的实例对象有不同的属性值。Java程序中的各个Java类属于同一类事物，描述这类事物的Java类名就是Class。

例：众多的人可以用一个Person类来表示，那么众多的Java类也就可以用一个Class类来表示。

Person类代表人，它的实例对象就是张三，李四这样的一个个具体的人，**Class类代表Java类，它的各个实例对象又分别对应各个类的内存中的字节码**，例如，Person类的字节码，ArrayList类的字节码，等等。

####**什么是字节码**
　　一个类被类加载器加载到内存中，占用一片内存空间，这个空间里面的内容就是类的字节码，不同的类的字节码是不同的，所以它们在内存中的内容是不同的，这个一个个的空间可以分别用一个个的对象来表示，这些对象显然具有相同的类型，这个类型是什么呢？Class类型
####**如何得到各个字节码对应的实例对象（Class类型）**
>* 类名.class,例如，System.class
* 对象.getClass(),例如，new Date().getClass()
* Class.forName("类名"),例如，Class.forName("java.util.Date");

　　开发中使用较多的是Class.forName（"类名"）的方式，按参数中指定的字符串形式的类名去搜索并加载相应的类，如果该类字节码已经被加载过，则返回代表该字节码的Class实例对象，否则，按类加载器的委托机制去搜索和加载该类，如果所有的类加载器都无法加载到该类，则抛出ClassNotFoundException。
```Java
String str1 = "abc";
Class c1s1 = str1.getClass();
Class c1s2 = String.class;
Class c1s3 = Class.forName("java.lang.String");
System.out.println(c1s1 == c1s2);//true
System.out.println(c1s2 == c1s3);//true
```
结果说明：
　　1. 一份字节码可以得到多个实例对象
　　2. 各个实例对象都将得到同一份字节码

####**九个预定义Class实例对象**
　　isPrimitive() : 字节码是否是基本类型
```Java
System.out.println(String.class.isPrimitive());//false
System.out.println(int.class.isPrimitive());//true
System.out.println(Integer.class.isPrimitive());//false
System.out.println(int.class == Integer.TYPE);//true
System.out.println(int[].class.isPrimitive());//false
System.out.println(int[].class.isArray());//true
```
####**总结**
>　　Java类被类加载器加载到内存中，开辟一段内存空间，这段空间的内容就是类的字节码。不同的类生成不同的字节码，每个字节码可以用一个个对象表示，这些对象具有相同的类型，即Class类型。
###2. **理解反射的概念**
　　反射就是把Java类中的各种成分映射成相应的Java类。例如，一个Java类中用Class类的对象来表示，一个类中的组成部分：成员变量、方法、构造方法、包等信息也用一个个的Java类来表示。表示Java类的Class类显然要提供一系列的方法，来获取其中的变量、方法、构造方法、修饰符、包等信息，这些信息就是用相应的实例对象来表示，它们是Field、Method、Constructor、Package等等。
　　例如，System.exit和System.getProperties()方法，分别可以用Method类的methodObj1，methodOjb2表示。
###3. **Constructor类**
　　Constructor类代表某个类中的一个构造方法。
####**得到某个类所有的构造方法**
　　`Constructor [] constructors　=　Class.forName("java.lang.String").getConstructors();`
####**得到某一个构造方法（根据参数来获取）**
　　`Constructor constructor = Class.forName(“java.lang.String”).getConstructor(StringBuffer.class);`
####**创建实例对象**
　　通常方式：`String str = new String(new StringBuffer("abc"));`
　　反射方式：`String str = (String)constructor.newInstance(new StringBuffer("abc"));`

####**Class.newInstance()方法**
　　例子:`String obj = (String)Class.forName("java.lang.String").newInstance();`
　　该方法内部先得到**默认**的构造方法，然后用该构造方法创建实例对象。
　　该方法内部的具体代码用到了缓存机制来保存默认构造方法的实例对象。
###**4.成员变量的反射(Field类)**
　　1. Field类代表某个类中的一个成员变量
　　2. 问题：得到的Field对象是对应到类上面的成员变量，还是对应到对象上的成员变量？
　　类只有一个，而该类的实例对象有多个，如果是与对象关联，那关联那个对象呢？所以字段Field代表的是类的成员变量，而不是具体的变量。
　　`Field getDeclaredField(String name)`返回一个 Field 对象，该对象反映此 Class 对象所表示的类或接口的指定已声明字段。
　　`Field[] getDeclaredFields()`返回 Field 对象的一个数组，这些对象反映此 Class 对象所表示的类或接口所声明的所有字段。 
　　`Field getField(String name)`返回一个 Field 对象，它反映此 Class 对象所表示的类或接口的指定公共成员字段。 
　　`Field[] getFields()`返回一个包含某些 Field 对象的数组，这些对象反映此 Class 对象所表示的类或接口的所有可访问公共字段。

代码：
```Java
import java.lang.reflect.*;
public class ReflectTest {
    public static void main(String[] args) throws Exception{
        ReflectPoint pt1 = new ReflectPoint(3,5);
        Field fieldY = pt1.getClass().getField("y");
        //fieldY的值是多少？是5？错，fieldY不是对象身上的变量，而是类上的，要用它去取某个对象上对应的值
        System.out.println(fieldY.get(pt1));//5
        //fieldX为私有的，.getField("x")取不到，得用.getDeclaredField("x")
        Field fieldX = pt1.getClass().getDeclaredField("x");
        //由于私有，不可访问，使用setAccessible(true)，可以表示可以取
        fieldX.setAccessible(true);
        //能够取到x的值
        System.out.println(fieldX.get(pt1));//3
        
    }
}
class ReflectPoint {
    private int x;
    public int y;
    public ReflectPoint(int x, int y) {
        super();
        this.x = x;
        this.y = y;
    }
}
```
####**成员变量反射的综合案例**
代码：
```Java
import java.lang.reflect.*;
public class ReflectTest {
    public static void main(String[] args) throws Exception{
        ReflectPoint pt1 = new ReflectPoint(3,5);
        changeStringValue(pt1);
        System.out.println(pt1);
    }
    private static void changeStringValue(Object obj) throws Exception {
        Field[] fields = obj.getClass().getFields();
        for(Field field : fields){
            //对应同一份字节码，所以用==
            if(field.getType()==String.class){
                String oldValue = (String)field.get(obj);
                String newValue = oldValue.replace('b', 'a');
                field.set(obj,newValue);
            }
        }
    }
}
class ReflectPoint {
    private int x;
    public int y;
    public String str1 = "ball";
    public String str2 = "basketball";
    public String str3 = "itcast";
    public ReflectPoint(int x, int y) {
        super();
        this.x = x;
        this.y = y;
    }
    public String toString() {
        return str1+":"+str2+":"+str3;
    }
}
```
###**5. 成员方法的反射**
　　Method类代表某个类中的一个成员方法
　　得到类中的某一个方法：
　　`Method charAt = Class.forName("java.lang.String").getMethod("charAt", int.class);`
　　调用方法：
　　　　通常方式：System.out.println(str.charAt(1));
　　　　反射方式：System.out.println(charAt.invoke(str, 1));
　　如果传递给Method对象的invoke()方法的第一个参数为null，这有着什么样的意义呢？说明该Method对象对应的是一个**静态方法**！

####**jdk1.4和jdk1.5的invoke方法的区别**
>* Jdk1.5：public Object invoke(Object obj,Object... args)
>* Jdk1.4：public Object invoke(Object obj,Object[] args)

　　即按jdk1.4的语法，需要将一个数组作为参数传递给invoke方法时，数组中的每个元素分别对应被调用方法中的一个参数，所以，调用charAt方法的代码也可以用Jdk1.4改写为 charAt.invoke("str", new Object[]{1})形式。
代码：
```Java
import java.lang.reflect.*;
public class ReflectTest {
    public static void main(String[] args) throws Exception{
        Method methodCharAt = String.class.getMethod("charAt", int.class);
        System.out.println(methodCharAt.invoke("abc",1 ));
        //System.out.println(methodCharAt.invoke(null,1 ));如果为null则为静态方法
        //System.out.println(methodCharAt.invoke("abc",new Object[]{2} ));1.5以前，没有可变参数，这么写
    }
}
```
###**6. 对接收数组参数的成员方法进行反射**
　　问题：平时如何调用一个类的main方法？
```Java
public class ReflectTest {
	public static void main(String[] args){
		TestArgument.main(new String[]{"aa", "bb", "cc"});
	}
}
class TestArgument{
	public static void main(String[] args){
		for(String arg : args){
			System.out.println(arg);
		}
	}
}
```
　　问题：当不知道类的名字时（类的名字作为参数），如何调用该类的main方法？
　　用反射的方式，这就是反射的作用，可以将类的名字作为参数传入，启动main方法。
　　启动Java程序的main方法的参数是一个字符串数组，即`public static void main(String[] args)`，通过反射方式来调用这个main方法时，如何为invoke方法传递参数呢？
　　按jdk1.5的语法，**整个数组是一个参数**，而按jdk1.4的语法，数组中的**每个元素对应一个参数**，当把一个字符串数组作为参数传递给invoke方法时，javac会到底按照哪种语法进行处理呢？
　　jdk1.5肯定要兼容jdk1.4的语法，会按jdk1.4的语法进行处理，**即把数组打散成为若干个单独的参数**。所以，在给main方法传递参数时，不能使用代码mainMethod.invoke(null,new String[]{"xxx"})，javac只把它当作jdk1.4的语法进行理解，而不把它当作jdk1.5的语法解释，因此会出现参数类型不对的问题。

　　解决办法：
>* mainMethod.invoke(null,new Object[]{new String[]{"xxx"}});
* mainMethod.invoke(null,(Object)new String[]{"xxx"}); //编译器会作特殊处理，编译时不把参数当作数组看待，也就不会数组打散成若干个参数了

代码：
```Java
import java.lang.reflect.*;
public class ReflectTest {
    public static void main(String[] args) throws Exception{
        TestArguments.main(new String[]{"sadfds","hjhgj"});
        System.out.println("-----------------------------");
        
        String startingClassName = args[0];
        Method mainMethod = Class.forName(startingClassName).getMethod("main", String[].class);
        mainMethod.invoke(null, (Object)new String[]{"sadfds","hjhgj"});
        System.out.println("-----------------------------");
        //不只是main方法，普通方法也一样
        Method testMethod = Class.forName(startingClassName).getMethod("test", String[].class);
        testMethod.invoke(null, new String[]{"sadfds","hjhgj"});
    }
}
class TestArguments{
    public static void main(String[] args){
        for(String arg : args){
            System.out.println(arg);
        }
    }
    public static void test(String[] args){
        for(String arg : args){
            System.out.println(arg);
        }
    }
}
```
###**7. 数组与Object的关系及其反射类型**
>* 具有**相同维数**和**元素类型**的数组属于同一个类型，即具有相同的Class实例对象。
>* 代表数组的Class实例对象的getSuperClass()方法返回的父类为Object类对应的Class。
>* 基本类型的一维数组可以被当作Object类型使用，不能当作Object[]类型使用。
>* 非基本类型的一维数组，既可以当做Object类型使用，又可以当做Object[]类型使用。

代码举例：
```Java
import java.lang.reflect.*;
public class ReflectTest {
    public static void main(String[] args) throws Exception{
        int[] a1 = new int[3];
        int[] a2 = new int[4];
        int[][] a3 = new int[2][3];
        String[] a4 = new String[3];
        
        System.out.println(a1.getClass()==a2.getClass());//true
        System.out.println(a1.getClass().getName());//[I(数组整形)
        //父类的名字
        System.out.println(a1.getClass().getSuperclass().getName());//java.lang.Object
        System.out.println(a2.getClass().getSuperclass().getName());//java.lang.Object
        System.out.println(a3.getClass().getSuperclass().getName());//java.lang.Object
        System.out.println(a4.getClass().getSuperclass().getName());//java.lang.Object
        System.out.println(String.class.getSuperclass().getName());//java.lang.Object
        System.out.println(StringBuffer.class.getSuperclass().getName());//java.lang.Object
        //发现a1，a2，a3，a4的super都是objcet所以可以类型转换
        //结论：int[]一维数组是Object,String是Object，int是基本类型，不是Object
        Object aObj1 = a1;
        Object aObj2 = a4;
        //Object[] aObj3 = a1;不对 int 不可以直接转为object
        Object[] aObj4 = a3;
        Object[] aObj5 = a4;
    }
}
```
　　Arrays.asList()方法处理int[]和String[]时的差异。 
```Java
System.out.println(Arrays.asList(a1)); //[[I@55f33675]
System.out.println(Arrays.asList(a4)); //[aa, bb, cc]
```
####**数组的反射应用**
　　Array工具类用于完成对数组的反射操作。
```Java
    printObject(a4);//aa bb cc  
    printObject("zyz");//zyz## 标题 ##
  
    private static void printObject(Object obj){  
        Class clazz = obj.getClass();  
        if(clazz.isArray()){  
            int len  = Array.getLength(obj);  
            for(int i=0;i<len;i++){  
                System.out.println(Array.get(obj, i));  
            }  
        }else{  
            System.out.println(obj);  
        }  
    }  
```


## 获取类的泛型类型

	public static Class getActualType(Class entity){
		//ParameterizedType:参数类型
		ParameterizedType parameterizedType = (ParameterizedType) entity.getGenericSuperclass();
		//获得真实类型参数数组  [0]表示第一个
		Class entityClass = (Class) parameterizedType.getActualTypeArguments()[0];
		return entityClass;
	}
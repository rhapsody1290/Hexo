---
title: Java类加载器深入讲解

date: 2016-07-29 00:00:00

categories:
- Java

tags:
- Java
- 类加载器

---
## 什么是类加载器?

　　加载类的工具，把硬盘上.class文件加载到内存，并进行一些处理，得到字节码

## 类加载器有什么作用?

　　当程序需要的某个类，那么需要通过类加载器把类的二进制加载到内存中，类加载器也是Java类

## 类加载器之间的父子关系和管辖范围。

![类加载器之间的父子关系和管辖范围图](http://my.csdn.net/uploads/201207/12/1342057661_9458.png)

```Java
#获得类加载器的名字
ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
while (classLoader != null) {
    System.out.println(classLoader.getClass().getName());
    classLoader = classLoader.getParent();
}
System.out.println(classLoader);
```

![此处输入图片的描述](http://my.csdn.net/uploads/201207/12/1342057738_7123.png)
##4. 类加载器的委托机制:
### **类的加载**
1. 当Java虚拟机要加载一个类时，到底派出哪个类加载器去加载呢?
　　①首先 **当前线程的类加载器** 去加载线程中的第一个类.
　　②如果类A中引用了类B，Java虚拟机将使用加载类A的类加载器加载类B
　　③还可以直接调用ClassLoader.loadClass()方法来指定某个类加载器去加载某个类.
2. 每个类加载器加载类时，又**先委托给其上级类加载器**
　　**当所有祖宗类加载器没有加载到类，回到发起者类加载器**，如果还加载不了，则抛出ClassNotFoundException异常。它不会去找发起者类加载器的儿子，因为没有getChild()方法，即使有，有那么多的儿子交给那一个呢?所以干脆就不交给儿子处理了。

### **委托机制有什么好处?**

* 集中管理，如果我们写了几个类加载器，都去加载某个类，那么内存中就有多份这个类的字节码
* 安全。系统类由系统的类加载器加载

### **能不能自己写一个类叫java.lang.System?**
　　为了不让我们写System类，类加载采用委托机制，这样可以保证爸爸优先，也就是使用的永远是爸爸的(系统的)System类，而不是我们写的System类.
## 编写自己的类加载器

```Java
public static void main(String[] args) throws Exception {
        String srcPath = args[0];
        String destDir = args[1];
        FileInputStream fis = new FileInputStream(srcPath);
        String destFileName = srcPath.substring(srcPath.lastIndexOf('\\')+1);
        String destPath = destDir + "\\" + destFileName;
        FileOutputStream fos = new FileOutputStream(destPath);
        cypher(fis，fos);
        fis.close();
        fos.close();
    }
 
    /**
     * 加密方法，同时也是解密方法
     * @param ips
     * @param ops
     * @throws Exception
     */
    private static void cypher(InputStream ips ，OutputStream ops) throws Exception{
        int b = -1;
        while((b=ips.read())!=-1){
            ops.write(b ^ 0xff);//如果是1就变成0，如果是0就变成1
        }
    }
```
　　然后在新建一个类，通过上面的方法将新建的类的字节码进行加密:
```Java
public class ClassLoaderAttachment extends Date { //为什么要继承Date待会再说?
    public String toString(){
        return "hello，itcast";
    } 
}
```
　　并在工程里新建一个文件夹，用来保存加密后的class文件.
　　![此处输入图片的描述](http://my.csdn.net/uploads/201207/12/1342057789_4367.png)
　　那么这就需要使用我们自己的类加载器来进行解密了.
```Java
public class MyClassLoader extends ClassLoader{
    public static void main(String[] args) throws Exception {
        String srcPath = args[0];
        String destDir = args[1];
        FileInputStream fis = new FileInputStream(srcPath);
        String destFileName = srcPath.substring(srcPath.lastIndexOf('\\')+1);
        String destPath = destDir + "\\" + destFileName;
        FileOutputStream fos = new FileOutputStream(destPath);
        cypher(fis，fos);
        fis.close();
        fos.close();
    }
 
    /**
     * 加密方法，同时也是解密方法
     * @param ips
     * @param ops
     * @throws Exception
     */
    private static void cypher(InputStream ips ，OutputStream ops) throws Exception{
        int b = -1;
        while((b=ips.read())!=-1){
            ops.write(b ^ 0xff);//如果是1就变成0，如果是0就变成1
        }
    }
 
    private String classDir;
 
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String classFileName = classDir + "\\" + name.substring(name.lastIndexOf('.')+1) + ".class";
        try {
            FileInputStream fis = new FileInputStream(classFileName);
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            cypher(fis，bos);
            fis.close();
            System.out.println("aaa");
            byte[] bytes = bos.toByteArray();
            return defineClass(bytes， 0， bytes.length);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
 
    public MyClassLoader(){
    }
 
    public MyClassLoader(String classDir){
        this.classDir = classDir;
    }
}
```
测试运行代码:
```Java
Class clazz = new MyClassLoader("myClass").loadClass("ClassLoaderAttachment");
//此处不能在使用ClassLoaderAttachment因为一旦用了之后，
//系统的类加载器就会去加载，导致失败，所以该类就继承了Date类了.
Date date = (Date)clazz.newInstance();
System.out.println(date);
```
运行结果：
![此处输入图片的描述](http://my.csdn.net/uploads/201207/12/1342057844_5083.jpg)
## 一个类加载器的高级问题:
　　我们知道tomcat服务器，是一个大大的java程序，那么它就必须在JVM上运行.这个大大的java程序内部也写了很多类加载器，它用这些类加载器去加载一些特定的类.注入servlet类.
下面我们新建一个javaweb工程，新建一个servlet程序.
```Java    
    public void doGet(HttpServletRequest request， HttpServletResponse response)
            throws ServletException， IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        ClassLoader classload = this.getClass().getClassLoader();
        while (classload != null) {
            out.println(classload.getClass().getName()+"<br>");
            classload = classload.getParent();
        }
        out.println();
        out.close();
    }
```
　　然后配置服务器，部署应用程序，启动tomcat服务器.在页面访问我们这个servlet，在页面打印的
结果如下图所示:
![此处输入图片的描述](http://my.csdn.net/uploads/201207/12/1342057884_8748.png)
这是从小到大排序的.
现在呢?我想把该servlet打成jar包，放在ExtClassLoad类加载器加载的路径.
通过Eclipse即可完成
![此处输入图片的描述](http://my.csdn.net/uploads/201207/12/1342057935_6368.png)

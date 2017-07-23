---
title: Java中配置文件放在哪里

date: 2016-07-13 22:04:00

categories:
- Java

tags:
- Java

---
## 绝对路径与相对路径

　　Java中路径可分为相对路径和绝对路径两种方式。<font color='red'>相对路径</font>是相对当前工作目录，例如当使用命令
  
	C:\Users\Asus>java MyClass xxx.properties  

要求在`C:\Users\Asus`目录下有xxx.properties文件，而在使用

	C:\>java MyClass xxx.properties

时则要求在C盘根目录下有xxx.properties文件，所以使用相对路径是**飘忽不定**的，不建议使用。  
　　但如果给出<font color='red'>绝对路径</font>，`D:\xxx.properties`,当工程给用户时，若用户没有D盘，就会出现问题。  
　　**综上所述**，仍旧采用绝对路径的方式来确定资源文件的地址，但是需要通过函数方法得到项目路径,在通过字符串连接的方式拼接得到绝对路径。

	//此时config.properties文件放在工程文件根目录下，即选择工程右键后，新建config.properties文件
	InputStream ips = new FileInputStream("config.properties");
	Properties properties = new Properties();
	properties.load(ips);
	System.out.println(properties.getProperty("name"));  

## Java中比较常用的加载资源的方式

　　类加载器把字节码加载到内存中，即它可以加载.class文件，也可以加载普通文件。  
　　当工程完成后，不会将工程目录下中的Src目录给用户（怎么可能会把源代码给用户），而是将bin目录下的文件，一些字节码等文件在用户电脑上运行。    
　　eclipse会自动将Java文件编译，并存放字节码在 `工程目录/bin/包名`目录下，Java文件对应编译后的字节码，普通文件(如config.properties文件)仍原封不动拷贝过去。  
　　类加载器会在``classPath``中搜索。  
　　<font color="red">使用类加载器时，默认的主目录是src</font>

	public class ReflectTest {
		public static void main(String[] args) throws Exception{
			/*InputStream ips = new FileInputStream("config.properties");
			Properties properties = new Properties();
			properties.load(ips);
			System.out.println(properties.getProperty("name"));   */ 
			InputStream ipsInputStream = ReflectTest.class.getClassLoader().getResourceAsStream("com/qianming/config.properties");
			
			//在class类中直接有一个getResourceAsStream方法，路径名相对当前包名的相对路径，所以这里可以直接写config.properties
			//InputStream ipsInputStream = ReflectTest.class.getResourceAsStream("config.properties");
	        //如果路径中写上'/'，则此时需相对根目录写路径
	        //InputStream ipsInputStream = ReflectTest.class.getResourceAsStream("/com/qianming/config.properties");
	
			Properties properties = new Properties();
			properties.load(ipsInputStream);
			System.out.println(properties.getProperty("name"));
		}
	}

***注意:***  
　　这里使用的路径是`com/qianming/config.properties`，即在`包/资源文件`的形式不能在com前加 **"/"**,记忆就行，否则会报错。


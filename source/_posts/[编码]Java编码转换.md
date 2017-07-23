---
title: Java编码转换

date: 2017-02-26 12:48:00

categories:
- 编码

tags:
- 编码

---

首先要注意，**字符串在java内存中总是按unicode编码**

## getBytes(charset)

Java中个 **getBytes(charset)** 函数的作用是将字符串所表示的是字符按照charset编码，并以字节表示

比如"中文"，在Java内存中用unicode编码，表示为0x4E2D 0x6587

![](http://i.imgur.com/6Maz7lJ.png)

如果charset为"gbk"，则被编码为0xD6D0 0xCEC4

![](http://i.imgur.com/MdevX2J.png)

如果charset为"utf-8"，则被编码为0xE4B8AD 0xE69687 

![](http://i.imgur.com/IEZP1tV.png)

**如果是charset为"iso-8859-1"，由于无法编码(不能存储中文)，最后返回0x3F 0x3F，即？？**

![](http://i.imgur.com/zUOwsyX.png)

如果包含英文字幕和中文字幕，如"a中国"，看一下**不同编码英文、中文所占的字节数**

在Java内存中用unicode编码，每个字符都编码为两个字节，占的位置一样大，长度为3
用gbk编码英文占1个字节，中文占两个字节
用utf8编码英文占1个字节，中文占3个字节
用iso885901编码英文占1个字节，不能存储中文

![](http://i.imgur.com/Wtw98Er.png)

## new String(charset)

这是java字符串处理的另一个标准函数，和上一个函数的作用相反，**将字节数组按照charset编码进行组合识别**，最后转换为unicode存储。参考上述getBytes的例子，"gbk" 和"utf8"都可以得出正确的结果"4e2d 6587"，但iso8859-1最后变成了"003f 003f"（两个问号）。 

因为utf8可以用来表示/编码所有字符，所以new String( str.getBytes( "utf8" ), "utf8" ) === str，即完全可逆。 

	String a = "中文";
	byte[] gbk = a.getBytes("gbk");
	byte[] utf8 = a.getBytes("utf-8");
	byte[] iso8859_1 = a.getBytes("iso-8859-1");
	
	System.out.println(new String(gbk,"gbk"));
	System.out.println(new String(utf8,"utf-8"));
	System.out.println(new String(iso8859_1,"iso-8859-1"));

输出

	中文
	中文
	??

## <font color='red'>使用ISO-8859-1存储中文</font>

http://blog.chinaunix.net/uid-26808060-id-5692454.html

在以上例子中，尝试将中文字符使用iso-8859-1编码，字节序都是0xFF，再用iso-8859-1解码后会得到？？，这是因为iso-8859-1是单字节编码，不能直接存储中文

在网络传输或其它应用中常常有统一的中间件，因此需要把其它类型的数据转换为中间件的类型。如在Java Web中，浏览器将编码后的字节流发送给Tomcat，在tomcat内部编码转换，并处理后，然后发送给接收端。**这时候如果采用不同的编码可能会出现未成预料的问题，如乱码**

**案例1：**

我们用socket传输String类型的数据时，**全部都统一为UTF-8编码**，这样比较可以避免一个"中文乱码"的问题，这也是**最推荐的解决中文乱码的方式。**

	//发送端，socket发送
	String sendString = "北京";
	System.out.println("发送数据:" + sendString);
	byte[] sendBytes = sendString.getBytes("UTF-8");
	
	//接受端，socket接收
	String recString = new String(sendBytes, "UTF-8");
	System.out.println("接收数据:" + recString);

**案例2**

但是想要发送的数据本身就是byte[]类型的。如果将其通过UTF-8编码转换为中间件String类型就会出现问题，（解码）如：

![](http://i.imgur.com/ibOwYKn.png)

**需要采用单字节的编码方式进行转换：**

String sendString=new String(  bytes ,"UTF-8"); 改为 String sendString=new String(  bytes , "ISO-8859-1" );

byte[] Mybytes=isoString.getBytes("UTF8"); 改为 byte[] Mybytes=isoString.getBytes(  "ISO-8859-1" );

这样所需要的字节就有恢复了

	//发送端
	byte[] bytes = new byte[]{50, 0, -1, 28, -24};
	String sendString = new String(bytes, "iso-8859-1");
	byte[] sendBytes = sendString.getBytes("UTF8");
	//接收端
	String recString = new String(sendBytes, "UTF-8");
	byte[] Mybytes = recString.getBytes("iso-8859-1");//发送的时候用iso-8859-1编码，解码是反过程

sendString显示的是拉丁字符，是iso-8859-1单字节解码后的结果

## 避免中文乱码及乱码产生的原因★★★

1、最简单避免中文乱码的方式是统一编码，**推荐整个系统中采用统一的utf-8编码方式**
2、编码、解码的字符集不一致会出现乱码，如采用utf-8编码，但是用gbk解码，则会产生乱码，如下：

	String str = "上海";
	byte[] b = str.getBytes("utf-8");
	String s = new String(b,"gbk");
	System.out.println(s);//输出结果：涓婃捣

**而且产生乱码不能恢复到正确的字符：**

![](http://i.imgur.com/jcha13T.png)

如"上"先用utf-8编码，再用gbk解码，在用gbk编码，字节序列b和gbk不同，说明由于不同编码长度不同，**产生的中文乱码不可恢复**

<font color='red'>**3、ISO8859-1的中文乱码**</font>


1、出现？？

	String send = "上海";
	byte[] t = send.getBytes("iso-8859-1");
	String recv = new String(t,"iso-8859-1");

原因是**正确接收到中文字符**，但在存储的时候编码成iso-8859-1，此时返回的字节都是0x3F，再次用iso-8859-1（不管用什么编码，如utf-8）解码出来的时候都出现？？。**一般出现？？，都是因为字符是中文，调用了byte[] t = send.getBytes("iso-8859-1");语句，乱码无法恢复**

2、出现拉丁乱码ä¸æµ·，将接收到的字符按照iso-8859-1编码得到字节序，**即为原来utf-8编码后的正确字节**，再进行utf-8解码即可（gbk同理）。**如果出现拉丁乱码，乱码可以恢复，先用拉丁编码，在用正确的字符集解码**

	String str = "上海";
	byte[] b = str.getBytes("utf-8");
	System.out.println("汉字：" + str + "utf-8编码形式：" + Arrays.toString(b));
	String s = new String(b, "iso8859-1");
	System.out.println("与之对应的iso8859-1解码形式：" + s);

	byte[] b1 = s.getBytes("iso8859-1");
	System.out.println(s + "与之对应的iso8859-1编码形式：" + Arrays.toString(b1));
	String s1 = new String(b1, "utf-8");
	System.out.println("解析后："+ s1);

结果：

	汉字：上海utf-8编码形式：[-28, -72, -118, -26, -75, -73]
	与之对应的iso8859-1解码形式：ä¸æµ·
	ä¸æµ·与之对应的iso8859-1编码形式：[-28, -72, -118, -26, -75, -73]
	解析后：上海

3、出现汉字的乱码：中文经过utf-8编码后，再进行iso-8809-1编码，最后用gbk解码。**产生原因是发送端、接收端编解码方式不一致**，在字节流没有差错的情况下可以恢复

    //发送
    String str = "上海";
    byte[] b = str.getBytes("utf-8");
    System.out.println("汉字：" + str + "utf-8编码形式：" + Arrays.toString(b));
    //中间件
    String s = new String(b, "iso8859-1");
    System.out.println("与之对应的iso8859-1解码形式：" + s);
    byte[] b1 = s.getBytes("iso8859-1");
    System.out.println(s + "与之对应的iso8859-1编码形式：" + Arrays.toString(b1));
    //接收
    String s1 = new String(b1, "gbk");
    System.out.println("发送接收端编码不一致，解析错误结果："+ s1);
    String s2 = new String(b1, "utf-8");
    System.out.println("发送接收端编码不一致，解析错误结果："+ s2);

结果：

	汉字：上海utf-8编码形式：[-28, -72, -118, -26, -75, -73]
	与之对应的iso8859-1解码形式：ä¸æµ·
	ä¸æµ·与之对应的iso8859-1编码形式：[-28, -72, -118, -26, -75, -73]
	发送接收端编码不一致，解析错误结果：涓婃捣
	发送接收端编码不一致，解析错误结果：上海

4、iso-8809-1在中间件中使用，传送中文字符，或传送字节

1、传送中文

发送的中英文字符经过**中文编码**后变成字节流发送到网络，**中间件采用ISO-8859-1编码**，接收端和发送端采用**统一的编码**，能够正确解码

    //发送
    String str = "上海";
    byte[] b = str.getBytes("utf-8");
    System.out.println("发送：" + str + "utf-8编码形式：" + Arrays.toString(b));
    //中间件
    String s = new String(b, "iso8859-1");
    System.out.println("与之对应的iso8859-1解码形式：" + s);
    byte[] b1 = s.getBytes("iso8859-1");
    System.out.println(s + "与之对应的iso8859-1编码形式：" + Arrays.toString(b1));
    //接收
    String s1 = new String(b1, "utf-8");
    System.out.println("接收："+ s1);

结果：

	发送：上海utf-8编码形式：[-28, -72, -118, -26, -75, -73]
	与之对应的iso8859-1解码形式：ä¸æµ·
	ä¸æµ·与之对应的iso8859-1编码形式：[-28, -72, -118, -26, -75, -73]
	接收：上海

2、传送字节
	
如果需要传送字节，在发送端利用iso-8859-1解码，获得字符串。**问题转换为如何正确传输该字符串**，此时可以将字符串用utf-8编码发送字节流，接收端用utf-8解码得到字符串。字符串正确传递后，在用iso-8859-1解码即能获得正确的字节


如果只有发送端和接收端：

	//发送端
	byte[] bytes = new byte[]{50, 0, -1, 28, -24};
	String sendString = new String(bytes, "iso-8859-1");
	byte[] sendBytes = sendString.getBytes("UTF8");
	//接收端
	String recString = new String(sendBytes, "UTF-8");
	byte[] Mybytes = recString.getBytes("iso-8859-1");

如果考虑了中间件，代码如下：

	//发送端
	byte[] bytes = new byte[]{50, 0, -1, 28, -24};
	System.out.println("传送的字节：" + Arrays.toString(bytes));
	String send = new String(bytes, "iso-8859-1");//发送所需的字节流问题转化为发送该字节流使用iso-8859-1解码后的字符串
	System.out.println("传送的字节经过iso-8859-1解码：" + send);
	byte[] send_bytes = send.getBytes("utf-8");
	System.out.println("字符经过utf-8编码：" + Arrays.toString(send_bytes));
	//中间件
	String t = new String(send_bytes, "iso-8859-1");
	System.out.println("中间件用iso-8859-1解码的字符：" + t);
	byte[] t_bytes = t.getBytes("iso-8859-1");
	System.out.println("中间件用iso-8859-1编码的字节" + Arrays.toString(t_bytes));
	//接收端
	String recv = new String(t_bytes, "utf-8");//正确接收字符串
	System.out.println("接收端用utf-8解码的字符：" + recv);
	byte[] recv_bytes = recv.getBytes("iso-8859-1");//获得原先所需的字节流
	System.out.println("接收端恢复编码：" + Arrays.toString(recv_bytes));

输出结果：

	传送的字节：[50, 0, -1, 28, -24]
	传送的字节经过iso-8859-1解码：2 ÿè
	字符经过utf-8编码：[50, 0, -61, -65, 28, -61, -88]
	中间件用iso-8859-1解码的字符：2 Ã¿Ã¨
	中间件用iso-8859-1编码的字节[50, 0, -61, -65, 28, -61, -88]
	接收端用utf-8解码的字符：2 ÿè
	接收端恢复编码：[50, 0, -1, 28, -24]


4、**中间件采用iso-8859-1编码不会出现乱码**，这也是许多软件都默认采用latin1作为编码的原因，如Tomcat默认是iso-8859-1编码，Mysql默认也是iso-8859-1编码

## Mysql 几个参数的意义

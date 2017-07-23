---
title: IP地址与整数转换

date: 2016-09-06 9:52:00

categories:
- 网络安全

tags:
- 网络安全

---
## Java中byte, int的转换  

### int -> byte

* 可以直接使用强制类型转换: byte b = (byte) aInt;
* 这个操作是直接截取int中最低一个字节，如果int大于255，则值就会变得面目全非了

### byte -> int

* 这里有两种情况，一种是要求***保持值不变***，例如进行***数值计算***，可采用强制类型转换：int i = (int) aByte; 
* 另一种是要求***保持最低字节中各个位不变***，***3个高字节全部用 0填充***，例如***进行编解码操作*** ，则需要采用位操作：int i = b & 0xff; 

### 理解★★★★★

* Java中使用System.out.println(int);因为println中形参为int类型，会将字符类型强行转化为整型
	  byte b = (byte) 127;
	  System.out.println(b);

* <font color='red'>字符类型转化为整型时有两种方式，一种是强转，高位用符号位填充；另一种是通过'与'的方式，高位用0填充</font>
* 如果采用强转的方式，即int i = (int) aByte；Java默认byte是有符号的，int也是有符号的，强转后int的3个高字节全部用 ***符号位*** 填充，直接进行强转强转后数值不变，适合用于数值计算	
* 采用位操作int i = bByte & 0xff 时，bByte和0xff***先分别转换成整型，再进行 '与' 操作***，得到一个32位整数，此时高位用0填充，常用于编码中
	    11111111 11111111 11111111 11101010 (a=-22)
	  &
	    00000000 00000000 00000000 11111111 (0xff)
	  ---------------------------------------------------------------------
	  = 00000000 00000000 00000000 11101010 (234)

* <font color='red'>"字节变量 & 0xff" 相当于将字节转换为无符号整型</font>
* Java中byte为***有符号类型***，占用一个字节，大小范围为-128~127，使用System.out.println(int)，首先会将byte强转为int类型，输出值保持不变；如果超过这个范围，会出现<font color='blue'>***溢出***</font>，正数变负数，负数变正数
	* 结果不变
		  byte b = (byte) 127;//结果127，不变
		  因为127（0111 1111），首位为符号位，127为单字节正数最大，（-128）10000 0000为负数最小
	* 正数变负数
		  byte b = (byte) 128;//结果-128
		  1000 0000（128），在Java中byte为有符号数，一字节，首位为1，则这个数是负数，打印结果是-128
	* 负数变正数
		  byte b = (byte) -129;//结果127
		  -129（1 0111 1111），超出范围溢出，首先进行截断，结果为0111 1111，变为正数127

### int InputStream.read() 读取字节流内部实现

	该函数返回一个int类型，范围从0至255，如果到达流末尾，返回-1。通过ByteArrayInputStream的源码可以看到是如何从byte转到int
	public synchronized int read() {
	    return (pos < count) ? (buf[pos++] & 0xff) : -1;
	}

### int <-> byte[]（一个整数4个字节）

方法一：（推荐）

<pre>
<font color='red'>//将整数转为一个byte数组，字节数组的低位是整型的低字节位</font>
public static byte[] toByteArray(int iSource, int iArrayLen) {
    byte[] bLocalArr = new byte[iArrayLen];
    for (int i = 0; (i < 4) && (i < iArrayLen); i++) {
        bLocalArr[i] = (byte) ((iSource >> 8 * i) & 0xFF);
    }
    return bLocalArr;
}
</pre>

* 移位
* 转成整型，与操作，取低字节
* 截断，转成byte

方法二：

	private static byte[] int2byte(int i) {
        byte[] bytes = new byte[4];
        bytes[0] = (byte) (0xff & i);
        bytes[1] = (byte) ((0xff00 & i) >> 8);
        bytes[2] = (byte) ((0xff0000 & i) >> 16);
        bytes[3] = (byte) ((0xff000000 & i) >> 24);
        return bytes;
    }

### byte[] <-> int

方法一：（推荐）

<pre>
<font color='red'>// 将byte数组转为一个整数,字节数组的低位是整型的低字节位</font>
public static int toInt(byte[] bRefArr) {
    int iOutcome = 0;
    byte bLoop;

    for (int i = 0; i < bRefArr.length; i++) {
        bLoop = bRefArr[i];
        iOutcome += (bLoop & 0xFF) << (8 * i);//左移并加起来
    }
    return iOutcome;
}
</pre>

* 转成整型
* 移位
* 相加

方法二：

	private static int byte2Int(byte[] bytes) {
	        int n = bytes[0] & 0xFF;
	        n |= ((bytes[1] << 8) & 0xFF00);
	        n |= ((bytes[2] << 16) & 0xFF0000);
	        n |= ((bytes[3] << 24) & 0xFF000000);
	        return n;
	    }

## IpUtil（IP地址与整数转换工具）

### 框图

![](http://i.imgur.com/TWMLfKy.png)

### IP地址转换为整数

* Ip地址可以分成四段，每段可以是8位无符号整数即0~255
* Ip地址转化为字节数组，数字出现的顺序不变，举个例子：
	  192.168.1.1 -> {192，168，1，1}
* ***字节数组的低位为整数的低位***，把每段拆分成一个二进制形式，组合起来，然后把这个***32位二进制数*** 变成一个***32位无符号整数***
	  1（0000 0001），1（0000 0001），168（1010 1000），192（1100 0000） ->16885952(32位整数)

### 整数转换为IP地址

* 把这个整数转换成一个无符号32位二进制数，举个例子：
	  16885852（0000 0001，0000 0001，1010 1000，1100 0000）
* ***字节数组的低位为整数的低位***，
	  192（1100 0000），168（1010 10000），1（0000 0001），1（0000 0001）
* 从左到右，每八位进行一下分割，得到4段8位的二进制数，把这些二进制数转换成整数然后加上"."就可以了
	  切割，转成十进制：192.168.1.1

### 直接转换（不利用IpUtil，利于理解）★★★★★★

	String ip = "192.168.1.1";
	String[] ipSection = ip.split("\\.");
	//ip->int
	int result = 0;
	for(int i = 0; i < ipSection.length; i++){
		result += Integer.parseInt(ipSection[i]) << 8*i;
	}
	System.out.println(result);//16885952
	//int -> ip
	byte[] bytes = new byte[4];
	for(int i = 0; i < ipSection.length; i++){
		bytes[i] = (byte)((result >> 8*i) & 0xff);
	}
	for(byte b : bytes){
		System.out.println(b & 0xff);//192,168,1,1
	}

### IpUtil（转换成字节数组）

	public class IpUtil {
	
	    /**
	     * 将字符串型ip转成int型ip
	     * @param strIp
	     * @return
	     */
	    public static int Ip2Int(String strIp){
	        String[] ss = strIp.split("\\.");
	        if(ss.length != 4){
	            return 0;
	        }
	        byte[] bytes = new byte[ss.length];
	        for(int i = 0; i < bytes.length; i++){
	            bytes[i] = (byte) Integer.parseInt(ss[i]);
	        }
	        return byte2Int(bytes);
	    }

	    /**
	     * 将int型ip转成String型ip
	     * @param intIp
	     * @return
	     */
	    public static String int2Ip(int intIp){
	        byte[] bytes = int2byte(intIp);
	        StringBuilder sb = new StringBuilder();
	        for(int i = 0; i < 4; i++){
	            sb.append(bytes[i] & 0xFF);
	            if(i < 3){
	                sb.append(".");
	            }
	        }
	        return sb.toString();
	    }
	
	    private static byte[] int2byte(int i) {
	        byte[] bytes = new byte[4];
	        bytes[0] = (byte) (0xff & i);
	        bytes[1] = (byte) ((0xff00 & i) >> 8);
	        bytes[2] = (byte) ((0xff0000 & i) >> 16);
	        bytes[3] = (byte) ((0xff000000 & i) >> 24);
	        return bytes;
	    }

	    private static int byte2Int(byte[] bytes) {
	        int n = bytes[0] & 0xFF;
	        n |= ((bytes[1] << 8) & 0xFF00);
	        n |= ((bytes[2] << 16) & 0xFF0000);
	        n |= ((bytes[3] << 24) & 0xFF000000);
	        return n;
	    }
	
	    public static void main(String[] args) {
	        String ip1 = "192.168.0.1";
	        int intIp = Ip2Int(ip1);
	        String ip2 = int2Ip(intIp);
	        System.out.println(ip2.equals(ip1));
	    }
	}


## 参考文件
java中byte, int的转换 - 小柯的日志 - 网易博客
http://freewind886.blog.163.com/blog/static/661924642011810236100/
Java将ip字符串转换成整数
http://outofmemory.cn/code-snippet/4837/Java-jiang-ip-charaeter-chuan-turn-huancheng-integer


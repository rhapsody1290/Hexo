---
title: Java基础常用

date: 2016-07-08 14:49:00

categories:
- Java

tags:
- Java

---
## 读取文件根目录
	根目录在工程目录
	InputStream ips = new FileInputStream("src/xx.properties");
	类加载器根目录在src下
	InputStream ips = test.class.getClassLoader().getResourceAsStream("xx.properties");
	maven项目根目录为resources
	InputStream ips = test.class.getClassLoader().getResourceAsStream("xx.properties");

## 读取文本文件

	//读取文件
    String realPath = this.getServletContext().getRealPath("record.txt");
    FileReader fileReader = new FileReader(realPath);
    BufferedReader br = new BufferedReader(fileReader);
    String nums = br.readLine();
    //一定要关闭流
    br.close();
    fileReader.close();
    out.println(nums);
    //写入文件
    FileWriter fileWriter = new FileWriter(realPath);
    BufferedWriter bw = new BufferedWriter(fileWriter);
    bw.write(String.valueOf(Integer.parseInt(nums) + 1));
    bw.close();
    fileWriter.close();

## 从控制台读取数据

	BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
	System.out.println(br.readLine());

## Java读取资源文件

	/**
	 * 文件类型是UTF-8，在String生成字符时需指定解码方式为utf-8；
	 */
	InputStream ips = null;
	try {
		ips = MyServer.class.getClassLoader().getResourceAsStream("com/main/戏曲.txt");
		int hasRead = 0;
		byte[] buffer = new byte[1024];
		StringBuffer content = new StringBuffer();
		while((hasRead = ips.read(buffer)) > 0){
			content.append(new String(buffer, 0, hasRead, "utf-8"));
		}
		System.out.println(content);
	} catch (Exception e) {
		e.printStackTrace();
	}finally{
		if (ips != null) {
	        try {
				ips.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
	    }
	}

## 字符串

	四舍五入
	float totalMoney = 123.124f;
	BigDecimal b = new BigDecimal(totalMoney);  
	totalMoney  = b.setScale(2,BigDecimal.ROUND_HALF_UP).floatValue();  
	保留两位小数
	String.format("%.2f", totalMoney)

## 生成指定长度的随机字符串	

	//生成指定长度的随机字符串	
	public static String GenRandomString(int length){
		Random random = new Random();	
		char[] charArray = "abcdefghijklmnopqrstuvwxyz1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
		char[] randomChar = new char [length];
		for(int i = 0; i < randomChar.length; i++){
			randomChar[i] = charArray[random.nextInt(charArray.length)];
		}		
		return new String(randomChar);	
	}

## JDBC

	MAVEN依赖：http://blog.csdn.net/earbao/article/details/44900083
	连接步骤：http://www.cnblogs.com/hongten/archive/2011/03/29/1998311.html

***JDBC查询***

	//到数据库中验证
	/*
	* 1.加载驱动
	* 2.得到连接
	* 3.创建PrepareStatment
	* 4.执行操作
	* 5.根据结果做处理
	* */
	Connection connection = null;
	PreparedStatement ps = null;
	ResultSet rs = null;
	
	try {
		//1.加载驱动
		Class.forName("com.mysql.jdbc.Driver");
		//2.获得连接
		connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/servlet_users_manager","root","root");
		//3.创建prepareStatement
		ps = connection.prepareStatement("select * from users where username = ? and passwd = ?");
		ps.setObject(1,username);
		ps.setObject(2,password);
		//4.执行操作
		rs = ps.executeQuery();
		//5.根据结果做处理
		if(rs.next()){
			//说明用户合法
			System.out.println(rs.getString("username") + " " + rs.getString("passwd"));
			request.getRequestDispatcher("/MainFrame").forward(request,response);
		}else{
			request.getRequestDispatcher("/Login").forward(request,response);
		}
	} catch (Exception e) {
		e.printStackTrace();
	}finally {
		//关闭资源
		if(rs != null){   // 关闭记录集
			try{
				rs.close();
			}catch(Exception e){
				e.printStackTrace() ;
			}
		}
		if(ps != null){   // 关闭声明
			try{
				ps.close() ;
			}catch(Exception e){
				e.printStackTrace() ;
			}
		}
		if(connection != null){  // 关闭连接对象
			try{
				connection.close() ;
			}catch(Exception e){
				e.printStackTrace() ;
			}
		}
	}

## 分页

***定义四个分页变量***

    pageNow   表示第几页,该变量是由用户来决定,因此变化
    pageSize  每页显示几条记录,由程序指定,也可以由用户定制
    pageCount 表示共有多少页, 该变量是计算出来->思考 怎样确定
    rowCount  共有多少条记录,该变量是查询数据库得到

***如何确定pageCount***	

    (1)
    if(rowCount% pageSize==0){
    	pageCount=rowCount/pageSize;
    }else{
    	pageCount= rowCount/pageSize+1;
    }
    试试: 比如 users表 9 条记录 pageSize=3  =>pageCount=3
      	 比如 users表 10 条记录 pageSize=3  =>pageCount=4
    (2)
    上面的算法等价于
    pageCount=rowCount% pageSize==0 ? rowCount/pageSize: rowCount/pageSize+1;
    
    该运算称为三目运算
    (3)
    更简单的算法是:★★★★★
    pageCount=(rowCount-1)/pageSize+1;
    试试: 比如 users表 9 条记录 pageSize=3  =>pageCount=3
      	 比如 users表 11 条记录 pageSize=3  =>pageCount=4
	为什么？
		如果pageSize整除rowCount，值不需要加一；
		如果pageSize不整除rowCount，值需要加一。
		我们可以使rowCount-1，保证每次都不能整除，这样可以得到统一公式:
			pageCount = （rowCount-1）/pageSize + 1

### 项目中传递参数

客户端传递：pageNow、pageSize（为保证程序健壮性，服务器中设置pageNow和pageSize的默认值）
服务器传递：数据和rowCount

## MD5

利用Java自带的MD5加密

	class MD5Util {
	    public final static String MD5(String s) {
	        char hexDigits[] = { '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',
	                'a', 'b', 'c', 'd', 'e', 'f' };
	        try {
	            byte[] strTemp = s.getBytes();
	            MessageDigest mdTemp = MessageDigest.getInstance("MD5");
	            mdTemp.update(strTemp);
	            byte[] md = mdTemp.digest();
	            int j = md.length;
	            char str[] = new char[j * 2];
	            int k = 0;
	            for (int i = 0; i < j; i++) {
	                byte byte0 = md[i];
	                str[k++] = hexDigits[byte0 >>> 4 & 0xf];
	                str[k++] = hexDigits[byte0 & 0xf];
	            }
	            return new String(str);
	        } catch (Exception e) {
	            return null;
	        }
	    }
	
	    public static void main(String[] args) {
	        // MD5_Test aa = new MD5_Test();
	        System.out.print(MD5Util.MD5("qm"));
	    }
	}

## 验证码

	public class check_code extends HttpServlet {
	    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        // 7.禁止浏览器缓存随机图片
	        response.setDateHeader("Expires", -1);
	        response.setHeader("Cache-Control", "no-cache");
	        response.setHeader("Pragma", "no-cache");
	        // 6.通知客户机以图片方式打开发送过去的数据
	        response.setHeader("Content-Type", "image/jpeg");
	        // 1.在内存中创建一副图片
	        BufferedImage image = new BufferedImage(60,30,BufferedImage.TYPE_INT_RGB);
	        // 2.向图片上写数据
	        Graphics g = image.getGraphics();
	        // 设背景色
	        g.setColor(Color.BLACK);
	        g.fillRect(0, 0, 60, 30);
	        // 3.设置写入数据的颜色和字体
	        g.setColor(Color.RED);
	        g.setFont(new Font(null, Font.BOLD, 20));
	        // 4.向图片上写数据
	        String num = makeNum();
	        //这句话就是把随机生成的数值，保存到session
	        request.getSession().setAttribute("check_code", num); //通过session就可以直接去到随即生成的验证码了
	        g.drawString(num, 5, 22);
	        // 5.把写好数据的图片输出给浏览器
	        ImageIO.write(image, "jpg", response.getOutputStream());
	    }
	    //该函数时随机生成4位数字
	    public String makeNum() {
	        Random r = new Random();
	        //9999999 可以生成7位
	        String num = r.nextInt(9999) + "";
	        StringBuffer sb = new StringBuffer();
	        //如果不够4位，前面补零
	        for (int i = 0; i < 4 - num.length(); i++) {
	            sb.append("0");
	        }
	        num = sb.toString() + num;
	        return num;
	    }
	
	    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	        this.doPost(request, response);
	    }
	}

## list转string[]

方法一：单个元素转换

	//ArrayList
	ArrayList<String> array = new ArrayList<String>();
	array.add("aaa");
	array.add("bbb");
	array.add("ccc");
	//转化成Object数组
	Object[] objs = array.toArray();
	//新建数组
	String[] strings = new String[array.size()];
	//每个元素类型转换
	int i = 0;
	for(Object obj : objs){
		if(obj instanceof String){
			strings[i++] = (String)obj;
		}
	}
	for(String s : strings){
		System.out.println(s);
	}


方法二：整个转

	//ArrayList
	ArrayList<String> array = new ArrayList<String>();
	array.add("aaa");
	array.add("bbb");
	array.add("ccc");
	
	String[] strings = new String[array.size()];
	array.toArray(strings);
	for(String s : strings){
		System.out.println(s);
	}


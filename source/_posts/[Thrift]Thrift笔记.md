---
title: Thrift笔记

date: 2017-02-13 16:21:00

categories:
- Thrift

tags:
- Thrift

---

## Thrift

http://www.ibm.com/developerworks/cn/java/j-lo-apachethrift/#ibm-pcon
目前流行的服务调用方式有很多种，例如基于 SOAP 消息格式的 Web Service，基于 JSON 消息格式的 RESTful 服务等。**其中所用到的数据传输方式包括 XML，JSON 等，然而 XML 相对体积太大，传输效率低，JSON 体积较小，新颖，但还不够完善**。本文将介绍由 Facebook 开发的远程服务调用框架 Apache Thrift，它采用接口描述语言定义并创建服务，支持可扩展的跨语言服务开发，所包含的代码生成引擎可以在多种语言中，如 C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, Smalltalk 等创建高效的、无缝的服务，其传输数据采用二进制格式，相对 XML 和 JSON 体积更小，对于高并发、大数据量和多语言的环境更有优势

### Thrift流程

1、使用 IDL 语法定义接口 Hello.thrift

    namespace java service.demo
    service Hello{
        string helloString(1:string para)
    }

service.demo：包名
Hello：生成的类名
helloString：定义的接口名

2、使用thrift编译器生成Java代码

命令：

    thrift -r --gen java tutorial.thrift

thrift编译器下载地址：

    http://www.apache.org/dyn/closer.cgi?path=/thrift/0.9.3/thrift-0.9.3.exe

3、生成对应语言的接口代码及服务调用的底层通信细节，生成的对应语言代码与编译器位于同一目录

    package service.demo;
    
    class Hello{
    
        //生成的对应语言的接口
        public interface Iface {
            public String helloString(String para) throws org.apache.thrift.TException;
        }
        
        //###客户端调用逻辑
        public static class Client extends org.apache.thrift.TServiceClient implements Iface {
            ...
            //实现的接口方法
            public String helloString(String para) throws org.apache.thrift.TException
            {
                //向服务器发送请求方法+参数
                send_helloString(para);
                //接受服务器的请求
                return recv_helloString();
            }
        
            //发送请求方法+参数，其中helloString为调用服务器的方法名，args为方法参数
            public void send_helloString(String para) throws org.apache.thrift.TException
            {
                helloString_args args = new helloString_args();
                args.setPara(para);
                sendBase("helloString", args);
            }
        
            //接受服务器返回的结果
            public String recv_helloString() throws org.apache.thrift.TException
            {
                helloString_result result = new helloString_result();
                receiveBase(result, "helloString");
                if (result.isSetSuccess()) {
                    return result.success;
                }
                throw new org.apache.thrift.TApplicationException(org.apache.thrift.TApplicationException.MISSING_RESULT, "helloString failed: unknown result");
            }
            ...
        }
        
    }
    //### 服务端处理逻辑
    public static class Processor<I extends Iface> extends org.apache.thrift.TBaseProcessor<I> implements org.apache.thrift.TProcessor {
    
        private static <I extends Iface> Map<String,  org.apache.thrift.ProcessFunction<I, ? extends  org.apache.thrift.TBase>> getProcessMap(Map<String,  org.apache.thrift.ProcessFunction<I, ? extends  org.apache.thrift.TBase>> processMap) {
            processMap.put("helloString", new helloString());
            return processMap;
        }
        
            public static class helloString<I extends Iface> extends org.apache.thrift.ProcessFunction<I, helloString_args> {
            public helloString() {
            super("helloString");
        }
        
        public helloString_args getEmptyArgsInstance() {
            return new helloString_args();
        }
        
        protected boolean isOneway() {
            return false;
        }
        
        public helloString_result getResult(I iface, helloString_args args) throws org.apache.thrift.TException {
            helloString_result result = new helloString_result();
                result.success = iface.helloString(args.para);
                return result;
            }
        }
    
    }
    
4、创建接口实现类

服务器创建接口实现类

    public HelloServiceImpl implements Hello.Iface{
        @Override
        public String helloString(String para) throws org.apache.thrift.TException{
            System.our.println("Hello");
        }
    }

5、启动Thrift服务器，HelloServiceImpl 作为具体的处理器传递给 Thrift 服务器

	public class Server {
	    /**
	     * 启动 Thrift 服务器
	     * @param args
	     */
	    public static void main(String[] args) throws TTransportException {
	        try {
	            //1、传输层，设置服务端口为 7911
	            TServerSocket serverTransport = new TServerSocket(7911);
	            //2、协议，工厂为 TBinaryProtocol.Factory
	            TBinaryProtocol.Factory proFactory = new TBinaryProtocol.Factory();
	            // 处理器
	            TProcessor processor = new Hello.Processor(new HelloImpl());
	
	            //3、服务端类型，关联端口、协议、处理器与 Hello 服务的实现
	            TThreadPoolServer.Args tArgs = new TThreadPoolServer.Args(serverTransport);
	            tArgs.processor(processor);
	            tArgs.protocolFactory(proFactory);
	
	            TServer server = new TThreadPoolServer(tArgs);
	            System.out.println("Start server on port 7911...");
	            server.serve();
	        } catch (TTransportException e) {
	            e.printStackTrace();
	        }
	    }
	}

6、客户端连接到端口，发送请求调用方法，服务端响应结果，客户端接受结果

	public class Client {
	    /**
	     * 调用 Hello 服务
	     *
	     * @param args
	     */
	    public static void main(String[] args) {
	        try {
	            // 设置调用的服务地址为本地，端口为 7911
	            TTransport transport = new TSocket("localhost", 7911);
	            transport.open();
	            // 设置传输协议为 TBinaryProtocol
	            TProtocol protocol = new TBinaryProtocol(transport);
	            //客户端调用
	            Hello.Client client = new Hello.Client(protocol);
	            // 调用服务的 helloVoid 方法
	            System.out.println(client.helloString("客户端传入参数"));
	            transport.close();
	        } catch (TTransportException e) {
	            e.printStackTrace();
	        } catch (TException e) {
	            e.printStackTrace();
	        }
	    }
	}

### Thrift架构

![](http://i.imgur.com/h1KGvdq.jpg)

如图所示，图中黄色部分是用户实现的业务逻辑，褐色部分是根据 Thrift 定义的服务接口描述文件生成的客户端和服务器端代码框架，红色部分是根据 Thrift 文件生成代码实现数据的读写操作。红色部分以下是 Thrift 的传输体系、协议以及底层 I/O 通信，使用 Thrift 可以很方便的定义一个服务并且选择不同的传输协议和传输层而不用重新生成代码。

Thrift 服务器包含用于绑定协议和传输层的基础架构，它提供阻塞、非阻塞、单线程和多线程的模式运行在服务器上，可以配合服务器 / 容器一起运行，可以和现有的 J2EE 服务器 /Web 容器无缝的结合。

![](http://i.imgur.com/nZ4Aoap.png)

该图所示是 HelloServiceServer 启动的过程以及服务被客户端调用时，服务器的响应过程。从图中我们可以看到，程序调用了 TThreadPoolServer 的 serve 方法后，server 进入阻塞监听状态，其阻塞在 TServerSocket 的 accept 方法上。当接收到来自客户端的消息后，服务器发起一个新线程处理这个消息请求，原线程再次进入阻塞状态。在新线程中，服务器通过 TBinaryProtocol 协议读取消息内容，调用 HelloServiceImpl 的 helloVoid 方法，并将结果写入 helloVoid_result 中传回客户端。

![](http://i.imgur.com/KqC4Vo6.png)

该图所示是 HelloServiceClient 调用服务的过程以及接收到服务器端的返回值后处理结果的过程。从图中我们可以看到，程序调用了 Hello.Client 的 helloVoid 方法，在 helloVoid 方法中，通过 send_helloVoid 方法发送对服务的调用请求，通过 recv_helloVoid 方法接收服务处理请求后返回的结果。

### 数据类型

Thrift 脚本可定义的数据类型包括以下几种类型：

	基本类型：
	bool：布尔值，true 或 false，对应 Java 的 boolean
	byte：8 位有符号整数，对应 Java 的 byte
	i16：16 位有符号整数，对应 Java 的 short
	i32：32 位有符号整数，对应 Java 的 int
	i64：64 位有符号整数，对应 Java 的 long
	double：64 位浮点数，对应 Java 的 double
	string：未知编码文本或二进制字符串，对应 Java 的 String
	结构体类型：
	struct：定义公共的对象，类似于 C 语言中的结构体定义，在 Java 中是一个 JavaBean
	容器类型：
	list：对应 Java 的 ArrayList
	set：对应 Java 的 HashSet
	map：对应 Java 的 HashMap
	异常类型：
	exception：对应 Java 的 Exception
	服务类型：
	service：对应服务的类

### ★协议

Thrift 可以让用户选择客户端与服务端之间传输通信协议的类别，在传输协议上总体划分为**文本 (text)** 和**二进制 (binary)** 传输协议，为节约带宽，提高传输效率，一般情况下使用二进制类型的传输协议为多数，有时还会使用基于文本类型的协议，这需要根据项目 / 产品中的实际需求。常用协议有以下几种：

* TBinaryProtocol —— 二进制编码格式进行数据传输

	  // 设置协议工厂为 TBinaryProtocol.Factory
	  TBinaryProtocol.Factory proFactory = new TBinaryProtocol.Factory();

* TCompactProtocol —— 高效率的、密集的二进制编码格式进行数据传输，构建 TCompactProtocol 协议的服务器和客户端只需替换案例中 TBinaryProtocol 协议部分即可
	  TCompactProtocol.Factory proFactory = new TCompactProtocol.Factory();

* TJSONProtocol —— 使用 JSON 的数据编码协议进行数据传输，构建 TJSONProtocol 协议的服务器和客户端只需替换案例中 TBinaryProtocol 协议部分即可
  	TJSONProtocol.Factory proFactory = new TJSONProtocol.Factory();

* TSimpleJSONProtocol —— 只提供 JSON 只写的协议，适用于通过脚本语言解析

### ★传输层

常用的传输层有以下几种：

* TSocket —— 使用阻塞式 I/O 进行传输，是最常见的模式（案例）


	//服务端
	TServerSocket serverTransport = new TServerSocket(7911);
	//客户端
	TTransport transport = new TSocket("localhost", 7911);

* TFramedTransport —— 使用非阻塞方式，按块的大小进行传输，类似于 Java 中的 NIO，若使用 TFramedTransport 传输层，其**服务器必须修改为非阻塞的服务类型**，**客户端只需替换案例中 TTransport 部分**

服务器

	
	TNonblockingServerTransport serverTransport; 
	serverTransport = new TNonblockingServerSocket(10005); 
	Hello.Processor processor = new Hello.Processor(new HelloServiceImpl()); 
	TServer server = new TNonblockingServer(processor, serverTransport); 
	System.out.println("Start server on port 10005 ..."); 
	server.serve();

客户端

  	TTransport transport = new TFramedTransport(new TSocket("localhost", 10005));

* TNonblockingTransport —— 使用非阻塞方式，用于构建异步客户端
使用方法请参考 Thrift 异步客户端构建

### ★服务端类型

常见的服务端类型有以下几种：

* TSimpleServer —— 单线程服务器端使用标准的阻塞式 I/O


	TServerSocket serverTransport = new TServerSocket(7911); 
	TProcessor processor = new Hello.Processor(new HelloServiceImpl()); 
	TServer server = new TSimpleServer(processor, serverTransport); 
	System.out.println("Start server on port 7911..."); 
	server.serve();

* TThreadPoolServer —— 多线程服务器端使用标准的阻塞式 I/O（案例）


	// 设置服务端口为 7911
	TServerSocket serverTransport = new TServerSocket(7911);
	// 设置协议工厂为 TBinaryProtocol.Factory
	TBinaryProtocol.Factory proFactory = new TBinaryProtocol.Factory();
	// 处理器
	TProcessor processor = new Hello.Processor(new HelloImpl());
	
	//关联端口、协议、处理器与 Hello 服务的实现
	TThreadPoolServer.Args tArgs = new TThreadPoolServer.Args(serverTransport);
	tArgs.processor(processor);
	tArgs.protocolFactory(proFactory);
	
	TServer server = new TThreadPoolServer(tArgs);
	System.out.println("Start server on port 7911...");
	server.serve();

* TNonblockingServer —— 多线程服务器端使用非阻塞式 I/O
使用方法请参考 Thrift 异步客户端构建

### Thrift 异步客户端构建

Thrift 提供非阻塞的调用方式，可构建异步客户端。在这种方式中，Thrift 提供了新的类 TAsyncClientManager 用于管理客户端的请求，在一个线程上追踪请求和响应，同时通过接口 AsyncClient 传递标准的参数和 callback 对象，服务调用完成后，callback 提供了处理调用结果和异常的方法。
**首先我们看 callback 的实现：**

	package service.callback; 
	import org.apache.thrift.async.AsyncMethodCallback; 
	
	public class MethodCallback implements AsyncMethodCallback { 
		Object response = null; 
		
		public Object getResult() { 
		    // 返回结果值
		    return this.response; 
		} 
		
		// 处理服务返回的结果值
		@Override 
		public void onComplete(Object response) { 
		    this.response = response; 
		} 
		// 处理调用服务过程中出现的异常
		@Override 
		public void onError(Throwable throwable) { 
		
		} 
	}

如代码所示，onComplete 方法接收服务处理后的结果，此处我们将结果 response 直接赋值给 callback 的私有属性 response。onError 方法接收服务处理过程中抛出的异常，此处未对异常进行处理。

**创建非阻塞服务器端实现代码**，将 HelloServiceImpl 作为具体的处理器传递给异步 Thrift 服务器，代码如下：

	package service.server; 
	import org.apache.thrift.server.TNonblockingServer; 
	import org.apache.thrift.server.TServer; 
	import org.apache.thrift.transport.TNonblockingServerSocket; 
	import org.apache.thrift.transport.TNonblockingServerTransport; 
	import org.apache.thrift.transport.TTransportException; 
	import service.demo.Hello; 
	import service.demo.HelloServiceImpl; 
	
	public class HelloServiceAsyncServer { 
		/** 
		 * 启动 Thrift 异步服务器
		 * @param args 
		 */ 
		public static void main(String[] args) { 
		    TNonblockingServerTransport serverTransport; 
		    try { 
		        serverTransport = new TNonblockingServerSocket(10005); 
		        Hello.Processor processor = new Hello.Processor( 
		                new HelloServiceImpl()); 
		        TServer server = new TNonblockingServer(processor, serverTransport); 
		        System.out.println("Start server on port 10005 ..."); 
		        server.serve(); 
		    } catch (TTransportException e) { 
		        e.printStackTrace(); 
		    } 
		} 
	}

HelloServiceAsyncServer 通过 java.nio.channels.ServerSocketChannel 创建非阻塞的服务器端等待客户端的连接。

**创建异步客户端实现代码**，调用 Hello.AsyncClient 访问服务端的逻辑实现，将 MethodCallback 对象作为参数传入调用方法中，代码如下：

	package service.client; 
	import java.io.IOException; 
	import org.apache.thrift.async.AsyncMethodCallback; 
	import org.apache.thrift.async.TAsyncClientManager; 
	import org.apache.thrift.protocol.TBinaryProtocol; 
	import org.apache.thrift.protocol.TProtocolFactory; 
	import org.apache.thrift.transport.TNonblockingSocket; 
	import org.apache.thrift.transport.TNonblockingTransport; 
	import service.callback.MethodCallback; 
	import service.demo.Hello; 
	
	public class HelloServiceAsyncClient { 
		/** 
		 * 调用 Hello 服务
		 * @param args 
		 */ 
		public static void main(String[] args) throws Exception { 
		    try { 
		        TAsyncClientManager clientManager = new TAsyncClientManager(); 
		        TNonblockingTransport transport = new TNonblockingSocket( 
		                "localhost", 10005); 
		        TProtocolFactory protocol = new TBinaryProtocol.Factory(); 
		        Hello.AsyncClient asyncClient = new Hello.AsyncClient(protocol, 
		                clientManager, transport); 
		        System.out.println("Client calls ....."); 
		        MethodCallback callBack = new MethodCallback(); 
		        asyncClient.helloString("Hello World", callBack); 
		        Object res = callBack.getResult(); 
		        while (res == null) { 
		            res = callBack.getResult(); 
		        } 
		        System.out.println(((Hello.AsyncClient.helloString_call) res) 
		                .getResult()); 
		    } catch (IOException e) { 
		        e.printStackTrace(); 
		    } 
		} 
	}
HelloServiceAsyncClient 通过 java.nio.channels.Socketchannel 创建异步客户端与服务器建立连接。在本文中异步客户端通过以下的循环代码实现了同步效果，读者可去除这部分代码后再运行对比。

异步客户端实现同步效果代码段

	Object res = callBack.getResult();
	// 等待服务调用后的返回结果
	while (res == null) {
	   res = callBack.getResult();
	}

我们可以构建一个 TNonblockingServer 服务类型的服务端，在客户端构建一个 TFramedTransport 传输层的同步客户端和一个 TNonblockingTransport 传输层的异步客户端，那么一个服务就可以通过一个 socket 端口提供两种不同的调用方式。有兴趣的读者可以尝试一下。
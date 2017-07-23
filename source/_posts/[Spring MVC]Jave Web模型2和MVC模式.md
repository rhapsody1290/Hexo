---
title: Jave Web模型2和MVC模式

date: 2016-09-07 23:00:00

categories:
- Spring MVC

tags:
- Spring MVC
- MVC
- 模型2

---

## 模型1和模型2介绍

* Java Web应用开发中有两种设计模型，为了方便，分别称为模型1和模型2
* 模型1是通过链接方式进行JSP页面之间的跳转
	* 方式***直接***，适合小型应用开发
	* 但在中型和大型应用中，这种方式会带来***维护***上的问题
* 模型2是基于MVC模式，是Java Web应用的推荐架构
	* 一个MVC模式的应用包括***模型***、***视图*** 和***控制器*** 3个模块
	* <font color='red'>***视图负责应用的展示，模型封装了应用的数据和业务逻辑，控制器负责接收用户输入，改变模型，以及调整视图的显示***</font>
	* <font color='blue'>***Servlet***</font> 或者<font color='blue'>***Filter***</font> 都可以充当控制器、Spring MVC和Struts1使用一个Servlet作为控制器，而Struts2 则使用一个Filter作为控制器
	* 大部分采用<font color='blue'>***JSP***</font> 页面作为应用的视图，当然也有别的技术
	* 模型会采用一个<font color='blue'>***JavaBean***</font> 来持有模型状态，并将业务逻辑放到一个Action类中。<font color='red'>***一个JavaBean必须拥有一个无参的构造器，通过get/set方法来访问参数，同时支持持久化***</font>

## 模型2应用 —— 产品信息保存

![](http://i.imgur.com/EarIrIZ.png)

### github

https://github.com/rhapsody1290/SpringMVC_study
包名为cn.apeius.product
### 工程目录

![](http://i.imgur.com/XHZXorE.png)

其中红框内的为`产品信息保存应用`相关文件

### ProductForm.jsp —— 填写产品信息表单

* ProductForm.jsp文件放在WEB-INF下，***不能通过URL直接访问***
* <font color='blue'>***一个控制器可以对应多个action(url)***</font>
* ***通过url：product_input.action***指向控制器ControllerServlet，由控制器根据相应的action跳转到页面ProductForm.jsp

***页面展示***

![](http://i.imgur.com/he0IokR.png)

***JSP代码***

<pre>
&lt;!DOCTYPE HTML>
&lt;html>
&lt;head>
&lt;title>Add Product Form&lt;/title>
&lt;style type="text/css">@import url(css/main.css);&lt;/style>
&lt;/head>
&lt;body>

&lt;div id="global">
&lt;form <font color='red'>action="product_save.action"</font> method="post">
	&lt;fieldset>
		&lt;legend>Add a product&lt;/legend>
			<font color='red'>&lt;p>
				&lt;label for="name">Product Name: &lt;/label>
				&lt;input type="text" id="name" name="name" 
					tabindex="1">
			&lt;/p></font>
			&lt;p>
				&lt;label for="description">Description: &lt;/label>
				&lt;input type="text" id="description" 
					name="description" tabindex="2">
			&lt;/p>
			&lt;p>
				&lt;label for="price">Price: &lt;/label>
				&lt;input type="text" id="price" name="price" 
					tabindex="3">
			&lt;/p>
			<font color='red'>&lt;p id="buttons">
				&lt;input id="reset" type="reset" tabindex="4">
				&lt;input id="submit" type="submit" tabindex="5" 
					value="Add Product">
			&lt;/p></font>
	&lt;/fieldset>
&lt;/form>
&lt;/div>
&lt;/body>
&lt;/html>
</pre>

* 表单的action为product_save.action，这个url会匹配到控制器ControllerServlet，调用相应的service方法完成产品保存工作，并完成页面的跳转，显示产品的详细信息
* fieldset可将表单内的元素进行分组
* 推荐使用标签 &lt;label for = '指向的标签name'>标签内容&lt;/label>，点击标签焦点就定位在指向的标签
* @import url(css/main.css)：css在web根目录下

### main.css —— 样式文件

	#global {
	    text-align: left;
		border: 1px solid #dedede;
		background: #efefef;
		width: 560px;
		padding: 20px;
		margin: 100px auto;
	}
	
	form {
	  font:100% verdana;
	  min-width: 500px;
	  max-width: 600px;
	  width: 560px;
	}
	
	form fieldset {
	  border-color: #bdbebf;
	  border-width: 3px;
	  margin: 0;
	}
	
	legend {
		font-size: 1.3em;
	}
	
	form label {
		width: 250px;
		display: block;
		float: left;
		text-align: right;
		padding: 2px;
	}
	
	#buttons {
		text-align: right;
	}

* ***灰色背景为一块 DIV，设置宽度560px，marigin：100px auto使 DIV 居中显示，页面下移100px***；高度不用设置，内部标签自动会撑开；另，设置padding为20px，DIV向外扩展，此时灰色面积尺寸为602（加上边框）
* ***字体verdana***在小字上仍有结构清晰端整、阅读辨识容易等高品质的表现
* <font color='red'>***怎样使得冒号对齐？***</font><font color='blue'>关键点是使设置标签长度一致，并使标签内容右对齐</font>；默认label是inline，长度设置无效，可采用如下两种办法：
	  方法一：使用inline-block来设置长度
	  form label {
	    width: 250px;
		display: inline-block;
		text-align: right;
		
	  }
	  方法二：使用block来设置长度
	  form label {
	      width: 250px;
	      display: block;
	      float: left; /*使用block后label占用一行，使用float让输入框移上来*/
	      text-align: right;
	      padding: 2px; /*不是关键*/
	  }
			


* ***心得：***DIV+CSS设计，外层DIV固定尺寸，内部元素相对外层进行设计，这个DIV为一个整体

### Product类和ProductForm类

***Product.java —— 产品信息JavaBean***

* Java规范中说要写入文件或是通过网络传输的对象必须是可序列化的，所以弄个标志接口Serializable 来标识一个类可以被序列化
* Product类实现了java.io.Serializationi接口，其实例可以安全地将数据保存到HttpSession中

<pre>
package cn.apeius.product.domain;
import java.io.Serializable;

public class Product <font color='red'>implements Serializable</font> {
    private static final long serialVersionUID = 748392348L;
	private String name;
    private String description;
    <font color='red'>private float price;</font>

    public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    public float getPrice() {
        return price;
    }
    public void setPrice(float price) {
        this.price = price;
    }
}
</pre>

***ProductForm.java***

* 表单类与HTML表单相映射，是后者在服务器端的代表
* ***Product和ProductForm类似，是否有必要存在？***表单对象会传递ServletRequest给其他组件，类似Validator，而ServletRequest是一个Servlet层的对象，<font color='blue'>***不应当暴露给应用的其它层***</font>
* 表单类不需要实现Serialization接口，因为表单对象很少存在HttpSession中

<pre>
package cn.apeius.product.form;

public class ProductForm {
    private String name;
    private String description;
    <font color='red'>private String price;</font>

    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    public String getPrice() {
        return price;
    }
    public void setPrice(String price) {
        this.price = price;
    }
}
</pre>

### ControllerServlet类★★★★★

控制器的操作：

	1、对URI处理获得action名
	2、根据action，创建表单对象，数据校验，创建领域对象，并执行领域对象的业务逻辑，例如将其持久化到数据库
	3、根据处理结果，跳转页面

<font color='blue'>url思考：</font>

* product_input.action、product_save.acton对应一个Servlet
* url的形式可以采用模块+操作的形式，举个例子：
	* 用户模块有login和logout操作，一个模块一个控制器
	* 可以采用/user/login.action，/user/logout.action对应匹配规则为/user/*的Servlet

***代码***

<pre>
package cn.apeius.product.servlet;
import cn.apeius.product.domain.Product;
import cn.apeius.product.form.ProductForm;

import java.io.IOException;

import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ControllerServlet extends HttpServlet {
    
    private static final long serialVersionUID = 1579L;

    @Override
    public void doGet(HttpServletRequest request, 
            HttpServletResponse response)
            throws IOException, ServletException {
        process(request, response);
    }

    @Override
    public void doPost(HttpServletRequest request, 
            HttpServletResponse response)
            throws IOException, ServletException {
        process(request, response);
    }

    private void process(HttpServletRequest request,
            HttpServletResponse response) 
            throws IOException, ServletException {
        /*
        1、URI的形式为：/应用名/资源名，例如/app10a/product_input
        2、一个Servlet可以对应多个action，本应用中访问/product_input.action和/product_save.action会进入ControllerServlet
        3、这个Servlet命名为ControllerServlet，是遵循了一个约定：所有Servlet类的名称都带有Servlet后缀
		*/
        <font color='red'>//1、对URI处理获得action名</font>
        String uri = request.getRequestURI();
        int lastIndex = uri.lastIndexOf("/");
        String action = uri.substring(lastIndex + 1); 
        <font color='red'>//2、根据action，创建表单对象，数据校验，创建领域对象，并并执行领域对象的业务逻辑，例如将其持久化到数据库</font>
        if (action.equals("product_input.action")) {
            //不需要调用service类执行业务逻辑
        } else if (action.equals("product_save.action")) {
            //创建表单对象
            ProductForm productForm = new ProductForm();
            //填充对象属性
            productForm.setName(request.getParameter("name"));
            productForm.setDescription(
                    request.getParameter("description"));
            productForm.setPrice(request.getParameter("price"));
            
            //创建模型
            Product product = new Product();
            product.setName(productForm.getName());
            product.setDescription(productForm.getDescription());
            try {
            	product.setPrice(Float.parseFloat(
            			productForm.getPrice()));
            } catch (NumberFormatException e) {
            }
            
            //调用service层的方法，保存产品，此处略
            
            //将产品模型保存在session，以便后续页面使用
            request.setAttribute("product", product);
        }

        <font color='red'>//3、根据处理结果，跳转页面</font>
        String dispatchUrl = null;
        if (action.equals("product_input.action")) {
            dispatchUrl = "/WEB-INF/jsp/ProductForm.jsp";
        } else if (action.equals("product_save.action")) {
            dispatchUrl = "/WEB-INF/jsp/ProductDetails.jsp";
        }
        if (dispatchUrl != null) {
            RequestDispatcher rd = 
                    request.getRequestDispatcher(dispatchUrl);
            rd.forward(request, response);
        }
    }
}

</pre>

### ProductDetails.jsp —— 显示产品详细信息

* ProductDetails.jsp 页面通过***表达式语言（EL）*** 访问 ***HttpServletRequest*** 所包含的对象

***代码***

	<!DOCTYPE HTML>
	<html>
	<head>
	<title>Save Product</title>
	<style type="text/css">@import url(css/main.css);</style>
	</head>
	<body>
	<div id="global">
	    <h4>The product has been saved.</h4>
	    <p>
	        <h5>Details:</h5>
	        Product Name: ${product.name}<br/>
	        Description: ${product.description}<br/>
	        Price: $${product.price}
	    </p>
	</div>
	</body>
	</html>

### 测试

可以通过url：

	http://localhost:8080/SpringMVC_study/product_input.action

访问应用
注意，可以将servlet控制器作为默认主页，使得在浏览器地址中仅输入域名+应用名，就可以访问到该Servlet控制器

	<jsp:forward page="/product_input.action"/>

### 缺点

* ***业务逻辑代码*** 都写在了Servlet控制器中，随着应用复杂度增加而不断膨胀
* <font color='red'>***哪些是业务逻辑？***</font>图中红框所示，根据不同的action，创建表单对象，数据校验，创建领域对象，并执行领域对象的业务逻辑；如果action多了，controller将会变得非常臃肿
![](http://i.imgur.com/VdCCYHr.png)

## 解耦控制器代码

### github

https://github.com/rhapsody1290/SpringMVC_study
包名为cn.apeius.product2

### 思路

* 将原来ControllerServlet中的***业务逻辑提取出来***
* 此时的Servlet变得更加专注，作用更像是一个***dispatcher***，***即检查每个url，根据访问的action，调用具体的controller完成业务逻辑，并完成页面跳转***，而非一个controller，因此改名为DispatcherServlet

### 工程目录

![](http://i.imgur.com/O4gRIs9.png)

### DispatcherServlet类

<font color='red'>***一个DispatcherServlet必须能够做如下事情：***</font>

* 根据URI调用相应的action
* 实例化正确的控制器类
* 调用控制器对象的响应方法
* 转到一个视图（JSP页面）

<pre>
public class DispatcherServlet extends HttpServlet {
    
    private static final long serialVersionUID = 748495L;

    @Override
    public void doGet(HttpServletRequest request, 
            HttpServletResponse response)
            throws IOException, ServletException {
        process(request, response);
    }

    @Override
    public void doPost(HttpServletRequest request, 
            HttpServletResponse response)
            throws IOException, ServletException {
        process(request, response);
    }

    private void process(HttpServletRequest request,
            HttpServletResponse response) 
            throws IOException, ServletException {

        //1、对URI处理获得action名
        String uri = request.getRequestURI();
        int lastIndex = uri.lastIndexOf("/");
        String action = uri.substring(lastIndex + 1);

        //2、根据action，调用具体的controller处理，DispatcherServlet只起到分派功能
        String dispatchUrl = null;
        if (action.equals("product_input.action")) {
        	<font color='red'>InputProductController controller = new InputProductController();
        	dispatchUrl = controller.handleRequest(request, response);</font>
        } else if (action.equals("product_save.action")) {
        	SaveProductController controller = new SaveProductController();
        	dispatchUrl = controller.handleRequest(request, response);
        }

        //3、根据处理结果，跳转页面
        if (dispatchUrl != null) {
            RequestDispatcher rd = 
                    request.getRequestDispatcher(dispatchUrl);
            rd.forward(request, response);
        }
    }
}
</pre>

### Controller接口

* 面向接口编程的方式，在DispatcherServlet中使用Controller来引用具体实现类，并调用其handleRequest方法完成具体业务逻辑操作

<pre>
public interface Controller {
	String handleRequest(HttpServletRequest request,
						 HttpServletResponse response);
}
</pre>

### InputProductController类和SaveProductController类

* 两个Controller都实现了Controller接口，Controller接口只有handleRequest方法
* Controller接口的实现类需要通过该方法访问到请求的HttpServletRequest和HttpServletResponse
* handleRequest返回结果为跳转文件路径

#### InputProductController类

* 直接返回输入产品的表单页面


	public class InputProductController implements Controller {
	
		public String handleRequest(HttpServletRequest request, 
				HttpServletResponse response) {
			return "/WEB-INF/jsp/ProductForm.jsp";
		}
	}

#### SaveProductController类

* 创建表单对象
* 创建模型
* 保存产品信息到数据库
* 返回跳转页面


	public class SaveProductController implements Controller {
	
		public String handleRequest(HttpServletRequest request, 
				HttpServletResponse response) {
	        ProductForm productForm = new ProductForm();
	        //填充表单数据
	        productForm.setName(
	                request.getParameter("name"));
	        productForm.setDescription(
	                request.getParameter("description"));
	        productForm.setPrice(request.getParameter("price"));
	        
	        //创建模型
	        Product product = new Product();
	        product.setName(productForm.getName());
	        product.setDescription(productForm.getDescription());
	        try {
	        	product.setPrice(Float.parseFloat(
	        			productForm.getPrice()));
	        } catch (NumberFormatException e) {
	        }
	
	        //将产品信息加入数据的代码，此处省略
	
	        request.setAttribute("product", product);
	        return "/WEB-INF/jsp/ProductDetails.jsp";
		}
	
	}

## 校验器

* Web应用执行action时，很重要的步骤是进行输入校验
* 因为校验工作很重要，Java社区专门发布了标准对Java世界的输入检验进行标准化

### github

https://github.com/rhapsody1290/SpringMVC_study
包名为cn.apeius.product2

### 工程目录

* 红色部分有修改，增加了ProductValidator类
* ProductForm.jsp展示输入校验的错误信息

![](http://i.imgur.com/zVKODsg.png)

### ProductValidator

* ProductValidator类中有一个validate方法，保证产品的字符串非空，价格是一个合理的数字
* validate返回一个包含错误信息的字符串列表，若返回一个空列表，则表示输入合法
* 在SaveProductController类中使用ProductValidator类


	public class ProductValidator {
		
		public List<String> validate(ProductForm productForm) {
			List<String> errors = new ArrayList<String>();
			String name = productForm.getName();
			if (name == null || name.trim().isEmpty()) {
				errors.add("Product must have a name");
			}
			String price = productForm.getPrice();
			if (price == null || price.trim().isEmpty()) {
				errors.add("Product must have a price");
			} else {
				try {
					Float.parseFloat(price);
				} catch (NumberFormatException e) {
					errors.add("Invalid price value");
				}
			}
			return errors;
		}
	}

### 新版的SaveProductController

* 首先对变淡类进行校验，如果校验发现有错误，则页面跳转到ProductForm.jsp；若没有错误，则创建一个Product对象

<pre>
public class SaveProductController implements Controller {

    public String handleRequest(HttpServletRequest request,
            HttpServletResponse response) {
        ProductForm productForm = new ProductForm();
        //填充表单属性
        productForm.setName(request.getParameter("name"));
        productForm.setDescription(request.getParameter("description"));
        productForm.setPrice(request.getParameter("price"));

        //校验表单
        ProductValidator productValidator = new ProductValidator();
        List<String> errors = productValidator.validate(productForm);
        if (errors.isEmpty()) {
            //创建领域对象Product
            Product product = new Product();
            product.setName(productForm.getName());
            product.setDescription(productForm.getDescription());
            product.setPrice(Float.parseFloat(productForm.getPrice()));

            //没有校验错误，执行action方法
            //保存产品信息到数据库的代码

            //将product存入request域中，便于后面页面显示
            request.setAttribute("product", product);
            return "/WEB-INF/jsp/ProductDetails.jsp";
        } else {
            //保存错误信息，在后续页面显示
            request.setAttribute("errors", errors);
			//保留表单信息，在后续页面显示
            request.setAttribute("form", productForm);
            return "/WEB-INF/jsp/ProductForm.jsp";
        }
    }

}
</pre>

### 新的ProductForm.jsp

* 用户提交了非法数据，页面将显示相应地错误信息

<pre>
&lt;%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
&lt;!DOCTYPE HTML>
&lt;html>
&lt;head>
	&lt;title>Add Product Form&lt;/title>
	&lt;style type="text/css">@import url(css/main.css);&lt;/style>
&lt;/head>
&lt;body>

&lt;div id="global">
	<font color='red'>&lt;c:if test="${requestScope.errors != null}">
		&lt;p id="errors">
			Error(s)!
		&lt;ul>
			&lt;c:forEach var="error" items="${requestScope.errors}">
				&lt;li>${error}&lt;/li>
			&lt;/c:forEach>
		&lt;/ul>
		&lt;/p>
	&lt;/c:if></font>
	&lt;form action="product_save.action" method="post">
		&lt;fieldset>
			&lt;legend>Add a product&lt;/legend>
			&lt;p>
				&lt;label for="name">Product Name: &lt;/label>
				&lt;input type="text" id="name" name="name"
					   tabindex="1">
			&lt;/p>
			&lt;p>
				&lt;label for="description">Description: &lt;/label>
				&lt;input type="text" id="description"
					   name="description" tabindex="2">
			&lt;/p>
			&lt;p>
				&lt;label for="price">Price: &lt;/label>
				&lt;input type="text" id="price" name="price"
					   tabindex="3">
			&lt;/p>
			&lt;p id="buttons">
				&lt;input id="reset" type="reset" tabindex="4">
				&lt;input id="submit" type="submit" tabindex="5"
					   value="Add Product">
			&lt;/p>
		&lt;/fieldset>
	&lt;/form>
&lt;/div>
&lt;/body>
&lt;/html>
</pre>

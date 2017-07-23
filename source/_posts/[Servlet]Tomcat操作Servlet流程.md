---
title: Tomcat操作Servlet流程

date: 2016-11-07 20:40:00

categories:
- Servlet

tags:
- Tomcat
- 标签2

---

## Tomcat调用Servlet的流程

* Listener的初始化最早，Filter次之。他俩的初始化都是在容器启动完成之前初始化的。 Servlet没有初始化，原因是没有匹配的请求进来。 如果想要servlet自动初始化，需要在指定的servlet中配置<load-on-startup>参数，没有此标签，默认启动时servlet不进行初始化。 

* 初始化的顺序跟Listener、Filter、Servlet在web.xml中的顺序无关。而多个Filter或多个Servlet的时候，谁的mapping在前面，谁先初始化。 

* 如果web.xml中配置了<context-param>，它用于向 ServletContext 提供键值对，即应用程序上下文信息。我们的 listener, filter 等在初始化时会用到这些上下文中的信息，context-param 配置节可写在任意位置，初始化顺序： context-param > Listener > Filter > Servlet

## 过滤器

* Filter的初始化方法在服务器启动时执行,过滤方法在请求发出后立即调用，可以过滤特定的URL
* 过滤器的URL匹配遵从最大长度匹配原则
* 一个URL匹配多个过滤器，**按照filter-mapping配置节点的出现顺序** 依次调用doFilter()

过滤器链：

参考：http://blog.sina.com.cn/s/blog_667ab8240101gfd6.html

有多个过滤器 filter1 filter2 filter3，组成一个过滤器链 FilterChain[filter1,filter2,filter3]

如何做到过滤器的依次执行？
调用过滤器链的doFilter参数，获取下一个过滤器链，然后执行下一个Filter的方法

<font color='red'>**前置方法和后置方法的执行顺序？**</font>

1、前置方法按照过滤器的调用顺序因此按顺序执行
2、后置方法的顺序与前置方法相反，即后调用的过滤器的后置方法先执行
3、**Chain.doFilter前的代码为访问Servlet前执行，Chain.doFilter后的代码访问Servlet后执行**（分析源代码）



1、下面是一个简单的时序图

![](http://i.imgur.com/J2JOzld.jpg)

2、对上面时序图中用到的主要类进行分析

1) ApplicationFilterChain类,有两个主要函数，下面是省略过的代码

	public voiddoFilter(request, response) {//暴露在外面的调用接口
		if( Globals.IS_SECURITY_ENABLED ) {
		      finalServletRequest req = request;
		      finalServletResponse res = response;
		      internalDoFilter(req,res);
		      return null;
		} else {
			internalDoFilter(request,response);
		}
	}

	private voidinternalDoFilter(request, response) {
		if (pos < n) {//判断是否还有filter需要执行
			ApplicationFilterConfig filterConfig = filters[pos++];
		    Filter filter = null;
			filter = filterConfig.getFilter();
			filter.doFilter(request, response, this);
			return ; //执行过滤器方法时，不执行以下代码，当pos=n，即执行完所有filter后执行servlet
		}

		//filter执行完后，执行servlet
     	if ((request instanceofHttpServletRequest) &&(response instanceof HttpServletResponse)) {
        	servlet.service((HttpServletRequest) request,(HttpServletResponse)response);	    
	}
	
	void addFilter(ApplicationFilterConfig filterConfig) {
		if (n == filters.length) {
		    ApplicationFilterConfig[] newFilters =
		        new ApplicationFilterConfig[n + INCREMENT];
		    System.arraycopy(filters, 0, newFilters, 0, n);
		    filters = newFilters;
		}
		filters[n++] = filterConfig;
	}

2) Servlet类的主要方法，以HttpServlet类为例，其主要方法是service(Request,Response)

	public void service(ServletRequest req, ServletResponse res)
        throws ServletException, IOException {
        HttpServletRequest  request;
        HttpServletResponse response;
        try {
            request = (HttpServletRequest) req;
            response = (HttpServletResponse) res;
        } catch (ClassCastException e) {
            throw new ServletException("non-HTTP request or response");
        }
        service(request, response);//内部的方法
    }

	protected void service(HttpServletRequest , HttpServletResponse)
        throws ServletException, IOException {
        String method = req.getMethod();
        if (method.equals(METHOD_GET)) {
            long lastModified = getLastModified(req);
            if (lastModified == -1) {
                doGet(req, resp); //常用的方法
            } else {
                long ifModifiedSince;
                try {
					ifModifiedSince = req.getDateHeader(HEADER_IFMODSINCE);
                } catch (IllegalArgumentException iae) {
               		ifModifiedSince = -1;
                }
                if (ifModifiedSince < (lastModified / 1000 * 1000)) {
                	maybeSetLastModified(resp, lastModified);
                    doGet(req, resp);
                } else {
					resp.setStatus(HttpServletResponse.SC_NOT_MODIFIED);
                }
            }
        } else if (method.equals(METHOD_HEAD)) {
            long lastModified = getLastModified(req);
            maybeSetLastModified(resp, lastModified);
            doHead(req, resp);
        } else if (method.equals(METHOD_POST)) {
            doPost(req, resp);//常用的方法
        } else if (method.equals(METHOD_PUT)) {
            doPut(req, resp);       
        } else if (method.equals(METHOD_DELETE)) {
            doDelete(req, resp);
        } else if (method.equals(METHOD_OPTIONS)) {
            doOptions(req,resp);
        } else if (method.equals(METHOD_TRACE)) {
            doTrace(req,resp);
        } else {
			String errMsg =lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[1];
            errArgs[0] = method;
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(HttpServletResponse.SC_NOT_IMPLEMENTED, errMsg);
        }
    }

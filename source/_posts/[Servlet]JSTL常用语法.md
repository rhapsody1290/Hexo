---
title: JSTL常用语法

date: 2017-01-17 09:30:00

categories:
- Servlet

tags:
- JSP

---

## Maven依赖

    <!--JSP相关-->
    <dependency>
        <groupId>jstl</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>

## JSP头

	<%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="UTF-8"%>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
	<%@ taglib prefix="fmt" uri="http://java.sun.com/jstl/fmt_rt"%>

## 常用标签

**PS：传入的对象除了一般的POJO对象，还可以是JSONObject和JSONArray**

&lt;c:if> 标签

	<!-- 判断集合不为空 -->
	<c:if test="${!empty list}">
		...
	</c:if>
	
	<!-- 元素判断 -->
	<c:if test="${salary > 2000}">
	   <p>我的工资为: <c:out value="${salary}"/><p>
	</c:if>

&lt;c:forEach> 标签

<pre>
&lt;c:forEach items="${list}" var="o" varStatus="status">
	&lt;!-- 下标 -->
	订单[${status.index + 1}] 的订单号是： ${o.orderNo}
&lt;/c:forEach>
</pre>

forEach嵌套

	<%
        Map<String, String[]> bigCities = new HashMap<String,String[]>();
        bigCities.put("Australia",new String[]{"Sydney","Melbourne","Perth"});
        bigCities.put("New Zealand",new String[]{"Auckland","Christchurch","Wellington"});
        bigCities.put("Indonesia",new String[]{"Jakarta","Surabaya","Medan"});
        request.setAttribute("bigCities",bigCities);
    %>

    <c:forEach var="mapItem" items="${bigCities}">
        ${mapItem.key} :
            <c:forEach var="city" items="${mapItem.value}">
                ${city}
            </c:forEach>
        <br/>
    </c:forEach>

遍历Map

	<%
        Map<String, String> capitals = new HashMap<String,String>();
        capitals.put("Indonesia","Jakarta");
        capitals.put("Malaysia","Kuala Lumpur");
        capitals.put("Thailand","Bangkok");
        request.setAttribute("capitals",capitals);
    %>

    <c:forEach var="capital" items="${capitals}">
        ${capital.key} ${capital.value} <br/>
    </c:forEach>

日期

	<fmt:formatDate value="${o.confirmDate}"  type="date" />
	<fmt:parseDate value="${now}" var="parsedEmpDate" pattern="dd-MM-yyyy" />

![](http://i.imgur.com/ij3bwfi.png)
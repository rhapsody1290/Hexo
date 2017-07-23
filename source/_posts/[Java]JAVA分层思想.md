---
title: JAVA分层思想

date: 2016-07-08 14:49:00

categories:
- Java

tags:
- Java

---

## 用户管理系统系统框架【需改造】

***存在的问题***  

　　LoginClServlet中太过臃肿，既有业务逻辑，又有对数据库的操作，后期难以进行维护

![这里写图片描述](http://img.blog.csdn.net/20160611213134633)

## 指导思想

① 业务逻辑代码和界面分离  
② 把常用的代码(对数据库的连接和操作)封装到工具类SqlHelper【有时也称为DAO，数据访问对象，是对数据库进行操作】

## 具体的方法
①	每一张表对应一个<font color="red">domain类</font>(表示数据)，还要对应一个<font color="red">Service类</font>（表示操作）  
比如 users 表 对应 Users 类(domain 类)，UserService类(该类会封装对users表的各种操作)

每个表对应一个domain对象和Service类，将关系模型转化为对象模型（如下图），实际上这里体现出数据和操作分离的思想  
② view负责与用户进行交互，并将数据传递给controller  
③ controller接收view中传递的数据，进行数据校验，并调用service方法，根据返回结果的不同跳转到不同的显示页面

![这里写图片描述](http://img.blog.csdn.net/20160611212436607)

## 改造后系统框架

![](http://i.imgur.com/DDiJspS.png)


* web层，structs位于web层，体现MVC的数据输入、数据处理、数据显示分离，当然web层需要调用service层中的方法完成数据处理。显示页面为MVC中的V，控制器为MVC中的C
* model层可以划分为业务层（service）、DAO层、数据持久层，这里强调一下，在一个项目中不一定全部有，可以根据实际情况选择
* hiberate（orm框架），处于数据持久层，主要解决关系模型和对象模型之间的阻抗，体现oop

***详细文档***  

https://github.com/rhapsody1290/servlet_study_usersmanager_MVC_change/blob/master/doc/%E5%88%86%E5%B1%82.xls

## 举个例子

***完成分页的mvc模式改写***  
　　首先在UsersService类中添加方法getUsersByPage,然后再  
***为什么要返回ArrayList ,而不是我们想到 ResultSet?***  
　　1. ArrayList 中封装 User对象，更加符合面向对象的编程方式  OOP     
　　2. 我们通过Resulst->User对象->ArrayList这样ArrayList和 Resultset没有关系，就可以及时关闭数据库资源

---

	//按照分页来获取用户列表
    public ArrayList<Users> getUsersByPage(int pageNow, int pageSize){
        ArrayList<Users> al = new ArrayList<Users>();

        ResultSet rs = SqlHelper.executeQuery("select * from users limit " +
                (pageNow - 1) * pageSize + "," + pageSize, null);
        try {
            while(rs.next()){
                Users user = new Users();
                user.setId(rs.getInt("id"));
                user.setUsername(rs.getString("username"));
                user.setEmail(rs.getString("email"));
                user.setGrade(rs.getInt("grade"));
                user.setPasswd(rs.getString("passwd"));
                al.add(user);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            SqlHelper.close(rs,SqlHelper.getPs(),SqlHelper.getCt());
        }
        return al;
    }

## Web-Service-DAO（数据访问层）大讨论

　　mvc规定我们应该怎样去开发软件（把数据输入，数据处理，数据显示分离）  
　　web(jsp V/Servlet C)-servie(M)-dao(M)这是一种mvc的具体实现  
　　web(jsp V/Servlet C)-service(M)开发方式也是一种具体的实现

![](http://i.imgur.com/j26engw.png)

* 举个例子，UserService.java中只包含业务逻辑，UserBean.java为数据对象，而对数据库的操作放在UserDao.java中
* 这种分层的好处是使数据和操作分离
* 将整个model层分为service层和dao层（数据访问层）
* 若UserService中业务需要对多张表进行操作，可以通过多个DAO的组合操作来实现
* 但在实际项目中，很多持久化逻辑本身就是业务逻辑（比如增加一个用户，增加用户信息到数据库时持久化逻辑，增加用户是业务逻辑），省略DAO层有时候更方便，更实用
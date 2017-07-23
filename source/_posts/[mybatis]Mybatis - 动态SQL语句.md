---
title: Mybatis - 动态SQL语句

date: 2017-03-27 13:03:00

categories:
- mybatis

tags:
- Mybatis
- 动态sql

---

## 多条件组合查询

多条件组合查询时，根据用户的输入不为空的条件，查询出符合条件的数据。比如用户可以有三个筛选条件：name，sex，password

1、如果输入了name、sex、password，则对应的sql语句是

	select * from user where name = ? and sex = ? and password = ?

2、如果输入了name，sex，对应的sql语句是

	select * from user where name = ? and sex = ?

3、如果输入了name，对应的sql语句是

	select * from user where name = ?

这时候就需要进行动态sql语句拼装

	String name = "qm";
	String sex = "男";
	String password = "123";
	StringBuilder sql = new StringBuilder("select * from user where 1 == 1");
	if(name != null && !name.trim().equals("")){
	    sql.append(" and name = ?");
	}
	if(sex != null && !sex.trim().equals("")){
	    sql.append(" and sex = ?");
	}
	if(password != null && !password.trim().equals("")){
	    sql.append(" and password = ?");
	}
	System.out.println(sql);

输出结果：

	select * from user where 1 == 1 and name = ? and sex = ? and password = ?

## 动态SQL语句

http://limingnihao.iteye.com/blog/782190

### where + if 的查询语句

如果name或sex为null，此语句很可能报错或查询结果为空。此时我们使用if动态sql语句先进行判断，如果值为null或等于空字符串，我们就不进行此条件的判断，增加灵活性。

where标签会知道如果它包含的标签中有返回值的话，它就插入一个where。此外，如果标签返回的内容是以AND 或 OR 开头的，则它会剔除掉

```
<select id="getUser" resultMap="User">  
	select * from t_user
	<where>
		<if test="name !=null ">  
	        name = #{name}
	    </if>  
	    <if test="sex != null and sex != '' ">  
	        and sex = #{sex}  
	    </if> 
	</where>
</select>  
```

### if + set 的更新语句

当update语句中没有使用if标签时，如果有一个参数为null，都会导致错误。
当在update语句中使用if标签时，如果前面的if没有执行，则或导致逗号多余错误。使用set标签可以将动态的配置 SET 关键字，和**剔除追加到条件末尾的任何不相关的逗号**。
 
使用if+set标签修改后，如果某项为null则不进行更新，而是保持数据库原值

```
<update id="updateUser" >  
    update user  
    <set>  
        <if test="name != null and name != '' ">  
            name = #{name},  
        </if>  
        <if test="sex != null and sex != '' ">  
            sex = #{sex} 
        </if>  
    </set>  
    where id = #{id};      
</update> 
```

### if + trim 代替 where/set 标签

trim是更灵活的去处多余关键字的标签，他可以实践where和set的效果。

### choose (when, otherwise)

choose标签是按顺序判断其内部when标签中的test条件出否成立，如果有**一个成立，则choose结束**。当choose中所有when的条件都不满则时，则执行otherwise中的sql。类似于Java 的switch 语句，choose为switch，when为case，otherwise则为default。

例如下面例子，同样把所有可以限制的条件都写上，方面使用。choose会从上到下选择一个when标签的test为true的sql执行。安全考虑，我们使用where将choose包起来，放置关键字多于错误。

```
<select id="getUser" resultType="User">  
    SELECT * from t_user
    <where>  
        <choose>  
            <when test="name !=null ">  
                name = #{name} 
            </when >  
            <when test="sex != null and sex != '' ">  
                and sex = #{sex}  
            </when >
            <otherwise>  
				and time >=#{data}
            </otherwise>  
        </choose>  
    </where>    
</select>  
```

## 获取自增的id的值

	<!--keyColumn：主键列的名字
		keyProperty:主键的实体对应的属性名-->
	<insert id = "adduser" useGeneratedKeys = "true" keyColumn = "id" keyProperty="id"></insert>

## 传递多个参数（JavaBean、Map）

**Map传递多个参数**，parameterType 可以是别名或完全限定名，map或者java.util.Map，这两个都是可以的，**直接在占位符中输入Map的key即可**

XML文件
```
<select id="selectBlogByMap" parameterType="map" resultType="Blog"> 
	select t.ID, t.title, t.content 
	FROM blog t 
	where t.title = #{h_title} 
	and t.content =#{h_content} 
</select> 
```

测试

```
public void testSelectByMap() { 
	SqlSession session = sqlSessionFactory.openSession(); 
	Map<String, Object> param=new HashMap<String, Object>(); 
	param.put("h_title", "oracle"); 
	param.put("h_content", "使用序列"); 
	Blog blog = (Blog)session.selectOne("cn.enjoylife.BlogMapper.selectBlogByMap",param); 
	session.close(); 
	System.out.println("blog title:"+blog.getTitle()); 
}
```

**通过JavaBean传递多个参数**，直接在占位符中输入实体的属性值

XML
```
<select id="selectBlogByBean" parameterType="Blog" resultType="Blog">
	select t.ID, t.title, t.content
	from blog t
	wheret.title = #{title}
	and t.content =#{content} 
</select>
```

测试
```
public void testSelectByBean() { 
SqlSession session = sqlSessionFactory.openSession(); 
Blog blog=new Blog(); 
blog.setTitle("oracle"); 
blog.setContent("使用序列！"); 
Blog newBlog = (Blog)session.selectOne("cn.enjoylife.BlogMapper.selectBlogByBean",blog); 
session.close(); 
System.out.println("new Blog ID:"+newBlog.getId()); 
```

## #{}的用法及传入多个参数

* `#{}` 存在mapper.xml中的sql语句部分，标识该位置可以接受参数信息，***相当于?占位符***
* 传入的参数和参数名无关，问题：如果有多个参数该怎么办？

![](http://i.imgur.com/T5wdiqe.png)

方法一：使用0，1……自然数取出对应的数据，0表示第一个参数

	select * from tb_user where user_name = #{0} and password = #{1}

方法二：使用param1，param2……param1表示第一个参数

	select * from tb_user where user_name = #{param1} and password = #{param2}

***方法三（常用）：在方法的定义上使用@param为传入的参数定义一个名字***

* 在UserMapper接口中声明函数login，使用@param为每个参数定义一个名字


	public User login(@Param("username") String username, @Param("password") String password);

* sql语句在#{}中填入定义的名字
	

	select * from tb_user where user_name = #{username} and password = #{password}

* 测试


	@Test
	public void testLogin(){
	    User user = userMapper.login("zhangsan","123456");
	    System.out.println(user);
	}

---
title: SpringMVC-Spring-Mybatis整合

date: 2016-10-07 12:52:00

categories:
- SSM

tags:
- JavaEE
- SSM

---

## 导入依赖

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
	         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	
	    <groupId>cn.apeius</groupId>
	    <artifactId>Demo</artifactId>
	    <version>1.0-SNAPSHOT</version>
	
	    <!--集中管理依赖版本号-->
	    <properties>
	        <junit.version>4.10</junit.version>
	        <spring.version>4.1.3.RELEASE</spring.version>
	        <mybatis.version>3.2.8</mybatis.version>
	        <mybatis.spring.version>1.2.2</mybatis.spring.version>
	        <mybatis.paginator.version>1.2.15</mybatis.paginator.version>
	        <mysql.version>5.1.32</mysql.version>
	        <slf4j.version>1.6.4</slf4j.version>
	        <jackson.version>2.4.2</jackson.version>
	        <druid.version>1.0.9</druid.version>
	        <httpclient.version>4.3.5</httpclient.version>
	        <jstl.version>1.2</jstl.version>
	        <servlet-api.version>2.5</servlet-api.version>
	        <jsp-api.version>2.0</jsp-api.version>
	        <joda-time.version>2.5</joda-time.version>
	        <commons-lang3.version>3.3.2</commons-lang3.version>
	        <commons-io.version>1.3.2</commons-io.version>
	    </properties>
	    <dependencies>
	
	        <!--单元测试-->
	        <dependency>
	            <groupId>junit</groupId>
	            <artifactId>junit</artifactId>
	            <version>${junit.version}</version>
	        </dependency>
	
	        <!--Spring-->
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-context</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-beans</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-webmvc</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-jdbc</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-aspects</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	
	        <!--Mybatis-->
	        <dependency>
	            <groupId>org.mybatis</groupId>
	            <artifactId>mybatis</artifactId>
	            <version>${mybatis.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.mybatis</groupId>
	            <artifactId>mybatis-spring</artifactId>
	            <version>${mybatis.spring.version}</version>
	        </dependency>
	
	        <!--分页插件-->
	        <dependency>
	            <groupId>com.github.pagehelper</groupId>
	            <artifactId>pagehelper</artifactId>
	            <version>3.7.5</version>
	        </dependency>
	        <dependency>
	            <groupId>com.github.jsqlparser</groupId>
	            <artifactId>jsqlparser</artifactId>
	            <version>0.9.1</version>
	        </dependency>
	
	        <!--连接池-->
	        <dependency>
	            <groupId>c3p0</groupId>
	            <artifactId>c3p0</artifactId>
	            <version>0.9.1.2</version>
	        </dependency>
	
	        <!--Mysql-->
	        <dependency>
	            <groupId>mysql</groupId>
	            <artifactId>mysql-connector-java</artifactId>
	            <version>${mysql.version}</version>
	        </dependency>
	
	        <!--日志-->
	        <dependency>
	            <groupId>org.slf4j</groupId>
	            <artifactId>slf4j-log4j12</artifactId>
	            <version>${slf4j.version}</version>
	        </dependency>
	
	        <!--Jackson Json处理工具包-->
	        <dependency>
	            <groupId>com.fasterxml.jackson.core</groupId>
	            <artifactId>jackson-databind</artifactId>
	            <version>${jackson.version}</version>
	        </dependency>
	
	        <!--连接池-->
	        <dependency>
	            <groupId>com.alibaba</groupId>
	            <artifactId>druid</artifactId>
	            <version>${druid.version}</version>
	        </dependency>
	
	        <!--httpclient-->
	        <dependency>
	            <groupId>org.apache.httpcomponents</groupId>
	            <artifactId>httpclient</artifactId>
	            <version>${httpclient.version}</version>
	        </dependency>
	
	        <!--JSP相关-->
	        <dependency>
	            <groupId>jstl</groupId>
	            <artifactId>jstl</artifactId>
	            <version>${jstl.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>javax.servlet</groupId>
	            <artifactId>servlet-api</artifactId>
	            <version>${servlet-api.version}</version>
	            <scope>provided</scope>
	        </dependency>
	        <dependency>
	            <groupId>javax.servlet</groupId>
	            <artifactId>jsp-api</artifactId>
	            <version>${jsp-api.version}</version>
	            <scope>provided</scope>
	        </dependency>
	
	        <!--时间操作组件-->
	        <dependency>
	            <groupId>joda-time</groupId>
	            <artifactId>joda-time</artifactId>
	            <version>${joda-time.version}</version>
	        </dependency>
	
	        <!--Apache工具组件-->
	        <dependency>
	            <groupId>org.apache.commons</groupId>
	            <artifactId>commons-lang3</artifactId>
	            <version>${commons-lang3.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.apache.commons</groupId>
	            <artifactId>commons-io</artifactId>
	            <version>${commons-io.version}</version>
	        </dependency>
	
	        <!--文件上传-->
	        <dependency>
	            <groupId>commons-fileupload</groupId>
	            <artifactId>commons-fileupload</artifactId>
	            <version>1.3.1</version>
	        </dependency>
	    </dependencies>
	</project>

## github

https://github.com/rhapsody1290/SSM

## 查询用户列表

### 定义EasyUIPage

封装对应的easyUi需要的数据

	public class EasyUIPage {
		private Long total;
		private List<?> rows;
	
		public Long getTotal() {
			return total;
		}
	
		public void setTotal(Long total) {
			this.total = total;
		}
	
		public List<?> getRows() {
			return rows;
		}
	
		public void setRows(List<?> rows) {
			this.rows = rows;
		}
	
	}

### controller

	@RequestMapping(value = "list")
	@ResponseBody
	public EasyUIPage queryAll(
			@RequestParam(value = "page", defaultValue = "1") Integer pageNum,
			@RequestParam(value = "rows", defaultValue = "5") Integer pageSize) {
		EasyUIPage esayUIPage = userService.queryAllUser(pageNum,pageSize);
		return esayUIPage;
	}

### 编写userservice

	@Override
	public EasyUIPage queryAllUser(Integer pageNum, Integer pageSize) {
		PageHelper.startPage(pageNum, pageSize);
		List<User> users = userMapper.queryAllUser();
		
		PageInfo<User> pageInfo = new PageInfo<User>(users);
		
		EasyUIPage easyUIPage = new EasyUIPage();
		//easyUIPage.setRows(users);
		easyUIPage.setRows(pageInfo.getList());
		easyUIPage.setTotal(pageInfo.getTotal());
		return easyUIPage;
	}
	
### 编写userMapper

	public List<User> queryAllUser();

### userMapper对应的xml

	<select id="queryAllUser" resultType="User">
		select * from tb_user
	</select>

## 页面跳转合并★★★★★★

	//页面跳转合并
    @RequestMapping(value = "/page/{pageName}")
    public String toPage(@PathVariable("pageName") String pageName){
        return pageName;
    }


## 添加用户

### 日期格式转换

SpringMVC默认不支持字符串转换成Date格式，可以采用SpringMVC自带的转换器，还有一种简单的方法：

	@DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birthday;

需要导入依赖：

	<!--时间操作组件-->
    <dependency>
        <groupId>joda-time</groupId>
        <artifactId>joda-time</artifactId>
        <version>${joda-time.version}</version>
    </dependency>

### 添加失败

使用try-catch

	@RequestMapping(value = "/save")
    @ResponseBody
    public Map<String, String> save(User user){
        Map<String,String> map = new HashMap<String, String>();
        try{
            Integer num = userService.addUser(user);
            if(num > 0){
                map.put("status","200");
            }else{
                map.put("status","500");
            }
        }catch (Exception e){
            map.put("status","500");
        }
        return map;
    }

## 导出excel

视图除了可以是html、jsp、json等常见视图之外，还可以是excel，其他

### poi依赖

	<!--操作excel-->
	<dependency>
	    <groupId>org.apache.poi</groupId>
	    <artifactId>poi</artifactId>
	    <version>3.10.1</version>
	</dependency>

### 导入编写好的Excel视图

UserExcelView、Constants两个Java文件

### 声明excel视图

	<bean name="userExcel" class="cn.apeius.usermanage.view.UserExcelView"/>

### 控制器

	@RequestMapping(value = "export/excel")
    public ModelAndView export(@RequestParam(value = "page", defaultValue = "1") Integer pageNow,
                               @RequestParam(value = "rows", defaultValue = "5") Integer pageSize){
        //1、查询需要导出的数据
        EasyUIPage page = this.userService.queryAllUsers(pageNow,pageSize);
        //2、传递数据
        ModelAndView mv = new ModelAndView();
        mv.addObject("userList",page.getRows());
        mv.setViewName("userExcel");//定义到自定义的excel视图
        return mv;
    }

### bean视图解析器

数字越小优先级越高

    <!--定义视图解析器-->
    <bean class = "org.springframework.web.servlet.view.BeanNameViewResolver">
        <property name="order" value="1"/>
    </bean>

## 删除用户

### 控制器

	@ResponseBody
    @RequestMapping(value = "/delete")
    //springmvc会自动把逗号切割，变成一个数组
    public Map<String,String> deleteById(@RequestParam(value = "ids") Long[] ids){
        Integer num = this.userService.deleteUserByIds(ids);
        Map<String,String> map = new HashMap<String, String>();
        if(num > 0){
            map.put("status","200");
        }else{
            map.put("status","208");
        }
        return map;
    }

### sql语句

	<delete id="deleteUserByIds">
        delete from tb_user where id in
        <foreach collection="ids" item="id" open="(" close=")" separator=",">
            #{id}
        </foreach>
    </delete>

## 通用mapper

通用的增删改查操作

### 导入依赖

	<dependency>
	    <groupId>com.github.abel533</groupId>
	    <artifactId>mapper</artifactId>
	    <version>2.3.4</version>
	</dependency>

### Mybatis配置文件方式

在mybatis-config.xml中添加如下配置:

	<plugin interceptor="com.github.abel533.mapperhelper.MapperInterceptor">
        <!--主键自增方法，默认值为MYSQL，详细说明请看文档-->
        <property name="IDENTITY" value="MYSQL"/>
        <!--通用mapper接口，多个通用接口用逗号隔开-->
        <property name="mappers" value="com.github.abel533.mapper.Mapper"/>
    </plugin>

### 通用mapper使用

#### 继承通用的Mapper<T>,必须指定泛型<T>

	public interface UserInfoMapper extends Mapper<UserInfo> {
	  //其他必须手写的接口...
	
	}

#### 泛型(实体类)<T>的类型必须符合要求★★★★★

1.	表名默认使用类名,驼峰转下划线(只对大写字母进行处理),如UserInfo默认对应的表名为user_info。
2.	表名可以使用***@Table(name = "tableName")***进行指定,对不符合第一条默认规则的可以通过这种方式指定表名.
3.	字段默认和@Column一样,都会作为表字段,表字段默认为Java对象的Field名字驼峰转下划线形式.
4.	可以使用***@Column(name = "fieldName")***指定不符合第3条规则的字段名
5.	***使用@Transient注解可以忽略字段,添加该注解的字段不会作为表字段使用（关联查询）***
6.	***建议一定是有一个@Id注解作为主键的字段,可以有多个@Id注解的字段作为联合主键.***


#### 通用mapper使用

	public class UserMapperTest {
	
	    private UserMapper mapper;
	
	    @Before
	    public void setUp() throws Exception {
	        ApplicationContext ac = new ClassPathXmlApplicationContext(
	                new String[]{"applicationContext.xml","applicationContext-mybatis.xml","applicationContext-tx.xml"});
	        mapper = ac.getBean(UserMapper.class);
	    }
	
	    @After
	    public void tearDown() throws Exception {
	
	    }
	
	    @Test
	    public void testSelectOne() throws Exception {
	        //创建User对象，设置的属性作为查询的约束
	        User user = new User();
	        user.setId(1L);
	        //使用selectOne必须保证结果唯一，如果结果太多则会报错
	        System.out.println(mapper.selectOne(user));
	    }
	
	    @Test
	    public void testSelect() throws Exception {
	        User record = new User();
	        //可以设置属性增加约束
	
	        List<User> users = mapper.select(record);
	        for(User user : users)
	            System.out.println(user);
	    }
	
	    @Test
	    public void testSelectCount() throws Exception {
	        User record = new User();
	        int count = mapper.selectCount(record);
	        System.out.println(count);
	    }
	
	    @Test
	    public void testSelectByPrimaryKey() throws Exception {
	        //参数表示主键的值
	        System.out.println(mapper.selectByPrimaryKey(1L));
	    }
	
	    @Test
	    public void testInsert() throws Exception {
	        User record = new User();
	        record.setUserName("踩雷");
	        record.setName("bajie");
	        record.setPassword("123456");
	        record.setBirthday(new Date());
	        mapper.insert(record);
	    }
	
	    @Test
	    public void testInsertSelective() throws Exception {
	        //与testInsert的区别：只添加设置值得字段，其余让数据库默认，推荐！！！
	        User record = new User();
	        record.setUserName("李璇");
	        record.setName("bajie");
	        record.setPassword("123456");
	        record.setBirthday(new Date());
	        mapper.insert(record);
	    }
	
	    @Test
	    public void testDelete() throws Exception {
	        User record = new User();
	        record.setName("xxxx");
	        mapper.delete(record);
	    }
	
	    @Test
	    public void testDeleteByPrimaryKey() throws Exception {
	        mapper.deleteByPrimaryKey(66L);
	    }
	
	    @Test
	    public void testUpdateByPrimaryKey() throws Exception {
	        User record = new User();
	        record.setId(66L);
	        record.setName("踩雷");
	        mapper.updateByPrimaryKey(record);
	    }
	
	    @Test
	    public void testUpdateByPrimaryKeySelective() throws Exception {
	        //只更新设置的值，推荐！！！！！！！
	        User record = new User();
	        record.setId(66L);
	        record.setName("踩雷");
	        mapper.updateByPrimaryKeySelective(record);
	    }
	
	
	    @Test
	    public void testExampleBasic(){
	        //进行复杂查询
	        Example example = new Example(User.class);
	        Example.Criteria criteria = example.createCriteria();
	        criteria.andGreaterThanOrEqualTo("age",30);
	        //添加and条件，密码为123456
	        criteria.andEqualTo("password","123456");
	        List<User> users = mapper.selectByExample(example);
	        for(User u : users)
	            System.out.println(u);
	    }
	
	    @Test
	    //SELECT AGE,NAME,UPDATED,USER_NAME USERNAME,BIRTHDAY,ID,CREATED,SEX,PASSWORD FROM tb_user
	    // WHERE ( AGE >= ? and NAME like ? ) or ( PASSWORD = ? and ID in(?,?) )
	    public void testExampleOr(){
	        //进行复杂查询，or
	        Example example = new Example(User.class);
	        Example.Criteria criteria = example.createCriteria();
	        criteria.andGreaterThanOrEqualTo("age",30);
	        criteria.andLike("name","%xx%");
	
	        Example.Criteria criteria12 = example.createCriteria();
	        //添加and条件，密码为123456
	        criteria12.andEqualTo("password","123456");
	        List<Object> ids = new ArrayList<Object>();
	        ids.add("1");
	        ids.add("2");
	        criteria12.andIn("id",ids);
	
	        //多个criteria之间是or的关系
	        example.or(criteria12);
	
	        List<User> users = mapper.selectByExample(example);
	        for(User u : users)
	            System.out.println(u);
	    }
	
	    @Test
	    public void testExampleSort(){
	        Example example = new Example(User.class);
	        example.setOrderByClause("age desc,id desc");
	        List<User> users = mapper.selectByExample(example);
	    }
	}

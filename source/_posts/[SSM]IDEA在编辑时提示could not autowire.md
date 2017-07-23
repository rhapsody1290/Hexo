---
title: IDEA在编辑时提示could not autowire

date: 2016-10-08 10:28:00

categories:
- SSM

tags:
- SSM

---

在开发中我在applicationContext-dao.xml中加入了mapper扫描器

	<!--mapper扫描器-->  
	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">  
	    <!--扫描包路径，如果需要扫描多个包，中间使用半角逗号隔开-->  
	    <property name="basePackage" value="com.qianlv.ssmdemo.mapper" />  
	    <!--这里不用sqlSessionFactory是因为如果用会导致上面配置的dataSource失效-->  
	    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />  
	</bean>  

但是在编辑一个Service中注入mapper会提示could not autowire，但是可以正常执行的。

	public class ItemsServiceImpl implements com.qianlv.ssmdemo.service.ItemsService{  
	  
	    @Autowired  
	    ItemsMapperCustom itemsMapperCustom;  
	  
	    public List<ItemsCustom> findItemsList(ItemsQueryVo itemsQueryVo) throws Exception {  
	        return itemsMapperCustom.findItemsList(itemsQueryVo);  
	    }  
	  
	}  

我们需要改一下IDEA的设置

![](http://i.imgur.com/lATPEQG.png)

将最右边的Serverity改为Warning

参考：http://blog.csdn.net/xlxxybz1314/article/details/51404700
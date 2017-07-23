---
title: Intellij IDEA15创建Maven管理的Java Web项目（使用IDEA启动方式，较复杂）

date: 2016-07-07 10:13:00

categories:
- Servlet

tags:
- Servlet
- IDEA

---


本文的思路是先创建Maven项目，再加上Java Web模块支持

## 创建Maven项目
***File - New - Project，创建Maven项目***

![](http://i.imgur.com/yCSg2OK.png)

***Next，填写GroupId，ArtifactId和Version***

![](http://i.imgur.com/VlBtIOA.png)

***Next，这里在Properties中添加一个参数`archetypeCatalog=internal`，不加这个参数，在maven生成骨架的时候将会非常慢，有时候会直接卡住。***

	来自网上的解释：  	
	archetypeCatalog表示插件使用的archetype元数据，不加这个参数时默认为remote，local，即中央仓库archetype元数据，由于中央仓库的archetype太多了，所以导致很慢，指定internal来表示仅使用内部元数据。

![](http://i.imgur.com/YePrI2x.png)

***Next，填写项目名和module名称，项目名和模块名可以不一样（还不是很清楚两者的区别）***

![](http://i.imgur.com/yrmOBcr.png)

***点击Finsh，项目的目录结构如下：***  

![](http://i.imgur.com/mppsHHu.png)

Maven规定，src文件下有`main`和`test`两个文件夹。其中：  

* main文件为项目主体目录，main下的java文件夹为源代码目录，resource为所需资源目录
* test为项目测试目录，test下的java文件夹为测试代码目录，resources为测试所需资源目录

***发现生成的Maven项目，没有Web目录！在项目名称右击，选择Add Framework Support***

![](http://i.imgur.com/p5NAWmy.png)

***在Add Framework Support对话框中勾选Web Application，版本选择3.0并勾选Create web.xml***

![](http://img.my.csdn.net/uploads/201303/14/1363248954_3896.png)

***点击OK后，看到如下界面，项目中出现了web文件夹，是不是很熟悉了，和MyEclipse中的项目结构类似***

![](http://i.imgur.com/L5LXM7A.png)

## 配置Tomcat服务器

***点击右上角的倒三角，选择`Edit Configurations`，弹出服务器配置页面***

![](http://i.imgur.com/77V6dHo.png)

***如下图，选择Local，然后点击Configure，在弹出的对话框中选择Tomcat安装目录***

![](http://img.my.csdn.net/uploads/201303/14/1363248975_9849.png)

***选择Tomcat Server，然后点击绿色的“+”号***

![](http://img.my.csdn.net/uploads/201303/14/1363248979_4524.png)

***点击“+”后选择Local，刚刚已经配置好了Local的Tomcat服务器***

![](http://img.my.csdn.net/uploads/201303/14/1363248982_3705.png)

***这里会新建一个Tomcat服务，输入任意名字即可。Update的快捷键是Ctrl+F10***

![](http://i.imgur.com/zVA6RYW.png)

***点击Deployment，然后点击右边的“+”，添加Artifact部署***

![](http://img.my.csdn.net/uploads/201303/14/1363248993_1308.png)

***输入应用程序Context，输入路径:`/工程名`***

![](http://i.imgur.com/xsZHFqP.png)

***点击界面上方的启动按钮就可以启动Tomcat服务器，启动后服务器自动打开浏览器***

![](http://i.imgur.com/fmJjfqg.png)

***回到主界面，如图，点击Run打开服务器视图，能看到项目的部署情况了，而且可以完成服务器重启/关闭，项目部署等操作***

![](http://i.imgur.com/LMFpfTI.png)



---
title: XML CDATA的作用

date: 2017-02-13 09:54:00

categories:
- SSM

tags:
- XML
- CDATA

---

http://www.cnblogs.com/myparamita/archive/2008/12/27/1363626.html

XML文件中的一些**特殊符号**，例如："<"、">"、"/"、"？"等，会破坏了XML结构

有两种方式解决：
* 转义字符
* CDATA部件

## 转义字符

如果在XML文档中使用类似"<"的字符, 那么解析器将会出现错误，因为解析器会认为这是一个新元素的开始。所以不应该象下面那样书写代码:

	<message>if salary < 1000 then</message> 

为了避免出现这种情况，必须将字符"<"转换成实体，象下面这样:

	<message>if salary &lt; 1000 then</message> 

下面是五个在XML文档中预定义好的实体:

	&lt; < 小于号 
	&gt; > 大于号 
	&amp; & 和 
	&apos; ' 单引号 
	&quot; " 双引号 

实体必须以符号"&"开头，以符号";"结尾。 

注意: 只有"<" 字符和"&"字符对于XML来说是严格禁止使用的。剩下的都是合法的，为了减少出错，使用实体是一个好习惯。

## CDATA

在CDATA内部的所有内容都会被解析器忽略。

如果文本包含了很多的"<"字符和"&"字符——就象程序代码一样，那么最好把他们都放到CDATA部件中。

一个 CDATA 部件以"<![CDATA["标记开始，以"]]>"标记结束:

	<script>
		<![CDATA[
			function matchwo(a,b)
			{
				if (a < b && a < 0) then
				{
					return 1
				}
				else
				{
					return 0
				}
			}
		]]>
	</script> 

在前面的例子中，所有在CDATA部件之间的文本都会被解析器忽略。

CDATA注意事项:
CDATA部件之间不能再包含CDATA部件（不能嵌套）。如果CDATA部件包含了字符"]]>" 或者"<![CDATA[" ，将很有可能出错哦。

同样要注意在字符串"]]>"之间没有空格或者换行符。
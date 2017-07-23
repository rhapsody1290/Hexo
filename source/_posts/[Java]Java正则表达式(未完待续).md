---
title: Java正则表达式(未完待续)

date: 2017-03-01 15:00:00

categories:
- Java

tags:
- 正则表达式

---

1、"."、"|"、"*"都是转义字符，**必须得加"`\\`"**
	
	如果用"."作为分隔的话，必须是如下写法：String.split("\\.")，这样才能正确的分隔开，不能用String.split(".")；
	如果用"|"作为分隔的话，必须是如下写法：String.split("\\|")，这样才能正确的分隔开，不能用String.split("|");

2、如果在一个字符串中有多个分隔符，可以用"|"作为连字符

	比如："acount=? and uu =? or n="，把三个都分隔出来，可以用String.split("and|or");

3、public String[] split(String regex，int limit)根据匹配给定的正则表达式来拆分此字符串。

此方法返回的数组包含此字符串的每个**子字符串**，这些子字符串由**另一个匹配给定的表达式的子字符串**终止或由**字符串结束符**来终止。数组中的子字符串按它们在此字符串中的顺序排列。如果表达式不匹配输入的任何部分，则结果数组只具有一个元素，即**此字符串**。

4、public string[] split(string regex)

这里的参数的名称是 regex ，也就是 regular expression （正则表达式）。这个参数并不是一个简单的分割用的字符，而是一个正则表达式，
他对一些特殊的字符可能会出现你预想不到的结果，比如测试下面的代码： 

## 正则表达式案例

多关键词
```
String s="utf-8#{gb2312}utf-8#{iso8859-1}gbk#{zzz}";//['utf-8','gbk','gb2312','iso8859-1']
Pattern pattern = Pattern.compile("utf-8|gbk|gb2312|iso8859-1");
Matcher m = pattern.matcher(s);
while(m.find()){
    System.out.println(m.group());
}
```


	
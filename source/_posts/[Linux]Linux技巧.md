---
title: Linux技巧

date: 2017-10-10 16:06:00

categories:
- Linux

tags:
- Linux

---

## crontab

查看当前用户的定时任务
crontab -l
crontab -e

案例：

	00 10 * * * sh /disk2/qianming.qm/
	calculate_daily_diff_predict_hot_poi_and_sendmail.sh >/disk2/qianming.qm/
	calculate_daily_diff_predict_hot_poi_and_sendmail.log 2>&1

参考：
	
	http://blog.csdn.net/liu414226580/article/details/16339935

## 查看日志

	tail -f xxx.log

## 输出重定向

0 表示stdin标准输入
1 表示stdout标准输出
2 表示stderr标准错误
command>a 这条命令，等价于command 1>a
即将命令标准的输出重定向到a文件

对于2>&1也就好理解了，2就是标准错误，1是标准输出，那么这条命令不就是相当于把标准错误重定向到标准输出

参考：
http://blog.csdn.net/ggxiaobai/article/details/53507530

## 命令后台运行

	nohup sh s.sh > output 2>&1 &

参考https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/ 

## 查看文件

带序号
less -N top300w_user_xy_report_201708_toqianming_sorted.txt
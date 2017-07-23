---
title: Linux视频教程第12讲.网络环境配置

date: 2017-04-12 23:24:00

categories:
- Linux

tags:
- Linux

---

##第一种方法

用root身份登录，运行setup命令进入到text mode setup utility对网络进行配置-network configuration，这里可以进行IP、子网掩码、默认网关、DNS的配置

用空格键自动分配 手动IP 按TAB就可以输入

这时网卡的配置没有生效，运行/etc/rc.d/init.d/network restart命令我们刚才做的设置才生效

##第二种方法

eth0 第一块网卡，eth1 第二块网卡.....

```
ifconfig eth0 x.x.x.x对网卡进行设置 
ifconfig eth0 network x.x.x.x对子网掩码设置 对广播地址和DNS使用默认的
```

**Note：这样配置网络将会立即生效，但是是临时生效**

##第三种方法

修改`/etc/sysconfig/network-scripts/ifcfg-eth0`这个文件里各个属性可以修改，包括IP、子网掩码、广播地址、默认网关等。 

里面的内容主要如下：

<pre>
onboot=yes (NO=禁用)
bootproto=static(静态)/dhcp(动态)
</pre>

这时网卡的配置没有生效，运行

	/etc/rc.d/init.d/network restart

命令我们刚才做的设置才生效

Note： 这种方法是最底层的修改方法在linux中，**所有设备都是文件**





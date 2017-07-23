---
title: Kafka分布式消息系统

date: 2017-06-22 17:39:00

categories:
- 分布式架构

tags:
- Kafka

---

Kafka与Zoopkeeper关系

ZooKeeper用于管理、协调Kafka代理。当Kafka系统中新增了代理或者某个代理故障失效时，ZooKeeper服务将通知生产者和消费者，生产者和消费者据此开始与其它代理协调工作

![](http://i.imgur.com/2tJ9DsD.png)

## 参考

http://www.infoq.com/cn/articles/apache-kafka
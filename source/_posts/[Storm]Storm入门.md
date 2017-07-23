---
title: Storm入门

date: 2017-04-20 17:34:00

categories:
- Storm

tags:
- Storm

---

http://ifeve.com/getting-started-with-stom-index/

## 专业术语

spout：管理Storm集群的输入流，为bolt提供数据
bolt：从spout或其它bolt接收数据，并处理数据，处理结果可作为其它bolt的数据源或最终结果
nimbus：雨云，主节点的守护进程，负责为工作节点分发任务
topology：拓扑结构，Storm的一个任务单元
define field(s)：定义域，由spout或bolt提供，被bolt接收

## Storm 概述

Storm是一个分布式的，可靠的，容错的数据流处理系统。它会把工作任务委托给不同类型的组件，每个组件负责处理一项简单特定的任务。**Storm集群的输入流由一个被称作spout的组件管理，spout把数据传递给bolt， bolt要么把数据保存到某种存储器，要么把数据传递给其它的bolt。**你可以想象一下，一个Storm集群就是在一连串的bolt之间转换spout传过来的数据。

## Storm 结构

在Storm集群中，有两类节点：**主节点master node**和**工作节点worker nodes**
* 主节点运行着一个叫做Nimbus的守护进程。这个守护进程负责在集群中分发代码，为工作节点分配任务，并监控故障
* Supervisor守护进程作为拓扑的一部分运行在工作节点上。一个Storm拓扑结构在不同的机器上运行着众多的工作节点

## Storm 操作模式

* 本地模式：Storm拓扑结构运行在本地计算机的单一JVM进程上
* 远程模式：在远程模式下，我们向Storm集群提交拓扑，它通常由许多运行在不同机器上的流程组成

## Storm 快速入门

教程在github中：

https://github.com/runfriends/GettingStartedWithStorm-cn/blob/master/chapter2/Hello%20World%20Storm.md#bolts

创建一个简单的拓扑，数单词数量。要创建这个拓扑，我们要用一个spout读取文本，第一个bolt用来标准化单词，第二个bolt为单词计数，流程如下图所示：

![](http://i.imgur.com/Nsx7tMR.png)

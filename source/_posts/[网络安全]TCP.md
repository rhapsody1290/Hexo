---
title: TCP

date: 2016-12-06 19:36:00

categories:
- 网络安全

tags:
- TCP

---

TCP是在不可靠的端到端网络协议（IP）之上实现的可靠数据传输协议

## 可靠数据传输的原理

### 完全可靠信道上的可靠数据传输：rdt1.0

* rdt发送方只有一个状态，通过rdt_send(data)从较高层接收数据，产生分组，并将分组发送到信道中后，状态又跳回到等待上传调用的状态
* rdt接收方只有一个状态，通过rdt_rcv（packet）从底层信道接收一个分组，提取数据，并将数据上传给高层，状态又回到等待下层调用

因为信道完全可靠，接收方不需要提供任何反馈信息给发送方

### 具有比特差错信道上的可靠数据传输：rdt2.0（不丢包，但包可能受损）

人类处理这类情形：报文接受者在听到每句话后会说OK，如果消息接受者听到一句含糊不清的话，他可能要求你重复刚才那句话。这种口述消息协议使用了**肯定确认**（OK）与**否定确认**（请重复一遍）

在计算机网络中，基于这种重传机制的可靠数据传输协议称为**自动重传请求**（ARQ），需要三种协议来处理比特差错：

* 差错检测
* 接收方反馈：肯定确认（ACK），否定确认（NAK）
* 重传

rdt2.0发送方两个状态：

1、等待上层调用：rdt_send事件发生，产生数据包，发送该分组，状态跳到2
2、等待接收方的ACK或NAK

* 如果收到ACK分组，则发送发知道最近传输的分组已经被正确接收，因此状态返回**等待上层调用**
* 如果收到一个NAK分组，重传最后一个分组，并**等待接受方的ACK或NAK**

rdt2.0接收方的只有一个状态：当分组到达时，接收方要么回答一个ACK，要么回答一个NAK

**问题：**

1、停等协议：只有当上一个分组发送成功，才能发送下一个分组，效率低
2、没有考虑ACK或NAK分组受损<font color='red'>**（假设分组不丢失）**</font>，发送方无法知道接收方是否正确接收了上一块发送的数据

**rdt2.1：引入序列号解决冗余分组**

解决ACK或NAK受损的一种方式是：当发送方收到含糊不清的ACK或NAK分组时，直接重发当前数据分组，但这引入了**冗余分组**，接收方不知道上次发送的ACK或NAK是否被发送方正确接收到，导致接收方无法知道接收到的分组是**新的**，还是一次**重传**（重传的数据包保证该数据包中的数据只提取一次）

解决这个问题的一个简单方法是在数据分组中 **添加一个新字段**，对于停等协议，1比特就够了，如果接收到的分组序号与最近收到的分组序号**相同**，知道是在**重发**，**不相同**则是一个**新分组**

发送方状态：
当发送完**分组0**，处于等待ACK或NAK的状态，如果**分组受损**或收到**NAK**，重发分组；如果收到ACK，说明接收方正确接收到分组0，状态跳转到等待发送**分组1**的状态

接收方状态：如果接收到受损包，直接发送NAK；如果收到分组的序号与上一次收到的序号一致，说明是重传包，直接发送ACK；如果收到的分组序号与上一次收到的不一致，发送NAk。具体的，划分出**两个状态**：**等待数据0**和**等待数据包1**

**rdt2.2：冗余ACK来代替NAK**

rdt2.1中接收到受损分组时发送一个NAK，通知发送端重发。如果不发送NAK，而是发送一个对上次正确接收的分组的ACK（即**冗余ACK**），发送方接收到对同一个分组的两个ACK，就知道接受方没有正确接收到跟在被确认两次的分组后面的分组

发送方状态：
在等待ACK0时，若收到损坏的分组或ACK1，则重传，否则状态跳到等待上层调用1

接收方状态：
如果接收到受损包，或收到分组的序号与上一次收到的序号一致，说明是重传包，直接发送以上一次的发送的ACK（冗余ACK）；如果收到的分组序号和上一次收到的不一致，则提取数据，发送ACk

### 具有比特差错的丢包信道上的可靠数据传输：rdt3.0

需要考虑两个问题：

* 如何检测丢包
* 发生丢包后该做什么

利用校验和、序号、ACK分组和重传可以解决后一个问题，即**重传**，所以重点是解决如何检测丢包问题

在发送方负责检测和恢复丢包，增加一个**倒计时定时器**，发送一个分组后（包括重传），便启动一个定时器

发送方状态：上层调用rdt_send(data)时，发送数据包，并**开启定时器**，状态转移到等待ACK的状态。**如果接收到破损包或者上一个分组包的ACK，先不采取任何操作，等到定时器超时后进行重传，这种方式也考虑到包丢失的情况**

接收方状态：如果接收到受损包，或收到分组的序号与上一次收到的序号一致，说明是重传包，直接发送以上一次的发送的ACK（冗余ACK）；如果收到的分组序号和上一次收到的不一致，则提取数据，发送ACk（**与rdt2.2一样**）

### 流水线可靠数据传输协议

停等协议的是一个功能正确的协议，但是它的性能低下，吞吐量低，是网络协议限制底层网络硬件所提供功能的形象示例

解决性能问题的一个简单方法是**采用流水线技术**，允许发送方发送多个分组而无须等待确认，解决流水线的差错恢复有两种基本方法：

* 回退N步
* 选择重传

#### 回退N步（GBN）

将**基序号（base）**定义为最早的未被确认分组的序号，将**下一个序号（nextseqnum）**定义为下一个待发送的分组序号

![](http://i.imgur.com/aVtCxCl.png)

序号分为4部分：

* [0，base-1]内的序号对应于已经发送并确认过的序号
* [base，nextseqnum-1]内的序号对应已经发送但未被确认的分组
* [nextseqnum，base+N-1]内的序号可用于那些要被立刻发送的分组，其数据来自上层
* 大于或等于base+N的序号是不能使用的，知道当前流水线中未被确认的分组已得到确认为止

GBN发送方响应的三种类型事件：

* 上层的调用：如果发送窗口未满，则创建一个分组并发送，否则不传送
* 收到ACK：采用累积确认，表示接收方已正确接收序号n之前的所有分组
* 超时时间：GBN协议中，使用的一个定时器当做是**最早的已发送但未被确认的分组**所使用过的定时器（即base分组），如果超时将重传所有已发送但未被确认的分组（即图中灰色的分组）；如果收到一个ACK，但仍有未被确认的分组，则定时器被重新启动，如果没有已发送但未被确认的分组，则定时器终止

接收方：**丢弃所有失序分组**。如果期望n分组，而n+1分组却到了，不需要缓存n+1分组，因为如果n分组丢失，发送方将会重新发送n分组和n+1分组，这样设计，接收方**不需要缓存任何失序分组**，接收方只需要维护**下一个按需接收的分组的序号**。

#### 选择重传



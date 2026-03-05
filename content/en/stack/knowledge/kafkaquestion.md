---
author: "jarvan"
title: "Kafka的重复消费和消息丢失问题"
date: 2022-05-17
description: "Kafka的重复消费和消息丢失问题"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- knowledge
categories:
- kafka
---

## 介绍

在Kafka中，**生产者（Producer）** 和 **消费者（Consumer）** 是通过 发布订阅模式 进行协作的，生产者将消息发送到Kafka集群，而消费者从Kafka集群中拉取消息进行消费，
无论是生产者发送消息到Kafka集群还是消费者从Kafka集群中拉取消息进行消费，都是容易出现问题的，比较典型的就是 
<font color='cyan'>**消费端的重复消费问题**</font> 、 <font color='cyan'>**生产端和消费端产生的消息丢失**</font> 问题 。
下面将对这两个问题出现的场景以及常见的解决方案进行讲解。
## 重复消费

### 1 重复消费出现的场景
重复消费出现的常见场景主要分为两种：
1. Consumer在消费过程中，应用进程被强制kill掉或者发生异常退出（挂掉…）
2. Consumer消费的时间过长

#### 1.1 消费者消费过程中，进程挂掉/异常退出
在Kafka消费端的使用中，位移（Offset）的提交有两种方式，自动提交 和 手动提交：
<font color='cyan'>**自动提交情况下**</font> ，当消费者拉取一批消息进行消费后，需要进行Offset的提交，在消费端提交Offset之前，Consumer挂掉了，当Consumer重启后再次拉取Offset，这时候拉取的依然是挂掉之前消费的Offset，因此造成 **重复消费** 的问题。
<font color='cyan'>**手动提交模式下**</font> ，在提交代码调用之前，Consumer挂掉也会造成重复消费。

#### 1.2 消费者消费时间过长
Kafka消费端的参数 <font color='cyan'>**max.poll.interval.ms**</font> 定义了两次poll的最大间隔，它的默认值是 5 分钟，表示 Consumer 如果在 5 分钟之内无法消费完 poll方法返回的消息，那么Consumer 会主动发起“离开组”的请求

在 <font color='cyan'>**离开消费组后**</font> ，开始Rebalance，因此 <font color='cyan'>**提交Offset失败**</font> 。之后 <font color='cyan'>**重新Rebalances**</font>，消费者再次分配Partition后，再次poll拉取消息依然从之前消费过的消息处开始消费，这样就造成重复消费。而且若不解决消费单次消费时间过长的问题，这部分消息可能会一直重复消费。

整体上来说，如果我们在消费中将消息数据处理入库，但是在 执行Offset提交时，Kafka宕机或者网络原因等 <font color='cyan'>**无法提交Offset**</font> ，当我们重启服务或者Rebalance过程触发，Consumer将再次消费此消息数据。
>PS :1.一开始以为poll()方法里传的是Kafka返回的记录条数，
但其实是传的时间（ms）
> 2.Kafka轮询一次就相当于拉取（poll）一定时间段broker中可消费的数据，
在这个指定时间段里拉取，时间到了就立刻返回数据。
> 3.例如poll（5000）：
即在5s中内拉去的数据返回到消费者端。

### 2 重复消费解决方案

#### 2.1 针对于消费端挂掉等原因造成的重复消费问题
这部分主要集中在消费端的 <font color='cyan'>**编码层面**</font> ，需要我们在设计代码时 <font color='cyan'>**以幂等性的角度进行开发设计**</font> ，保证同一数据无论进行多少次消费，所造成的结果都一样。
处理方式可以在消息体中 <font color='cyan'>**添加唯一标识**</font>(比如将消息生成md5保存到Mysql或者是Redis中，在处理消息前先检查下Mysql/Redis是否已经处理过该消息了)，消费端进行确认此唯一标识是否已经消费过，如果消费过，则不进行之后处理。从而尽可能的避免了重复消费。

幂等性角度大概两种实现：

1. <font color='cyan'>**解决方法一：**</font>将唯一标识存入第三方介质（如Redis）
  要操作数据的时候先判断第三方介质(数据库或者缓存)有没有这个唯一标识。
2. <font color='cyan'>**解决方法二：**</font>将版本号(offset)存入到数据里面
  然后再要操作数据的时候用这个版本号做 <font color='cyan'>**乐观锁**</font> ，当 <font color='cyan'>**版本号大于**</font> 原先的才能操作。

>PS:乐观锁的实现：
> 数据库表设计时加入的"版本号"字段
> CREATE TABLE items (
>   id INT PRIMARY KEY AUTO_INCREMENT,
>   value VARCHAR(255),
>   version INT DEFAULT 0
> );
> 更新操作： `UPDATE items SET value = 'new value', version = version + 1 WHERE id = 1 AND version = 0;`

#### 2.2 针对于Consumer消费时间过长带来的重复消费问题
1. <font color='cyan'>**解决方法一：**</font>提高单条消息的处理速度。
例如对消息处理中比较耗时的操作可通过 异步的方式 <font color='cyan'>**进行处理**</font>、<font color='cyan'>**多线程处理**</font>  等。
2. <font color='cyan'>**解决方法二：**</font>将 `max.poll.interval.ms` 值设置大一点
避免不必要的rebalance，此外可适当减小 `max.poll.records` 的值，默认值是500，可根据实际消息速率适当调小。

## 消息丢失
在Kafka中，消息丢失在Kafka的 **生产端** 和 **消费端** 都会出现。在此之前我们先来了解一下生产者和消费者的原理。

### 1 生产端问题
#### 生产者原理：
Kafka生产者生产消息后，会将消息发送到Kafka集群的Leader中，然后Kafka集群的Leader收到消息后会返回ACK确认消息给生产者Producer。
主要拆解为以下几个步骤：
1. Producer先从Kafka集群找到该Partition的Leader
2. Producer将消息发送给Leader，Leader将该消息写入本地
3. Follwer从Leader pull消息，写入本地Log后Leader发送ACK
4. Leader 收到所有 ISR（In-Sync Replicas，同步副本集） 中的 Replica 的 ACK 后，增加High Watermark，并向 Producer 发送 ACK

![/images/docImages/kfk1.png](/images/docImages/kfk1.png)

因此，Kafka集群（其实是分区的Leader）最终会返回一个ACK来确认Producer推送消息的结果，这里
Kafka提供了 <font color='cyan'>**发送消息三种模式：**</font>
1. <font color='cyan'>**NoResponse RequiredAcks = 0**</font>：
这个代表的就是不进行消息推送是否成功的确认。
2. <font color='cyan'>**WaitForLocal RequiredAcks = 1**</font>：
当local(Leader)确认接收成功后，就可以返回了。
3. <font color='cyan'>**WaitForAll RequiredAcks = -1**</font>：
当所有的Leader和Follower都接收成功时，才会返回。

因此这个配置的影响也分为下面三种情况：
1. 设置为 0
Producer不进行消息发送的确认，Kafka集群（Broker）可能由于一些原因并没有收到对应消息，从而引起消息丢失。
2. 设置为 1
Producer在确认到 Topic Leader 已经接收到消息后，完成发送，此时有可能 Follower 并没有接收到对应消息。此时如果 Leader 突然宕机，在经过选举之后，没有接到消息的 Follower 晋升为 Leader，从而引起消息丢失。
3. 设置为 -1
可以很好的确认Kafka集群是否已经完成消息的接收和本地化存储，并且可以在Producer发送失败时进行重试。

#### 生产端解决消息丢失方案：
1. <font color='cyan'>**解决方法一：**</font> 通过设置 <font color='cyan'>**RequiredAcks模式**</font> 来解决，选用WaitForAll（对应值为-1）可以保证数据推送成功，不过会影响延时。
2. <font color='cyan'>**解决方法二：**</font> 引入 <font color='cyan'>**重试机制**</font> ，设置重试次数和重试间隔。
3. <font color='cyan'>**解决方法三：**</font> 使用Kafka的 <font color='cyan'>**多副本机制**</font> 保证Kafka集群本身的可靠性，确保当Leader挂掉之后能进行Follower选举晋升为新的Leader。



### 2 消费端问题
消费端的消息丢失问题：
消费端的消息丢失主要是因为在 <font color='cyan'>**消费过程中出现了异常**</font> ，但是对应消息的 <font color='cyan'>**Offset 已经提交**</font>，那么消费异常的消息将会丢失。
前面介绍过，Offset的提交包括 手动提交 和 自动提交 ，可通过kafka.consumer.enable-auto-commit进行配置。
+ 手动提交 
  可以灵活的确认是否将本次消费数据的Offset进行提交，可以很好的避免消息丢失的情况。
+ 自动提交
  是引起消息丢失的主要诱因。因为消息的消费并不会影响到Offset的提交。


----
><font color='cyan'>**解决方法：**</font> 尽量使用手动提交的方式，或者用sarama自动提交方式（先进行标记，标记前处理数据，再进行提交，因此没有标记，offset没有提交）
----
大部分的解决方案为了 尽可能的保证数据的完整性，都是尽量去选用 <font color='cyan'>**手动提交**</font> 的方式，当数据处理完之后再进行提交。
当然，在golang中我们主要使用sarama包的Kafka，sarama自动提交的原理是先进行标记，再进行提交，如下代码所示：
```go
type exampleConsumerGroupHandler struct{}

func (exampleConsumerGroupHandler) Setup(_ ConsumerGroupSession) error   { return nil }
func (exampleConsumerGroupHandler) Cleanup(_ ConsumerGroupSession) error { return nil }
func (h exampleConsumerGroupHandler) ConsumeClaim(sess ConsumerGroupSession, claim ConsumerGroupClaim) error {
   for msg := range claim.Messages() {
      fmt.Printf("Message topic:%q partition:%d offset:%d
", msg.Topic, msg.Partition, msg.Offset)
      // 标记消息已处理，sarama会自动提交
      // 处理数据（如真正持久化mysql...）
      sess.MarkMessage(msg, "")
   }
   return nil
}
```
因此，我们完全可以在标记之前进行数据的处理，例如插入Mysql等，当出现插入成功后程序崩溃，下一次最多重复消费一次（因为还没标记，Offset没有提交），而不会因为Offset超前，导致应用层消息丢失了。

手动提交模式下当然是很灵活的控制的，但确实已经没必要了：
```go
consumerConfig := sarama.NewConfig()
consumerConfig.Version = sarama.V2_8_0_0
consumerConfig.Consumer.Return.Errors = false
consumerConfig.Consumer.Offsets.AutoCommit.Enable = false  // 禁用自动提交，改为手动
consumerConfig.Consumer.Offsets.Initial = sarama.OffsetNewest
 
func (h msgConsumerGroup) ConsumeClaim(sess sarama.ConsumerGroupSession, claim sarama.ConsumerGroupClaim) error {
   for msg := range claim.Messages() {
      fmt.Printf("%s Message topic:%q partition:%d offset:%d  value:%s
", h.name, msg.Topic, msg.Partition, msg.Offset, string(msg.Value))
 
      // 插入mysql....
      // 手动提交模式下，也需要先进行标记
      sess.MarkMessage(msg, "")
      consumerCount++
      if consumerCount%3 == 0 {
         // 手动提交，不能频繁调用
         t1 := time.Now().Nanosecond()
         sess.Commit()
         t2 := time.Now().Nanosecond()
         fmt.Println("commit cost:", (t2-t1)/(1000*1000), "ms")
      }
   }
   return nil
}
```


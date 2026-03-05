---
author: "Karson"
title: "golang中使用kafka的综合指南"
date: 2022-05-17
description: "golang中使用kafka的综合指南"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- knowledge
categories:
- kafka

---

## 背景

kafka 是一个比较流行的 <font color='cyan'>**分布式 、 可拓展 、 高性能 、 可靠性**</font> 的流处理平台。
在处理kafka的数据时，这里有 确保处理效率和可靠性的 多种最佳实践。
下面介绍这几种实践方式，并通过sarama实现他们。

## 需要注意的点(最佳实践方案)

### 1.选择合适的提交策略
Kafka提供两种提交策略，<font color='cyan'>**自动**</font> 和 <font color='cyan'>**手动**</font>。
+ 虽然自动操作很容易使用，    但它可能会导致 <font color='cyan'>**数据丢失或重复**</font> 。
+ 手动提交提供了更高级别的控制，确保消息至少处理一次或恰好一次，具体取决于用例。


#### a、自动提交
Sarama 的 ConsumerGroup 默认情况下会自动提交偏移量。这意味着它会定期提交已成功消费的消息的偏移量，这允许消费者在重新启动或消费失败时从中断的地方继续。
下面是一个自动提交的消费者组消费消息的例子：
```go
    // 自动提交偏移量
    config := consumerGroup.Config()
    config.Consumer.Offsets.AutoCommit.Enable = true
    config.Consumer.Offsets.AutoCommit.Interval = 1 * time.Second
```
根据 `config.Consumer.Offsets.AutoCommit.Interval` 可以看到，消费者会每秒自动提交offset。

#### b、手动提交
手动提交使我们更好地控制何时提交消息偏移量。下面是一个手动提交的消费者组消费消息的例子：
```go
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
### 2.尽量减少Kafka的传输次数
<font color='cyan'>**大批量读取消息**</font> 可以显著 <font color='cyan'>**提高吞吐量**</font>。
这可以通过调整 `fetch.min.bytes` 和 `fetch.max.wait.ms` 等参数来实现。

减少kafka的传输次数可以通过 优化从kafka中 <font color='cyan'>**读取和写入**</font> 数据的方式来实现：
#### a、增加批次的大小
使用kafka批量发送消息的效果优于逐个发送消息，批次越大，kafka发送数据效率就越高。但是需要权衡延迟和吞吐量之间的关系。较大的批次虽然代表着更大的吞吐量，但也会增加延迟。
<font color='cyan'>**因为批次越大，填充批次的时间也越久**</font>

在Go中，我们可以在使用sarama包生成消息时设置批次大小：
可以通过kafka.Config结构体的 `Producer.Return.Successes` 配置项来设置批次大小.个配置项决定了当一个消息成功被Kafka接收时，是否会被客户端返回。如果设置为true，那么成功投递的消息会被批量返回。
```go
    config := sarama.NewConfig()
    config.Producer.Return.Successes = true
    config.Producer.BatchSize = 16384 // 设置批次大小为16KB
    config.Producer.Flush.Messages = 100 // 当批次消息数达到100时刷新到磁盘
```

#### b、使用长轮询
长轮询是指消费者轮询时如果Kafka中没有数据，则消费者将等待数据到达。这减少了往返次数，因为消费者不需要在没有数据时不断请求数据。
```go
    // 设置最大轮询间隔时间为10秒
    config.Consumer.MaxProcessingTime = 10 * time.Second
```

### 3.尽量使用消费者组
Kafka允许 <font color='cyan'>**多个消费者**</font> 组成一个 <font color='cyan'>**消费者组并行消费数据**</font> 。这使得 Kafka 能够将数据分发给一个组中的所有消费者，从而实现高效的数据消费。

消费者组是一组协同工作消费来自kafka主题的消息的消费者。消费者组允许我们在多个消费者之间分配消息，从而提供横向拓展能力。使用消费者组时，kafka负责将分区分配给组中的消费者，并确保 <font color='cyan'>**每个分区同时仅被一个消费者消费**</font>。
接下来是sarama中消费者组的使用：
#### a.使用消费者组需要实现一个ConsumerGroupHandler接口：
该接口具有三个方法：Setup、Cleanup、 和ConsumeClaim
```go
func (consumerGroupHandler) Setup(_ sarama.ConsumerGroupSession) error   { return nil }
func (consumerGroupHandler) Cleanup(_ sarama.ConsumerGroupSession) error { return nil }
func (h consumerGroupHandler) ConsumeClaim(sess sarama.ConsumerGroupSession, claim sarama.ConsumerGroupClaim) error {
	for msg := range claim.Messages() {
		fmt.Printf("%s Message topic:%q partition:%d offset:%d  value:%s\n",h.name, msg.Topic, msg.Partition, msg.Offset, string(msg.Value))
		// 手动确认消息
		sess.MarkMessage(msg, "")
	}
	return nil
}
```



### 4.调整消费者缓冲区大小
通过调整 <font color='cyan'>**消费者的缓冲区大小**</font> ，如 `receive.buffer.bytes` 和 `max.partition.fetch.bytes` ，可以根据消息的 预期大小 和 消费者的内存容量 进行调整。这可以提高消费者的表现。

在sarama中，我们可以调整消费者缓冲区的大小，以调整消费者在处理消息之前可以在内存中保存的消息数量。
默认情况下，缓冲区大小设置为256，这代表Sarama在开始处理消息之前将在内存中保存最多256条消息。<font color='cyan'>**如果消费者速度很慢**</font>，增加缓冲区大小可能有助于提高吞吐量。但是，更大的缓冲区也会消耗更多的内存。


### 5.处理rebalance
当 新的消费者加入消费者组，或者现有的消费者离开 时，Kafka会 <font color='cyan'>**触发`rebalance`以重新分配负载**</font> 。在此过程中， <font color='cyan'>**消费者停止消费数据**</font> 。因此，快速有效地处理重新平衡可以提高整体吞吐量。

当新消费者添加到消费者组或现有消费者离开消费者组时，kafka会重新平衡该组中的消费者。rebalance是kafka确保消费者组中的所有消费者不会消费同一分区的保证。

在sarama中，处理rebalance是通过 <font color='cyan'>**Setup**</font> 和<font color='cyan'>**CleanUp**</font> 函数来完成的。

通过正确处理重新平衡事件，您可以确保应用程序正常处理消费者组的更改，例如消费者离开或加入，并且在这些事件期间不会丢失或处理两次消息。

### 6.监控消费者
使用 Kafka 的消费者指标来 <font color='cyan'>**监控消费者的性能**</font>  。 <font color='cyan'>**定期监控**</font> 可以帮助我们识别性能瓶颈并调整消费者的配置。

监控Kafka消费者对于确保系统的健康和性能至关重要，我们需要时刻关注延迟、处理时间和错误率的指标。

---
>PS:Golang没有内置对 Kafka 监控的支持，但有几个库和工具可以帮助我们。让我们看一下其中的一些：
> 1 . Sarama的Metrics：Sarama 提供了一个指标注册表，它报告了有助于监控的各种指标，例如请求、响应的数量、请求和响应的大小等。这些指标可以使用 Prometheus 等监控系统来收集和监控。
> 2 . JMX Exporter：如果您在 JVM 上运行 Kafka， 则可以使用 JMX Exporter 将kafka的 MBeans 发送给Prometheus
> 3 . Kafka Exporter：Kafka Exporter是一个第三方工具，可以提供有关Kafka的更详细的指标。它可以提供消费者组延迟，这是消费kafka消息时要监控的关键指标。
> 4 . Jaeger 或 OpenTelemetry：这些工具可用于分布式追踪，这有助于追踪消息如何流经系统以及可能出现瓶颈的位置。
> 5 . 日志：时刻关注应用程序日志，记录消费者中的任何错误或异常行为。这些日志可以帮助我们诊断问题。
> 6 . 消费者组命令， 可以使用kafka-consumer-groups 命令来描述消费者组的状态。

请记住，不仅要追踪这些指标，还要针对任何需要关注的场景设置警报。通过这些方法，我们可以在问题还在初始阶段时快速做出响应。
## 实践例子
### 安装驱动包：
`迁移了 go get github.com/IBM/sarama`
```shell
go get github.com/Shopify/sarama
## 迁移了 go get github.com/IBM/sarama
```

### 生产者（Producer）
创建一个函数，用于连接并返回一个Kafka生产者：
```go
func createProducer(brokers []string) (sarama.AsyncProducer, error) {
	config := sarama.NewConfig()
	config.Producer.Return.Successes = true
	config.Producer.Timeout = 5 * time.Second
	return sarama.NewAsyncProducer(brokers, config)
}
```
创建一个函数，用于发送消息到指定的Kafka主题：

```go
func produceMessage(producer sarama.AsyncProducer, topic, value string) {
	message := &sarama.ProducerMessage{
		Topic: topic,
		Value: sarama.StringEncoder(value),
	}

	producer.Input() <- message
}
```

### 消费者（Consumer）
创建一个函数，用于连接并返回一个Kafka消费者：

```go
func createConsumer(brokers []string, groupID string) (sarama.ConsumerGroup, error) {
	config := sarama.NewConfig()
	config.Consumer.Offsets.Initial = sarama.OffsetOldest
	return sarama.NewConsumerGroup(brokers, groupID, config)
}
```
定义一个消费者组对象：
```go
type KafkaConsumerGroupHandler struct {
	ready chan bool
}

func (handler *KafkaConsumerGroupHandler) Setup(_ sarama.ConsumerGroupSession) error {
	close(handler.ready)
	return nil
}

func (handler *KafkaConsumerGroupHandler) Cleanup(_ sarama.ConsumerGroupSession) error {
	return nil
}

func (handler *KafkaConsumerGroupHandler) ConsumeClaim(sess sarama.ConsumerGroupSession, claim sarama.ConsumerGroupClaim) error {
	for message := range claim.Messages() {
		fmt.Printf("消息: 主题=%s 分区=%d 偏移量=%d\n", message.Topic, message.Partition, message.Offset)
		fmt.Printf("消息内容: %s\n", string(message.Value))
		sess.MarkMessage(message, "")
	}
	return nil
}
```

创建一个函数，用于消费指定的Kafka主题：

```go
func consumeMessages(consumer sarama.ConsumerGroup, topics []string) {
	handler := &KafkaConsumerGroupHandler{
		ready: make(chan bool),
	}

	for {
		err := consumer.Consume(context.Background(), topics, handler)
		if err != nil {
			log.Printf("消费者错误: %v", err)
		}

		select {
		case <-handler.ready:
		default:
			return
		}
	}
}
```
### main方法
在 main 函数中调用以上方法展示生产和消费操作：
```go
func main() {
	brokers := strings.Split("localhost:9092", ",")
	topic := "my_topic"
	groupID := "my_group"

	// 创建生产者
	producer, err := createProducer(brokers)
	if err != nil {
		log.Fatal("无法创建生产者:", err)
	}
	defer func() {
		if err := producer.Close(); err != nil {
			log.Fatal("无法关闭生产者:", err)
		}
	}()

	// 发送消息
	produceMessage(producer, topic, "hello world")

	// 创建消费者
	consumer, err := createConsumer(brokers, groupID)
	if err != nil {
		log.Fatal("无法创建消费者:", err)
	}
	defer func() {
		if err := consumer.Close(); err != nil {
			log.Fatal("无法关闭消费者:", err)
		}
	}()

	topics := []string{topic}
	wg := &sync.WaitGroup{}
	wg.Add(1)

	go func() {
		defer wg.Done()
		consumeMessages(consumer, topics)
	}()

	// 监听退出信号
	sigterm := make(chan os.Signal, 1)
	signal.Notify(sigterm, os.Interrupt)
	<-sigterm

	// 优雅关闭消费者
	wg.Wait()
}
```

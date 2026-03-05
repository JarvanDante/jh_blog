---
author: "jarvan"
title: "Outbox + MQ 消费者分层：如何避免“发了消息却没落地”"
date: 2025-05-26T10:00:00+08:00
description: "结合 jh 项目实践，系统讲清 outbox + MQ 的一致性设计、消费者分层、幂等重试与可观测治理。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - golang
  - outbox
  - rabbitmq
  - consistency
categories:
  - Golang
  - Microservice
---

## 背景：为什么会出现“消息发了，但数据没落地”

在资金、游戏、报表等链路中，我最早遇到的问题是：业务数据库更新成功，但下游系统（例如 ClickHouse、游戏账户系统）没有同步成功。根因通常是：

- 业务提交成功后，MQ 发送失败
- MQ 发送成功后，消费者写库失败
- 消费端重复消费导致脏数据

这类问题在分布式系统里是常态，不能靠“单次调用成功”来设计。

---

## jh 项目里的落地模式

我采用的是 **本地事务 + Outbox + MQ + 消费者幂等**：

1. 业务事务内：写业务表 + 写 outbox 事件（同库同事务）
2. outbox-relay 常驻进程：扫描待投递事件，投到 RabbitMQ
3. 消费者按业务域分层：balance/game/clickhouse 等
4. 消费失败重试 + 死信处理 + 运营可补偿

这样做的核心价值是：避免跨库分布式事务，转成可控的“最终一致性”。

---

## 消费者分层的关键点

### 分层原则

- **按业务责任拆分消费者**：避免“一个消费者处理全世界”
- **按副作用拆分 topic/queue**：账户同步、报表同步、通知发送分开
- **按重要性分优先级**：资金链路优先级最高

### 我在实践中的结构建议

- `mq-outbox-relay`：只负责投递，不做业务逻辑
- `balance-flow-consumer`：资金流水相关
- `game-mq-consumer`：游戏域同步
- `game-clickhouse-sync`：分析库同步

并且保留一个 `mq-outbox-once` 定时兜底，防止 relay 暂时故障时事件积压过久。

---

## 幂等与重试策略

### 幂等主键

消费端必须有业务幂等键，例如：

- 订单号 `order_no`
- 业务事件 ID（outbox_id）

在 ClickHouse 写入前，我会先做去重查询（或基于主键/唯一键策略）。

### 重试策略

- 可重试错误：网络抖动、短暂超时、下游 5xx
- 不可重试错误：字段缺失、协议不兼容、数据格式错

实践中，**解析失败消息直接丢弃并告警**，**写库失败消息 requeue 重试**，避免“坏消息污染整队列”。

---

## 示例：outbox 事件结构（简化）

```go
type OutboxEvent struct {
    ID         int64
    Topic      string
    EventKey   string // 幂等键
    Payload    string
    Status     string // pending/sent/failed
    RetryCount int
    CreatedAt  time.Time
}
```

消费侧示例（伪码）：

```go
func Handle(msg Message) error {
    evt := parse(msg)
    if !evt.Valid() {
        return AckAndDrop // 不可重试
    }

    if alreadyProcessed(evt.EventKey) {
        return Ack // 幂等命中
    }

    if err := applyBusiness(evt); err != nil {
        return NackRequeue // 可重试
    }

    markProcessed(evt.EventKey)
    return Ack
}
```

---

## 可观测性：不要只看“发送成功”

我会关注四组指标：

1. outbox pending 数量（积压）
2. mq_sent / mq_failed（投递质量）
3. consumer success / retry / dead-letter
4. 目标库落地指标（例如 ch_synced / ch_failed）

一个实战经验：**只看 MQ 成功没有意义，必须看最终落地成功率。**

---

## 常见坑与规避

1. relay 和业务消费者混在一个进程，互相影响
2. 没有幂等，重试把数据写脏
3. 失败消息没有死信，导致无限重试
4. 没有补偿任务，历史失败消息无法修复

---

## 总结

Outbox + MQ 不会让系统“永不出错”，但它能把错误变成可恢复、可观测、可补偿的问题。这就是工程价值。

---
author: "jarvan"
title: "Outbox + MQ 在 PHP 项目中的一致性落地"
date: 2020-08-17T10:00:00+08:00
description: "Outbox + MQ 在 PHP 项目中的一致性落地"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - php
  - architecture
categories:
  - Php
---

很多团队在“写库成功但消息没发出去”这里翻车。我后来统一改成 Outbox 模式。

## 为什么不用跨库事务

2PC 成本高、故障复杂。对大多数业务来说，**本地事务 + 最终一致性**更现实。

## 实现步骤

1. 同事务写业务表与 outbox 表
2. relay 进程扫描 pending 事件发 MQ
3. 消费端幂等处理

```php
DB::transaction(function () use ($order) {
    Order::create($order);
    Outbox::create([
        'topic' => 'order.created',
        'event_key' => $order['order_no'],
        'status' => 'pending',
    ]);
});
```

## 我踩过的坑

- relay 与业务 API 混部，互相抢资源
- event_key 不稳定导致幂等失效
- 失败重试没有上限，堆积爆炸

## 结语

Outbox 不是银弹，但它把一致性问题变成了“可观测、可补偿、可回放”的工程问题。


## 经验补充：容易被忽略的“最后一公里”

在项目后期，我发现真正拉开工程水平差距的，不是“会不会写”，而是是否处理了这些细节：

1. **幂等键生命周期**：TTL 太短会误重放，太长会占用过多内存
2. **重试策略分级**：网络错误可重试，参数错误应直接落死信
3. **可观测字段统一**：`trace_id`、`request_id`、`biz_id` 必须全链路一致
4. **跨团队约定**：错误码与回包结构稳定，避免联调时反复返工

## 一个“事故预防”清单

- 关键流程必须有演练脚本（至少月度一次）
- 核心指标必须有阈值告警（延迟、错误率、队列堆积）
- 高风险功能必须支持一键关闭
- 每次事故都产出“可执行”的改进项，而不是口号

## 延伸阅读

- RabbitMQ 数据可靠性：https://www.rabbitmq.com/docs/reliability
- Stripe 幂等重试设计：https://docs.stripe.com/api/idempotent_requests
- PHP-FPM 进程管理参数：https://www.php.net/manual/en/install.fpm.configuration.php


## 线上案例切片（真实排障视角）

这部分我补一个更接近生产的案例切片，方便读者理解“从告警到修复”的完整路径。

### 告警现象

- 时间窗口：发布后 10~20 分钟
- 首条告警：`5xx rate` 超阈值
- 伴随信号：`P95 latency` 抬升、部分队列出现堆积

### 快速定位路径

1. 先看网关与应用错误码分布，确认是入口问题还是下游问题。  
2. 拉取最近 200 行关键日志，对比发布前后相同请求差异。  
3. 对高频 SQL / 外部依赖做抽样验证，确认是否为热点瓶颈。  
4. 若 15 分钟内无法稳定恢复，执行预案回滚。

### 处置结果（模板化）

- 影响范围：部分高频接口
- 处置动作：参数回退 + 降级开关 + 队列限速
- 恢复时间：通常 10~30 分钟
- 根因归类：配置漂移 / 索引失配 / 重试放大（三类最常见）

---

## 实操命令片段（便于读者复现）

```bash
# 1) 查看服务状态
docker compose ps

# 2) 查看最近日志（按服务）
docker logs --tail=200 php-fpm || true
docker logs --tail=200 nginx || true

# 3) 核验数据库慢查询（示意）
# SHOW FULL PROCESSLIST;
# EXPLAIN ANALYZE <SQL>;

# 4) 快速回滚（示意）
# docker compose up -d --force-recreate <service>
```

---

## 读者常问（FAQ）

### Q1：这套方案适合中小团队吗？
适合。关键是先把“最小治理闭环”做起来：日志统一、核心告警、回滚预案。

### Q2：一定要一次性做全套吗？
不需要。建议按优先级迭代：**可用性 > 一致性 > 性能优化 > 架构美化**。

### Q3：怎么证明自己做过实战？
拿出三类证据：

- 发布与回滚记录
- 指标变化前后对比
- 事故复盘文档（含改进项闭环）

---

## 章节小结

做高级 PHP 不是“会写语法糖”，而是把系统做成：

- 可上线
- 可观测
- 可回滚
- 可持续演进

这也是我在项目里长期坚持的方法论。


> 补充：如果团队已有 APM，建议把 trace_id 回写到业务日志，排障效率会再提升一个台阶。

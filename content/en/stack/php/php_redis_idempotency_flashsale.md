---
author: "jarvan"
title: "PHP + Redis 防超卖与幂等设计：秒杀场景的工程解法"
date: 2020-08-02T10:00:00+08:00
description: "PHP + Redis 防超卖与幂等设计：秒杀场景的工程解法"
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

秒杀问题本质是“高并发下同一库存被重复消费”。我用过最稳的一套是：Redis 预扣 + 订单幂等 + 最终对账。

## 核心设计

- Redis Lua 保证扣减原子性
- 请求带 `idempotency_key`
- 订单落库唯一索引兜底

```lua
-- stock_deduct.lua
local stock = tonumber(redis.call('GET', KEYS[1]) or '0')
if stock <= 0 then return -1 end
redis.call('DECR', KEYS[1])
return 1
```

## PHP 侧幂等校验

```php
if (Redis::setnx('idem:'.$key, 1) === 0) {
    throw new BizException('重复请求');
}
Redis::expire('idem:'.$key, 300);
```

## 实战注意

- 预扣成功不代表订单一定成功，要有补偿
- MQ 消费失败要能重试且不重复落单
- 活动结束必须做 Redis 与 DB 对账

这套方案不是最“学术”，但在业务高峰期足够抗打。


## 架构细化：把“技术方案”翻译成“可运营系统”

很多文章停留在代码层，但线上系统要考虑运营与故障恢复：

- **谁来报警**：告警分级（P1/P2/P3）和接收人
- **谁来决策**：是否降级、是否回滚、是否熔断
- **谁来复盘**：故障窗口、影响范围、后续改进项

我通常会把改动拆成“功能发布”和“治理发布”两类：

- 功能发布关注需求完成度
- 治理发布关注稳定性指标与风险收敛

这样做之后，团队对线上问题的反应会快很多。

## 一份可复用的排障顺序

```text
入口是否正常 -> 网关是否正常 -> 服务依赖是否正常 -> 数据是否一致 -> 是否可回滚
```

## 延伸阅读

- OWASP API Security Top 10：https://owasp.org/www-project-api-security/
- Kubernetes Deployment（滚动升级/回滚思路）：https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- Laravel Queue（队列落地实践）：https://laravel.com/docs/10.x/queues


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


> 补充：对读多写少接口，先做缓存命中率优化，收益通常比“盲目扩容”更高。

---
author: "jarvan"
title: "Redis 在 Python 项目中的高并发实践：缓存、分布式锁与防穿透"
date: 2022-11-22T10:00:00+08:00
description: "Redis 在 Python 项目中的高并发实践：缓存、分布式锁与防穿透"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - python
  - microservice
categories:
  - Python
  - Architecture
---


## Redis 在高并发里的三类核心用法

1. 缓存热点数据
2. 分布式锁控制并发修改
3. 防穿透/击穿/雪崩

## 缓存模式

```python
def get_user_profile(uid):
    key = f"u:{uid}:profile"
    data = redis.get(key)
    if data:
        return json.loads(data)
    row = db.query_user(uid)
    redis.setex(key, 300, json.dumps(row))
    return row
```

## 分布式锁（简化）

```python
token = uuid4().hex
locked = redis.set('lock:order:123', token, nx=True, ex=10)
if locked:
    try:
        do_update()
    finally:
        # 需 Lua 校验 token 再删，避免误删
        release_lock('lock:order:123', token)
```

## 防穿透

- 布隆过滤器
- 空值缓存（短 TTL）

## 防雪崩

- TTL 加随机抖动
- 热点 key 预热

## 总结

Redis 不是万能加速器，关键是把“读写模型、TTL、并发控制”设计清楚。


## 代码进一步增强：可观测与容错骨架

下面给一个“生产可用”骨架思路（简化版）：

```python
import time
from contextlib import contextmanager

@contextmanager
def timer(metric, labels):
    start = time.time()
    try:
        yield
    finally:
        metric.labels(**labels).observe((time.time() - start) * 1000)


def safe_call(fn, fallback=None):
    try:
        return fn()
    except TimeoutError:
        return fallback() if fallback else None
```

使用方式：

- 外部依赖调用统一走 `safe_call`
- 关键路径统一打 latency 指标
- 失败路径统一记录 trace_id + error_code

这样做可以显著降低“线上问题定位完全靠猜”的情况。

---

## 发布与回滚策略（建议写进团队 SOP）

- 发布前：检查配置差异、数据库变更、依赖健康状态
- 发布中：按 5% -> 20% -> 全量放量
- 发布后：观察 30 分钟关键指标
- 回滚条件：提前量化，不临场争论

可执行的回滚触发条件示例：

- 错误率超过基线 2 倍且持续 5 分钟
- P95 超过阈值且无下降趋势
- 关键业务转化率明显异常

---

## 经验总结（工程化视角）

一个技术方案是否成熟，不看它“写得多漂亮”，而看它在真实流量下是否：

- 可观测
- 可回滚
- 可补偿
- 可持续演进

把这四件事做扎实，系统稳定性通常会进入新的台阶。


## 深度扩展：从“方案可行”到“长期稳定”的最后一步

很多技术文章会停在“功能完成”，但生产系统真正的难点是持续运营。这里补充一个更完整的落地框架，建议直接用于团队实践。

### 1）把风险前置到设计阶段

在编码前先做一个 30 分钟风险评审：

- 依赖是否有单点（DB/MQ/配置中心）
- 失败后是否可重试
- 重试后是否会重复副作用
- 是否有可回滚路径
- 是否有兜底降级能力

如果这些问题没有答案，代码再漂亮也会在真实流量下暴露问题。

### 2）把“成功标准”量化

建议每篇提到的方案都配一个可验证目标：

- 错误率下降多少
- P95 收敛多少
- 堆积恢复时间缩短多少
- 故障平均恢复时长（MTTR）是否下降

这样在复盘时可以拿数据说话，而不是主观评价“感觉有优化”。

### 3）故障演练制度化

每月做一次小型演练，场景可以固定为：

- 下游依赖超时
- 队列消费阻塞
- 配置错误发布
- 数据库连接池耗尽

演练目标不是“演得好看”，而是验证 runbook 是否真的可执行。

### 4）改进项要能落地

复盘里最常见问题是“结论很多，行动很少”。建议改进项必须包含：

- owner（负责人）
- deadline（截止时间）
- metric（验收指标）
- rollback plan（失败兜底）

---

## 附：团队级实施模板（可复制）

```text
[变更名称]
- 目标：
- 影响范围：
- 发布计划：5% -> 20% -> 100%
- 监控看板：错误率 / P95 / backlog / 下游超时
- 回滚条件：
- 回滚命令：
- 值班负责人：
```

把模板沉淀下来，团队整体稳定性会比“个人英雄式排障”提升明显。

---

## 章节收口

工程经验的价值，不在于“会不会某个框架”，而在于是否建立了一套能在复杂环境下反复成功的交付机制。只要机制稳定，业务增长时系统也能稳住。

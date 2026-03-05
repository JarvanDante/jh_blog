---
author: "jarvan"
title: "MySQL 到 ClickHouse 同步失败的根因定位：从 ch_failed 到表结构对齐"
date: 2025-05-31T10:00:00+08:00
description: "一次真实同步故障复盘：如何从指标异常、消费者日志到 schema 差异，完成 ClickHouse 链路修复。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - golang
  - mysql
  - clickhouse
  - postmortem
categories:
  - Golang
  - Database
---

## 现象

线上某时段出现：

- `mq_sent` 持续增长
- `ch_synced = 0`
- `ch_failed` 快速增长

业务层看似正常，但分析报表完全不更新。

---

## 初步判断

这类指标组合说明：

1. 业务侧 outbox 与 MQ 发送链路大概率正常
2. 问题在消费端到 ClickHouse 落地阶段

排查优先级：

1. 消费者是否在运行
2. ClickHouse 连接是否正常
3. 插入 SQL 与表结构是否一致

---

## 定位过程

消费端日志出现关键报错：

> PrepareBatch failed: No such column agent_id in table game_analytics.bet_records

最终确认：应用写入字段包含 `agent_id`，但目标表未加该列。

---

## 根因

**根因是 schema 演进不一致**：

- 业务模型升级后新增字段
- ClickHouse 表结构未同步变更
- 批量写入阶段直接失败，触发整批重试

---

## 修复动作

```sql
ALTER TABLE game_analytics.bet_records
ADD COLUMN IF NOT EXISTS agent_id UInt32 DEFAULT 0;
```

修复后观察：

- `ch_synced` 开始增长
- `ch_failed` 不再持续攀升
- 报表恢复更新

---

## 技术细节：为什么会“整批失败”

在批量消费逻辑里，如果 PrepareBatch/Append/Send 任一环节失败，整批消息会被 Nack 并重回队列。这种策略保证了“要么整批成功、要么重试”，但副作用是一个字段问题会放大为持续失败。

---

## 防复发机制

1. **发布前 schema diff 检查**
   - 对比应用模型字段与 CH 表结构
2. **启动前自检**
   - 关键表和关键列不存在则 fail fast
3. **错误分类**
   - 结构性错误（不可重试）单独告警，不做无限重试
4. **链路指标看板**
   - 不只看 MQ 成功，要看最终落地

---

## 示例：上线前 schema 校验思路（伪码）

```go
func EnsureColumns(db CH, table string, required []string) error {
    cols := db.Describe(table)
    miss := diff(required, cols)
    if len(miss) > 0 {
        return fmt.Errorf("missing columns: %v", miss)
    }
    return nil
}
```

---

## 总结

这次故障本质不是“消费挂了”，而是“模型和存储契约失配”。在微服务体系里，契约治理比单点修复更重要。

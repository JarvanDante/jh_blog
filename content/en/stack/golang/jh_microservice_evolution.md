---
author: "jarvan"
title: "从单体到微服务：我的 jh 项目架构演进实录"
date: 2025-05-12T10:30:00+08:00
description: "结合 jh 项目真实落地，复盘从单体到微服务的演进路径、关键技术决策、稳定性治理与工程化经验。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - golang
  - microservice
  - architecture
  - docker
  - gateway
categories:
  - Golang
  - Architecture
---

## 为什么要从单体走向微服务

在 jh 项目早期，业务集中在单体服务里，优势是开发快、上线快，但随着模块增长，很快出现了几个典型问题：

1. **发布耦合严重**：一个小改动要整包发布，回归成本高。
2. **故障域太大**：某个模块波动可能拖垮全站。
3. **扩容不精细**：热点模块（如游戏、资金）和冷模块绑在一起扩容，资源浪费。
4. **职责边界不清**：代码越来越难维护，排障路径越来越长。

因此我把架构逐步演进为：

- `jh_gateway`：统一入口、鉴权、路由分发、限流
- `jh_app_service`：前台账户与站点基础能力
- `jh_game_service`：游戏域、洗码额度、分析数据链路
- `jh_balance_service`：余额、流水、资金域操作
- `jh_manage_service`：总后台管理链路

基础设施层引入了：**Consul + RabbitMQ + Redis + MySQL + ClickHouse + MinIO + Nginx**。

---

## 演进路径：不是重写，而是“可回滚”的渐进拆分

我采用的是“绞杀者模式（Strangler Pattern）”而非一次性重构：

1. 先加 `gateway`，把所有入口收口。
2. 按业务域拆服务，先拆波动最大的模块（游戏、资金）。
3. 保留兼容路由，确保旧链路可回退。
4. 引入异步 outbox 和消费链路，减少强耦合同步调用。

这个策略的核心是：**每一步都可观测、可回滚**。

---

## 核心链路设计

### 1) 请求入口统一

Nginx 只做反代与静态资源，业务入口统一进网关：

- `/api/frontend` -> 前台路由
- `/api/backend` -> 站点后台
- `/api/manage` -> 总后台

网关层负责：

- JWT 校验
- Domain Resolver（按域名映射站点）
- 限流（Redis）
- 路由分发到对应 gRPC service

### 2) 数据存储分层

- **MySQL**：交易与主业务数据（强一致）
- **ClickHouse**：分析与报表（高吞吐查询）
- **MinIO**：图片/资源对象存储

### 3) 异步化关键路径

资金、游戏、报表同步采用 outbox + MQ，避免跨服务事务放大故障。

---

## 一个关键稳定性案例：ClickHouse 同步失败

线上曾出现：

- `mq_sent > 0`
- `ch_synced = 0`
- `ch_failed` 持续增长

最终定位到表结构与写入字段不一致：`game_analytics.bet_records` 缺少 `agent_id` 列，导致批量 `PrepareBatch` 失败。

修复：

```sql
ALTER TABLE game_analytics.bet_records
ADD COLUMN IF NOT EXISTS agent_id UInt32 DEFAULT 0;
```

这个问题让我把“发布前 schema 对齐检查”固化为上线前必做项。

---

## 工程化与部署：从 ACK 转向低成本 Compose

在成本与复杂度平衡下，项目从 ACK 逐步转向 `docker-compose`（ECS + RDS）方案。

实践中的经验：

- 镜像必须统一架构（`amd64`），避免 `arm64/amd64` 运行时不匹配。
- 服务间通信一律走服务名，不写容器 IP。
- 固定 tag 分批发布，必要时可快速回滚。
- Nginx 反代务必透传 `Host`，否则站点解析会错位。

构建示例（多架构）：

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t your/image:tag \
  --push .
```

---

## 我在这个阶段形成的架构原则

1. **先边界，后细节**：先定义服务职责，再做代码拆分。
2. **稳定优先于优雅**：线上可观测、可回滚，比“理论最优”更重要。
3. **单一事实源**：配置优先级必须明确，避免“改了不生效”。
4. **故障常态化治理**：把事故处理沉淀为 SOP 和检查清单。

---

## 这段经历的价值

这个项目不是“写了几个接口”，而是完整经历了：

- 架构拆分与边界治理
- 网关与服务发现实践
- 异步一致性链路落地
- 生产故障定位与恢复
- 成本约束下的部署工程化

我更在意的是：**能把系统从“能跑”推进到“可运营、可维护、可扩展”。**

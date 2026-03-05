---
author: "jarvan"
title: "ACK(阿里云K8S) 到 Docker Compose：低成本微服务落地实践"
date: 2025-06-15T10:00:00+08:00
description: "复盘 jh 项目从 ACK 到 Docker Compose 的迁移决策、成本收益、发布策略与踩坑总结。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - golang
  - docker
  - compose
  - k8s
categories:
  - Golang
  - DevOps
---

## 迁移动机

ACK 的弹性和治理能力很强，但在当前阶段，成本和运维复杂度对小团队压力较大。为了更快迭代和控制支出，我把 jh 项目逐步迁到 ECS + Docker Compose。

---

## 不是“降级”，而是阶段性最优解

迁移判断标准：

- 当前并发规模可控
- 团队人力有限
- 以快速交付和稳定运行为优先

在这个阶段，Compose 的可控性和维护成本明显更友好。

---

## 迁移落地结构

- 入口：Nginx
- 网关：jh_gateway
- 业务服务：app/game/balance/manage
- 基础组件：Consul/RabbitMQ/Redis/MySQL/ClickHouse/MinIO

统一通过 compose 服务名通信，避免硬编码容器 IP。

---

## 发布策略

1. 镜像固定 tag（避免 latest 漂移）
2. 分批发布（先网关后服务）
3. 每次发布后跑冒烟清单
4. 保留上一版镜像快速回滚

常用流程：

```bash
docker compose pull
docker compose up -d --force-recreate
```

---

## 典型坑：架构不匹配

Mac 本地构建经常生成 `arm64`，ECS 多为 `amd64`，导致容器起不来。

解决：

```bash
docker buildx build --platform linux/amd64 -t your/image:tag --push .
```

---

## 迁移后的收益

- 成本更可控
- 发布节奏更快
- 排障链路更短

同时也有边界：

- 缺少 K8S 原生弹性能力
- 多环境治理要靠更严谨的流程

---

## 结论

架构选择不是信仰问题，而是阶段问题。对当前 jh 项目而言，Compose 是“成本、效率、稳定性”三者平衡下的最优解。

---
author: "jarvan"
title: "一份可落地的微服务本地编排手册：mydocker_dev 目录、docker-compose.yml 与模块职责全解"
date: 2025-05-08T11:20:00+08:00
description: "基于 /Users/wangdante/D/mydocker_dev 的真实结构，系统梳理目录分层、Compose 编排、服务职责、链路流向与运维注意事项。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - golang
  - microservice
  - docker
  - compose
  - architecture
categories:
  - Golang
  - Architecture
---

## 写在前面

这篇文章不是概念介绍，而是基于我实际使用的目录：

`/Users/wangdante/D/mydocker_dev`

目标是把三件事讲清楚：

1. 当前目录结构在做什么  
2. `docker-compose.yml` 是如何编排整个系统的  
3. 各目录（配置、反代、日志、定时任务）在链路中的职责

> 说明：以下内容用于架构说明，生产环境请将密钥/密码改为环境变量或密钥管理方案。

---

## 1) 目录结构（tree -N）

```text
/Users/wangdante/D/mydocker_dev
|-- cf_origin.key
|-- cf_origin.pem
|-- cron
|   `-- crontab
|-- docker-compose.yml
|-- fluent-bit
|   |-- fluent-bit.conf
|   `-- parsers.conf
|-- jh_app_service
|   `-- config
|       `-- config.yaml
|-- jh_balance_service
|   `-- config
|       `-- config.yaml
|-- jh_game_service
|   `-- config
|       `-- config.yaml
|-- jh_gateway
|   `-- config
|       `-- config.yaml
|-- jh_manage_service
|   `-- config
|       `-- config.yaml
|-- mysql
|   `-- data
`-- nginx
    `-- config
        |-- backend.conf
        |-- h5.conf
        |-- manage.conf
        `-- upstreams.conf
```

这份结构非常典型：

- `docker-compose.yml`：总编排入口
- `nginx/config`：按业务域拆分反代配置
- `jh_*_service/config`：各服务独立配置目录
- `cron/crontab`：定时任务定义
- `fluent-bit`：日志采集与解析
- `cf_origin.*`：Cloudflare 源站证书

---

## 2) docker-compose.yml 总览：四层编排

从内容上看，这份 compose 可以分四层：

### A. 入口与观测层

- `nginx`：统一外部入口（80）
- `jaeger`：链路追踪
- `elasticsearch + kibana + fluent-bit`：日志采集与检索

### B. 基础设施层

- `consul`：服务发现
- `mysql`：业务主库
- `redis`：缓存/限流
- `rabbitmq`：异步消息总线
- `minio`：对象存储
- `clickhouse`：分析查询

### C. 核心业务层

- `jh_gateway`
- `jh_app_service`
- `jh_balance_service`
- `jh_game_service`
- `jh_manage_service`
- `jh_h5` / `jh_backend` / `jh_manage_backend`

### D. 异步执行层

- `job_runner`（cron）
- `jh_game_clickhouse_sync`
- `jh_game_mq_consumer`
- `jh_balance_flow_consumer`
- `jh_balance_mq_consumer`
- `jh_app_outbox_relay`
- `jh_balance_outbox_relay`
- `jh_game_outbox_relay`

这层次设计的价值是：**入口、业务、基础设施、异步任务职责清晰，排障路径明确。**

---

## 3) 关键链路说明

## 3.1 HTTP 请求链路

```text
Client -> Nginx -> jh_gateway -> (app/balance/game/manage service)
```

网关负责统一鉴权、路由、中间件处理，避免前端直接依赖内部服务地址。

## 3.2 异步一致性链路

```text
业务写库 -> outbox 表 -> outbox-relay -> RabbitMQ -> 消费者 -> 目标系统（MySQL/ClickHouse/游戏侧）
```

这是典型的 Outbox + MQ 最终一致性实践，避免跨库分布式事务。

## 3.3 日志与追踪链路

```text
Docker logs -> Fluent Bit -> Elasticsearch -> Kibana
业务span -> Jaeger
```

用于快速定位 502、超时、消息丢失、慢查询等问题。

---

## 4) 各目录与文件职责（逐项说明）

## 4.1 `docker-compose.yml`

系统级编排文件，定义了：

- 全部容器服务
- 依赖顺序（`depends_on` + healthcheck）
- 网络与固定地址（`karson` 网段）
- 持久化卷（MySQL/CH/MinIO/ES/RabbitMQ 等）

### 建议

- 生产环境尽量少用固定 IP，优先服务名
- 密钥改为 `.env` / 密钥系统，不要明文入库

---

## 4.2 `nginx/config/`

- `upstreams.conf`：定义上游服务
- `h5.conf`：前台 H5 路由与静态规则
- `backend.conf`：站点后台
- `manage.conf`：总后台

### 常见坑

- upstream 端口写错（容器端口 vs 主机映射端口）
- 缺少 `Host` 透传导致站点解析失败
- history 路由未做 fallback 导致刷新 404

---

## 4.3 `jh_*_service/config/config.yaml`

每个服务的核心配置入口，通常包含：

- DB 连接
- Redis / RabbitMQ / Consul
- JWT / 上传策略 / 第三方接口

### 实战经验

“配置改了不生效”多数是多配置源覆盖导致。建议固定优先级并在启动日志打印最终生效值。

---

## 4.4 `cron/crontab` + `job_runner`

用于执行定时补偿和周期任务。当前方案使用 `docker:cli` 镜像并挂载 `docker.sock`，由 `crond` 执行计划任务。

### 注意

如果容器内没有 docker cli，会出现任务“看似触发、实际空跑”。

---

## 4.5 `fluent-bit/`

- `fluent-bit.conf`：输入输出管道
- `parsers.conf`：日志解析规则

目标是把容器日志统一送入 ES，便于 Kibana 检索。

---

## 4.6 `mysql/data` 与各 volume

数据持久化目录与卷定义，防止容器重建导致数据丢失。

建议按环境分离卷，并定期备份关键库。

---

## 5) 网络与端口设计评价

当前使用 `172.31.0.0/16` 自定义网段并给关键容器分配固定 IP，优点是排障直观。  
但长期看，建议逐步过渡到**服务名寻址**，降低重编排和迁移成本。

---

## 6) 这份编排的亮点与改进建议

### 亮点

1. 基础设施与业务服务拆分清晰
2. 异步消费者独立容器化，职责明确
3. 可观测体系（日志+追踪）完整
4. 反向代理按业务域拆配置，可维护性较好

### 建议改进

1. 密钥外置（ENV/Secret）
2. 健康检查覆盖更多业务服务
3. 统一服务名与端口语义，减少误配
4. 增加发布前检查脚本（端口、配置、依赖）

---

## 7) 快速启动与验证建议

```bash
docker compose pull
docker compose up -d

docker compose ps
docker logs --tail=100 nginx
docker logs --tail=100 jh_gateway
```

验证优先级：

1. 基础设施是否健康（consul/rabbitmq/mysql/redis）
2. 网关健康接口是否可用
3. 前后台关键登录链路是否通
4. MQ 消费与 ClickHouse 同步指标是否增长

---

## 总结

`mydocker_dev` 这套编排已经具备一个中小规模微服务系统的完整骨架：入口、网关、服务发现、异步一致性、可观测、对象存储、分析库都齐全。  

它最大的价值不在“组件多”，而在于：

- 每层职责有边界
- 故障可定位
- 发布可迭代

这也是我在 Go 微服务工程里最看重的能力：**让系统长期稳定地演进，而不是只在某次演示中跑通。**

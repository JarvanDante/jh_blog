---
author: "jarvan"
title: "网关与服务发现：jh 项目中的路由收口、故障定位与稳定性设计"
date: 2025-05-21T14:10:00+08:00
description: "结合 jh_gateway + Consul 的真实案例，详解网关路由、服务发现、故障模式与实战排障方法。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - golang
  - gateway
  - consul
  - grpc
  - nginx
categories:
  - Golang
  - Microservice
---

## 背景：为什么“网关 + 服务发现”是必选项

在多服务架构里，客户端不应该直接感知每个服务地址。否则会出现：

- 路由散落在前端/各服务配置中
- 版本切换难
- 故障排查链路不清

jh 项目把所有入口收口到 `jh_gateway`，通过 Consul 做服务发现，实现：

- 统一鉴权
- 统一路由
- 统一限流
- 统一可观测

---

## 网关职责分层（实践版本）

### 1) 协议与入口

- 外部入口：HTTP（Nginx -> Gateway）
- 内部调用：gRPC（Gateway -> 业务服务）

### 2) 中间件链

典型链路：

1. CORS
2. DomainResolver（按 Host 解析站点）
3. Logging / Trace
4. RateLimit
5. RequestParser
6. Auth（按路由策略跳过或校验）
7. CircuitBreaker

### 3) 路由分发

- `/api/frontend/*` -> app_service
- `/api/game/*` -> game_service
- `/api/balance/*` -> balance_service
- `/api/manage/*` -> manage_service

---

## 服务发现的关键设计

### 注册信息必须稳定

服务注册名、调用名、健康检查名必须一致。  
一旦命名漂移，就会出现经典问题：`no healthy instance`。

### 端口语义必须统一

线上常见坑是“容器内端口”和“主机映射端口”混淆。  
在容器网络中，网关应访问服务的**容器内部端口**，而不是宿主机映射端口。

---

## 真实故障案例复盘

### 案例 A：`502 Bad Gateway`

现象：

- 前端登录请求返回 502
- Nginx 日志 `connect() failed (111)`

根因：

- upstream 指向了错误端口（配置与服务实际监听不一致）

修复动作：

- 用网关启动日志确认实际监听端口
- Nginx upstream 改为网关真实可达端口
- 容器内 `curl` 验证连通性

### 案例 B：路由通了但业务 500

现象：

- HTTP 200/网关可达
- 业务返回“登录服务暂不可用”

根因：

- 下游服务数据库账号权限错误（MySQL 1045）

关键结论：

- 网关问题与业务问题要分层定位，别把 500 全归为网关故障。

---

## 我的排障方法论（可复用）

我把排障固定成 4 层检查：

1. **入口层**：DNS / Nginx / SSL
2. **网关层**：路由匹配、鉴权、中间件
3. **发现层**：Consul 注册状态与实例健康
4. **业务层**：下游服务日志、DB/MQ 连接状态

常用命令（示例）：

```bash
# 看网关关键日志
docker logs --tail=200 jh_gateway

# 容器内验证到下游服务连通
docker exec -it nginx sh -c "curl -I http://jh_gateway:8000/gateway/health"

# 核验服务实例状态（示意）
# consul catalog/services 或健康检查接口
```

---

## Go 侧一个可读性更好的“路由分发”示例

```go
func Dispatch(path string) (service string) {
    switch {
    case strings.HasPrefix(path, "/api/frontend/"):
        return "jh_app_service"
    case strings.HasPrefix(path, "/api/game/"):
        return "jh_game_service"
    case strings.HasPrefix(path, "/api/balance/"):
        return "jh_balance_service"
    case strings.HasPrefix(path, "/api/manage/"):
        return "jh_manage_service"
    default:
        return ""
    }
}
```

这类分发逻辑要配合：

- 路由单测
- 注册名常量化
- 启动时配置自检（无效路由直接 fail fast）

---

## 稳定性改进清单（我在项目里落地过）

1. 网关启动时输出路由映射与目标服务名
2. 服务注册名集中管理，避免硬编码字符串散落
3. 关键链路加健康探针（`/gateway/health` + 下游探测）
4. 统一错误翻译与透传，降低排障噪音
5. 发布前执行“端口/服务名/配置”三件套检查

---

## 我在这个模块的能力证明

我在网关与服务发现这一块，不只是“会配 Nginx/Consul”，而是具备：

- 从日志中快速定位层级问题的能力
- 把临时修复沉淀为稳定机制的能力
- 在成本约束和业务压力下做工程取舍的能力

这也是我理解的后端工程价值：
**不是让系统跑起来，而是让系统长期稳定地跑下去。**

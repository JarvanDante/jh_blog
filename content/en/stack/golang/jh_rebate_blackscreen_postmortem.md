---
author: "jarvan"
title: "接口“成功”但页面黑屏？一次 /rebate 线上事故复盘"
date: 2025-06-05T10:00:00+08:00
description: "复盘 jh_h5 /rebate 黑屏事件：从无报错现象到字段容错与网关输出策略双保险。"
draft: false
hideToc: false
enableToc: true
enableTocContent: true
tags:
  - golang
  - frontend
  - incident
  - api
categories:
  - Golang
  - Frontend
---

## 事故现象

- `/rebate` 页面打开后短暂显示，随后黑屏
- 接口返回 `查询成功`，无明显 500
- 控制台初期缺少有效报错

这是典型“接口成功 ≠ 页面可用”的故障。

---

## 定位方法

我用了三步法：

1. **最小化页面**：先确认路由壳层是否正常
2. **二分恢复组件**：逐段恢复模块，定位触发点
3. **打开可观测**：增加 `router.onError` 日志与 chunk 失败自愈

最终锁定：洗码额度字段缺失触发格式化崩溃。

---

## 根因

前端代码默认认为某字段一定存在，执行了类似 `undefined.toLocaleString()`。当接口只返回“查询成功”但没带数值字段时，页面直接异常。

---

## 修复方案（双保险）

### 前端侧

- 新增 `toSafeNumber`
- `formatMoney` / `formatMoneyPrecise` 全部容错
- `fetchQuota` normalize 默认值

### 网关侧

- Protobuf JSON 输出开启 `EmitUnpopulated`
- 让零值字段也输出，减少前端字段缺失概率

---

## 关键代码思路

```ts
function toSafeNumber(val: unknown): number {
  const n = Number(val)
  return Number.isFinite(n) ? n : 0
}

function formatMoney(val: unknown) {
  return toSafeNumber(val).toLocaleString('en-US', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  })
}
```

---

## 为什么要做“前后端双保险”

只改前端不够：后端未来变更仍可能出现契约漂移。  
只改后端也不够：网络、灰度、缓存都可能导致字段异常。  

所以要同时做：

- 前端永远容错
- 后端尽量输出完整契约

---

## 复盘结论

“成功响应 + 黑屏”是前端稳定性体系的试金石。真正稳定的系统，不会因为一个缺省字段直接挂页面。

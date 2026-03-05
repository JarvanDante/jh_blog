---
author: "jarvan"
title: "亿级仓库礼物的建设与演进"
date: 2023-05-18
description: "酷狗直播中亿级仓库礼物的建设与演进。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- db
categories:
- db
image: images/docImages/liwuzb.png
---

## 仓库礼物概览
![/images/docImages/fanxing1.png](/images/docImages/fanxing1.png)
1. 1.普通送礼
充值星币
2. 2.仓库送礼
活动赢星币

### 需求：
+ 存储要求：
     1. 数量
     2. 礼物种类
     3. 过期属性
     4. 绑定属性

+ 送礼功能
+ 入/出仓库功能

### 目标：
+ 实现基本功能条件下，符合资金安全规范标准
+ 实现弹性和灵活扩展的机制，能够随时根据需求进行横向或纵向的扩展
+ 酷狗直播S级业务，保证高性能、高可用，5个9的级别(99.999%)

## 仓库礼物基础设计
实现基本功能条件下，在数据层面上兼顾 性能、可扩展性

### 1.库存表
![/images/docImages/kucun.png](/images/docImages/kucun.png)
### 2.订单表
![/images/docImages/order.png](/images/docImages/order.png)

### 避免多发、少扣
从三个方面处理：从数据结构、编码、监控 多种方面，保障资金安全

#### 一、DB(数据结构)
1. 数据库关键字段 <font color='cyan'>**unsigned**</font>  ,避免扣穿
2. 订单流水表 <font color='cyan'>**记录操作数据**</font>，支持 <font color='cyan'>**平衡对账**</font>
3. 余额表增加更新时间与流水表一致，支持 <font color='cyan'>**期初期末对账**</font>
#### 二、编码
1. <font color='cyan'>**幂等**</font>，重复请求，返回一致结果
2. <font color='cyan'>**DB乐观锁**</font> ，避免并发扣错
3. <font color='cyan'>**预算SDK对接**</font> ，避免业务多发

![/images/docImages/designmodel.png](/images/docImages/designmodel.png)


> 扩展：数据库架构常见架构设计：
> 一、主备架构（Master-Slave Architecture）：
> 主备架构是指数据库系统中有一个主节点（Master）和一个或多个备节点（Slave），主节点负责处理所有的写入操作，而备节点负责复制主节点上的数据。当主节点出现故障时，备节点可以接管主节点的工作，以保证数据库的可用性
> 二、双主架构（Master-Master Architecture）：
> 双主架构是指数据库系统中有两个主节点，每个主节点都可以处理读写操作。当一个主节点出现故障时，另一个主节点可以接管其工作，以保证数据库的可用性
> 三、主从架构（Master-Slave Architecture）：
> 主从架构和主备架构类似，但是备节点可以被配置为只读节点，主节点处理所有的写入操作，而从节点负责处理读取操作。当主节点出现故障时，备节点可以接管主节点的工作，并成为新的主节点。
> 四、一致性解决方案（Consistency Solution）：
> 在分布式数据库系统中，一致性解决方案是指确保不同节点之间数据的一致性。常见的一致性解决方案包括 <font color='cyan'>**基于时间戳的复制**</font>、 <font color='cyan'>**基于多版本并发控制（MVCC）的复制**</font> 、<font color='cyan'>**基于Paxos协议的一致性算法**</font> 、<font color='cyan'>**基于Raft协议的一致性算法**</font> 等。这些算法都旨在保证不同节点之间数据的一致性和可靠性

### 数据库架构设计原则(扩展)
#### 一、数据库的范式化设计：
通过范式化的设计，可以减少数据冗余和数据不一致的问题。常见的范式包括第一范式（1NF）、第二范式（2NF）和第三范式（3NF）等
#### 二、数据库的反范式化设计：
有些情况下，反范式化的设计可以提高数据库的性能。反范式化的设计包括将数据冗余存储、增加冗余索引、分区表、分片等技术
#### 三、合理分配数据和索引：
合理的数据和索引分配可以提高数据库的查询性能。通常建议将索引分配到较小的表中，同时将数据均匀分配到各个节点中
#### 四、优化数据库查询：
通过对查询进行优化，可以提高数据库的查询性能。包括对查询进行优化，例如使用索引、避免全表扫描、使用优化的SQL语句等
#### 五、合理的分区和分片：
分区和分片是分布式数据库中常用的技术，可以提高数据库的可扩展性。合理的分区和分片可以减少网络开销、提高查询性能、降低数据库负载等
#### 六、安全性和可靠性：
在数据库架构设计中，安全性和可靠性是非常重要的。包括对数据库进行备份、数据加密、访问控制、防火墙等技术的应用。
#### 七、监控和调优：
在数据库架构设计中，监控和调优也是非常重要的。包括实时监控数据库的性能、调整数据库配置参数、调整应用程序、优化数据库查询等。

### 服务部署
部署架构：<font color='cyan'>**同城主备**</font> 模型
![/images/docImages/servicemodel.png](/images/docImages/servicemodel.png)

### 双机房缓存一致性问题
问题：备用机房有 <font color='cyan'>**1%的流量**</font> ，可能会导致双边机房缓存 <font color='cyan'>**数据不一致**</font>
方案：<font color='cyan'>**扣款逻辑读主库，业务操作成功后，双删两机房Redis**</font>

![/images/docImages/istio.png](/images/docImages/istio.png)

#### 三、监控

## 仓库礼物过期属性设计

### 仓库过期需求介绍
需求背景：大量僵尸账户仓库库存 <font color='cyan'>**堆积**</font>， <font color='cyan'>**金额庞大**</font>，对平台而言存在资金隐患

+ 增加礼物过期时间
+ 同一礼物聚合在一起
+ 显示最近过期的礼物数量
+ 优先扣除最近过期

### 支持过期属性面临的挑战
<font color='cyan'>**用户体量 大**</font>、<font color='cyan'>**礼物种类 多**</font>、且每个礼物需要支持 <font color='cyan'>**过期属性**</font>

![/images/docImages/til.png](/images/docImages/til.png)

技术挑战：<font color='cyan'>**存储压力 大**</font> 、 <font color='cyan'>**S级服务性能要求 高**</font>

![/images/docImages/zhibo.png](/images/docImages/zhibo.png)

### 过期属性方案选型
目的：性能优化
方案：聚合 同一个用户、同一礼物、按月分桶，过期时间以json存储

版本1：增加过期列
问题：如果用户大量送礼，可能导致大事务

![/images/docImages/v1.png](/images/docImages/v1.png)

版本2：聚合一个用户、同一礼物过期列、以json存储
问题：大字段存储、会影响到查询性能

![/images/docImages/v2.png](/images/docImages/v2.png)

版本3：聚合同一个用户、同一礼物、按月分桶，过期时间以json存储

![/images/docImages/v3.png](/images/docImages/v3.png)

### 性能测试
结论：可满足当前性能要求
<font color='cyan'>**限制过期时间为3个月，
出仓接口最大QPS: 2000+ ，
入仓：3000+，
查询最大支持：8000+**</font>

![/images/docImages/ychj.png](/images/docImages/ychj.png)

![/images/docImages/csjg.png](/images/docImages/csjg.png)



## 仓库礼物指定房间设计
### 指定房间送礼介绍
需求：农场产出的礼物，只能在 <font color='cyan'>**当前直播间**</font> 送出

目标：
1. 实现指定房间送礼
2. 隔离性：不影响现有繁星仓库业务
3. 扩展性：业务全面铺开后无性能压力


![/images/docImages/liwu.png](/images/docImages/liwu.png)

### 支持指定房间面临的挑战
存在 <font color='cyan'>**无限膨胀**</font> 的可能性
需要在原有的仓库的基础上增加 <font color='cyan'>**toKugouId字段**</font> ，以仓库目前的体量估算，对仓库db性能有 <font color='cyan'>**很大影响**</font> 。

![/images/docImages/wxpz.png](/images/docImages/wxpz.png)


### 解决性能问题
如何解决活动仓库数量级 无限膨胀 的问题？
 
#### 业务维度
1. 控制活动专属礼物种类
2. 限制过期时间不超过3个月
3. 限制只能绑定主播

#### 技术维度
1. 按kugouId分表
2. 后续按toKugouId分库
3. 独立服务部署，不影响现有仓库

![/images/docImages/fenku.png](/images/docImages/fenku.png)


### 整体架构设计
目的：不影响当前繁星仓库
方案：新增活动仓库服务，独立部署，送礼服务聚合数据
![/images/docImages/fenbiao.png](/images/docImages/fenbiao.png)

### 仓库列表设计
#### 问题：如何实现仓库列表的统计、明细功能
方案：空间换时间，增加 <font color='cyan'>**冗余统计表**</font>

需求：查询 kugouId = 1 和 toKugouId = 2 的仓库列表统计信息与明细

toKugouId可能会无限膨胀，如果按kugouId为条件查，可能会导致慢查询

方案：增加冗余表，避免慢查询出现
缺点：入仓 + 出仓 多一次写入
![/images/docImages/giftList.png](/images/docImages/giftList.png)

#### 问题：按kugouId查询仓库列表，如何实现 分页查询
方案：前端内存分页，<font color='cyan'>**一次性取3000条数据**</font>
![/images/docImages/fenye.png](/images/docImages/fenye.png)

![/images/docImages/sjjg.png](/images/docImages/sjjg.png)
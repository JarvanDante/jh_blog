---
author: "Karson"
title: "分布式事务框架 seata"
date: 2023-05-19
description: "Seata 开源的、高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- knowledge
categories:
- seata
---

## 理论基础
### CAP定理
1998年，加州大学，Eric Brewer提出，分布式系统有三个指标：
+ Consistency(一致性)
+ Availability(可用性)
+ Partition tolerance(分区容错性)

分布式系统无法同时满足这三个指标，这个结论就叫CAP定理。
![/images/docImages/cap1.png](/images/docImages/cap1.png)
![/images/docImages/cap2.png](/images/docImages/cap2.png)
> PS:分布式系统中一定会出现不避免的问题：<font color='cyan'>**分区问题（P）**</font>
> 当分区出现时，系统的 <font color='cyan'>**一致性（C）和 可用性（A）就无法同时满足，要么是CP，要么是AP**</font>
> ES集群是CP？还是AP？ ==> 是CP

### BASE理论
BASE理论是对CAP的一种解决思路，包含三个思想：
+ Basically Available(基本可用)：分布式系统在出现故障时，允许损失部分可用性，即保证核心可用
+ Soft State(软件状态)：在一定时间内，允许出现中间状态，比如临时的不一致状态
+ Eventually Consistent(最终一致性)：虽然无法保证强一致性，但是在软状态结束后，最终达到数据一致

解决方案：
+ AP模式：各子事务分别执行和提交，允许出现结果不一致，然后采用弥补措施恢复数据即可，实现<font color='cyan'>**最终一致**</font> 。
+ CP模式：各个子事务执行后互相等待，同时提交，同时回滚，达成 <font color='cyan'>**强一致**</font> 。但事务等待过程，处理弱可用状态。

#### <font color='cyan'>**"事务协调者" 非常重要**</font>
解决分布式事务的思想和模型 概念：
+ 全局事务：整个分布式事务
+ 分支事务：分布式事务中包含的每个子系统的事务
+ 最终一致思想：各分支事务分别执行并提交，如果有不一致的情况，再想办法恢复数据
+ 强一致思想：各分支事务执行完业务不要提交，等待彼此结果，而后统一提交或回滚

## Seata架构
Seata事务管理中有三个重要角色：
+ TC（Transaction Coordinator）- 事务协调者：<font color='cyan'>**维护全局和分支事务的状态**</font>，协调全局事务提交或回滚
+ TM（Transaction Manager）- 事务管理器：<font color='cyan'>**定义全局事务的范围**</font>、开始全局事务、提交或回滚全局事务
+ RM（Resource Manager ）- 资源管理器：管理分支事务处理的资源，<font color='cyan'>**与TC交谈以注册分支事务和报告分支事务的状态**</font>，并驱动分支事务提交或回滚

![/images/docImages/cap3.png](/images/docImages/cap3.png)

### Seata提供了四种事务模式：
四种不同的分布式事务解决方案：
+ XA模式：<font color='cyan'>**强一致性分阶段事务模式**</font>，牺牲了一定的可用性，<font color='cyan'>**无业务侵入**</font>
+ TCC模式：<font color='cyan'>**最终一致的分阶段事务模式，有业务侵入**</font>
+ AT模式：<font color='cyan'>**最终一致的分阶段事务模式，无业务侵入，也是Seata的默认模式**</font>
+ SAGA模式：<font color='cyan'>**长事务模式，有业务侵入**</font>

#### --XA模式--
XA规范 是X/Open 组织定义的分布式事务处理（DTP,Distributed Transaction Processing）标准，XA 规范 描述了全局的TM 与局部的RM 之间的接口，几乎所有主流的数据库都对XA规范 提供了支持。
![/images/docImages/cap4.png](/images/docImages/cap4.png)

#### seata的XA模式
RM一阶段的工作：
1. <font color='cyan'>**注册**</font>分支事务到 <font color='cyan'>**TC**</font>
2. 执行分支事务<font color='cyan'>**sql但不提交**</font>
3. <font color='cyan'>**报告状态**</font>到 <font color='cyan'>**TC**</font>

TC二阶段的工作：
TC检测各分支事务执行状态：
a. 如果都成功，通知所有RM提交事务
b. 如果有失败，通知所有RM回滚事务
RM二阶段的工作：
接口TC指令，提交或回滚事务
![/images/docImages/cap5.png](/images/docImages/cap5.png)

#### XA模式优/缺点
优点：
+ 事务强一致性，满足ACID原则
+ 没有代码侵入，实现简单
缺点：
+ 耗时，性能较差
+ 依赖关系型数据库

#### --AT模式--
优化了XA模型中锁定周期过长的问题

阶段一RM的工作：
1. <font color='cyan'>**注册**</font>分支事务
2. <font color='cyan'>**记录undo-log**</font>(数据快照)
3. 执行业务<font color='cyan'>**sql并提交**</font>
4. <font color='cyan'>**报告事务状态**</font>
阶段二提交时RM的工作：
a. 删除undo-log即可
阶段二回滚时RM的工作：
a. 根据undo-log恢复数据到更新前

![/images/docImages/cap6.png](/images/docImages/cap6.png)


##### AT模式与XA模式的 <font color='cyan'>**区别**</font>：
1. XA模式不提交；AT模式提交
2. XA模式利用数据库机制实现回滚；AT模式利用数据快照实现回滚
3. XA模式强一致；AT模式最终一致

#### AT模式的脏写问题
全局锁：由TC记录当前正在操作某行数据的事务，该事务持有全局锁，具备执行权。 xid/table/pk
全局锁与DB锁互相等待可能死锁

#### AT模式优/缺点
优点：
1. 一阶段完成直接提交事务，释放数据库资源，性能比较好
2. 全局锁实现读写隔离
3. 无代码侵入，框架自动完成回滚和提交
缺点：
+ 最终一致性，两阶段之间软状态
+ 快照功能会影响性能，但比XA模式要好很多

#### --TCC模式(强性能)--
TCC模式与AT模式非常相似，每阶段都是独立事务，不同的是TCC通过人工编码实现数据恢复。需要实现三个方法：
+ Try：资源的检测和预留；
+ Confirm：完成资源操作业务；要求Try 成功 Confirm 一定要能成功。
+ Cancel：预留资源释放，可以理解为try的反向操作


![/images/docImages/cap8.png](/images/docImages/cap8.png)

![/images/docImages/cap7.png](/images/docImages/cap7.png)

#### TCC模式优/缺点
优点：
+ <font color='cyan'>**性能好**</font>，一阶段完成直接提交事务，释放数据库资源
+ <font color='cyan'>**无快照，无全局锁**</font>，性能最强
+ 不依赖数据库事务，而是依赖补偿操作，<font color='cyan'>**可以用于非事务型数据库**</font>

缺点：
+ <font color='cyan'>**代码侵入**</font>，要人为编写try/confirm/cancel接口，太麻烦
+ 软状态，事务是<font color='cyan'>**最终一致**</font>
+ 要考虑confirm和cancel的失败情况，做好 <font color='cyan'>**幂等性**</font> 处理

#### 空回滚
当某分支事务的try阶段 <font color='cyan'>**阻塞**</font> 时，可能导致全局事务超时而触发二阶段的cancel操作。在未执行try操作时先执行了cancel操作，这时cancel不能做回滚，就是 <font color='cyan'>**空回滚**</font>。


#### 业务悬挂
对于已经空回滚的业务，如果以后继续执行try，就<font color='cyan'>**永远不可能**</font> confirm 或 cancel ，这就是 <font color='cyan'>**业务悬挂**</font> 。


![/images/docImages/cap9.png](/images/docImages/cap9.png)

设计一张记录事务状态的表：
```sql
CREATE TABLE `account_freeze_tbl` (
  `xid` varchar(255) NOT NULL COMMENT '事务id',
  `user_id` varchar(255) DEFAULT NULL COMMENT '用户id',
  `freeze_money` int unsigned DEFAULT '0' COMMENT '冻结金额',
  `state` tinyint(1) DEFAULT NULL COMMENT '事务状态0:try1:confirm2:cancel',
  PRIMARY KEY (`xid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3 COMMENT='冻结金额事务表';
```

#### TCC实现步骤：
##### Try业务：
+ 记录冻结金额和事务状态到 account_freeze 表
+ 扣减account表可用金额

##### Confirm业务：
+ 根据xid删除account_freeze表的冻结记录

##### Cancel业务：
+ 修改account_freeze表，冻结金额为0，state为2
+ 修改account表，恢复可用金额

##### 如何判断是否空回滚：
+ cancel业务中，根据xid查询account_freeze，<font color='cyan'>**如果为null则说明try还没做，需要空回滚**</font>

##### 如何避免业务悬挂：
+ try业务中，根据xid查询account_freeze，<font color='cyan'>**如果已经存在则证明cancel已经执行，则拒绝执行try业务**</font>

#### --Saga模式--
saga柜式是Seata 提供的长事务解决方案：
分两个阶段：
1. 一阶段：直接提交本地事务
2. 二阶段：成功则什么都不做；失败则通过编写补偿业务来回滚

saga模式优点：
+ 事务参与者异步调用，高吞吐
+ 一阶段直接提交事务，无锁，性能好
+ 不用编写TCC中的三个阶段，实现简单

缺点：
+ 时效性差，软状态持续时间不确实
+ 没有锁，没有事务隔离，会有脏写

### 四种模式对比
![/images/docImages/cap10.png](/images/docImages/cap10.png)

|      | XA              | AT                  | TCC                       | SAGA                                          |
|------|-----------------|---------------------|---------------------------|-----------------------------------------------|
| 一致性  | 强一致             | 弱一致                 | 弱一致                       | 最终一致                                          |
| 隔离性  | 完全隔离            | 基于全局锁隔离             | 基于资源预留隔离                  | 无隔离                                           |
| 代码侵入 | 无               | 无                   | 有，要编写三个接口                 | 有，要编写状态机和补偿业务                                 |
| 性能   | 差               | 好                   | 非常好                       | 非常好                                           |
| 场景   | 对一致性、隔离性有高要求的业务 | 基于关系型数据库的大多数分布式事务场景 | 对性能要求较高的事务、有非关系型数据库要参与的事务 | 业务流程长、业务流程多、参与者包含其它公司或遗留系统服务，无法提供TCC模式要求的三个接口 |


## TC服务的高可用和异地容灾
TC服务作为Seata的核心服务，一定要保证 <font color='cyan'>**高可用 和 异地容灾**</font>。
TC的异地多机房容灾架构：
![/images/docImages/cap11.png](/images/docImages/cap11.png)
---
author: "Karson"
title: "微服务架构之幂等性问题"
date: 2022-05-17
description: "微服务架构之幂等性问题"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- knowledge
categories:
- kafka
---

## 前言
小伙伴们有没有遇到过生产环境经常出现过重复的数据？在排查问题的时候，数据又是正常的。这个是何解呢？怎么会出现这种情况，而且还很难排查问题。今天我给大家分享一下这里的原因，以及解决方案。

产生重 <font color='cyan'>**复数据或数据不一致**</font> （假定程序业务代码没问题），绝大部分就是发生了重复的请求，
<font color='cyan'>**重复请求**</font> 是指同一个请求因为某些原因被多次提交。
导致这个情况会有几种场景：
1. 微服务场景:
在我们传统应用架构中调用接口，要么成功，要么失败。但是在微服务架构下，会有 <font color='cyan'>**第三个情况【未知】**</font> ，也就是超时。如果<font color='cyan'>**超时**</font> 了，微服务框架会进行 <font color='cyan'>**重试**</font>。
2. 用户交互的时候多次点击:
如：快速点击按钮多次
3. MQ消息中间件:
消息重复消费
4. 第三方平台的接口:
（如：支付成功回调接口），因为异常也会导致多次异步回调
5. 其他中间件/应用服务根据自身的特性，也有可能进行重试



我们知道了发生的原因，本质就是多次请求了，那如何解决呢？

## 幂等性

定义：<font color='cyan'>**多次调用对系统的产生的影响是一样的**</font> ，即对资源的作用是一样的，但是返回值允许不同

## 幂等场景

我们来看一下SQL相关业务是否幂等？

### 一、查询
`select * from user where xxx`，不会对数据产生任何变化，<font color='cyan'>**具备幂等性**</font>。

### 二、新增
`insert into user(userid,name) values(1,'a')`
如 userid为**唯一主键**，即重复操作上面的业务，只会插入一条用户数据，<font color='cyan'>**具备幂等性**</font> 。
如 userid**不是主键**，可以重复，那上面业务多次操作，数据都会新增多条，<font color='cyan'>**不具备幂等性**</font> 。

### 三、修改
区分直接赋值和计算赋值
1. 直接赋值，`update user set point = 20 where userid=1`，不管执行多少次，point都一样，<font color='cyan'>**具备幂等性**</font>
2. 计算赋值，`update user set point = point + 20 where userid=1`，每次操作point数据都不一样，<font color='cyan'>**不具备幂等性**</font> 

### 四、删除
`delete from user where userid=1`，多次操作，结果一样，<font color='cyan'>**具备幂等性**</font>  。

常用的方案：<font color='cyan'>**token机制、乐观锁机制、唯一主键机制、去重表机制**</font>

## token机制

token方式的流程，上一张图，比较清晰：
![/images/docImages/tk1.png](/images/docImages/tk1.png)

上图就是 <font color='cyan'>**token + redis的幂等方案**</font>  ，适用绝大部分场景。
主要思想：
1. <font color='cyan'>**服务端提供token的接口**</font>。我们在分析业务的时候，哪些业务是存在幂等问题的，就必须在执行业务前，先去获取token，服务器会把token保存到redis中。（微服务肯定是分布式了，如果单机就适用jvm缓存）。
2. 然后调用业务接口请求时，<font color='cyan'>**携带token过去**</font>  ，一般放在请求头部。
3. 服务器判断token是否存在<font color='cyan'>**redis中**</font> ，<font color='cyan'>**存在表示第一次请求**</font> ，可以继续执行业务，执行业务完成后，最后需要把 <font color='cyan'>**redis中的token删除**</font>。
4. 如果判断token <font color='cyan'>**不存在redis中**</font> ，就<font color='cyan'>**表示是重复操作**</font> ，直接 <font color='cyan'>**返回重复标记给client**</font> ，这样就保证了业务代码，不被重复执行。

这种方案是比较常用的方案，也是网上经常介绍的，但是有一点不同的地方：
+ 网上方案：检验token存在（表示第一次请求）后，就立刻删除token，再进行业务处理
+ 上面方案：检验token存在（表示第一次请求）后，先进行业务处理，再删除token

关键点就是 <font color='cyan'>**先删除token，还是后删除token**</font> 。

### 网上方案缺点:
先删除token，这是出现系统问题导致业务处理出现异常，业务处理没有成功，接口调用方也没有获取到明确的结果，然后进行重试，但token已经删除掉了，服务端判断token不存在，<font color='cyan'>**认为是重复请求**</font> ，就直接返回了，无法进行业务处理了

### 上面方案缺点:
后删除token 也是会存在问题的，如果进行业务处理成功后，<font color='cyan'>**删除redis中的token失败**</font>，这样就导致了有 <font color='cyan'>**可能会发生重复请求**</font>，因为token没有被删除

其实上面的问题就是数据库和缓存redis数据不一致的问题

总结，我认为 <font color='cyan'>**网上方案先删除token，先保证不会因为重复请求，业务数据出现问题。顶多再让用户处理一次**</font>
出现 <font color='cyan'>**业务异常**</font>，可以让调用方配合处理一下，<font color='cyan'>**重新获取新的token**</font>，再次由业务调用方发起重试请求就ok了

### token机制缺点
业务请求每次请求，都会有额外的请求（一次获取token请求、判断token是否存在的业务）。其实真实的生产环境中，1万请求也许只会存在10个左右的请求会发生重试，为了这10个请求，我们让9990个请求都发生了额外的请求。（当然redis性能很好，耗时不会太明显）


## 乐观锁机制
乐观锁这里解决了 <font color='cyan'>**计算赋值型**</font> 的修改场景，
我们对之前的sql语句进行修改
`update user set point = point + 20, version = version + 1 where userid=1 and version=1`
<font color='cyan'>**加上了版本号**</font> 后，就让此计算赋值型业务，<font color='cyan'>**具备幂等性**</font> 。

### 乐观锁机制缺点
就是在操作业务前，需要先查询出当前的version版本

## 唯一主键机制
这个机制是利用了数据库的 <font color='cyan'>**主键唯一约束**</font> 的特性，解决了在insert场景时幂等问题。
但主键的要求不是自增的主键，这样就需要业务生成全局唯一的主键，必须要处理 <font color='cyan'>**分布式唯一主键ID**</font> 的生成，
如果是分库分表场景下，路由规则要保证相同请求下，落地在同一个数据库和同一表中，要不然数据库主键约束就不起效果了，因为是不同的数据库和表主键不相关。
###  唯一主键机制缺点
因为对主键有一定的要求，这个方案就跟业务有点耦合了，无法用自增主键了。

## 去重表机制
这个方案业务中要有 <font color='cyan'>**唯一主键**</font>，这个 <font color='cyan'>**去重表中只要一个字段**</font> 就行，设置唯一主键约束，当然根据业务自行添加其他字段。
主要流程上图
![/images/docImages/maq1.png](/images/docImages/maq1.png)

上面的主要流程就是 <font color='cyan'>**把唯一主键插入去重表，再进行业务操作，且他们在同一个事务中**</font>。
这个保证了重复请求时，因为去重表有唯一约束，导致请求失败，避免了幂等问题

注意：
这里要注意的是，去重表和业务表应该在 <font color='cyan'>**同一库中**</font> ，这样就保证了在同一个事务，即使业务操作失败了，也会把去重表的数据回滚。这个很好的保证了数据一致性。

这个方案也是比较常用的，<font color='cyan'>**去重表是跟业务无关的**</font>，很多业务可以共用同一个去重表，只要规划好唯一主键就行了

## 总结
上面介绍了一些幂等方案，小伙伴们根据自身的业务进行选择，尽量不要让系统变的复杂，所以推荐 <font color='cyan'>**唯一主键**</font> 和 <font color='cyan'>**乐观锁方式**</font> ，因为实现比较简单
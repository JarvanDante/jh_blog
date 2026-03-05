---
author: "karson"
title: "CentOS24年停更后平替系统"
date: 2022-01-05 00:00:01
description: "CentOS 停止技术支持后，我们应该如何选择适合的操作系统?"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags: 
- linux
- categories:

---
CentOS Linux 将于 Jun 30th, 2024 EOF(结束生命周期). 届时, 我们的 OS 还有哪些选择?
+ <font style="color:red">不要选择 CentOS Stream ! </font>
+ <font style="color:red">不要选择 CentOS Stream ! </font>
+ <font style="color:red">不要选择 CentOS Stream ! </font>
## Linux生态
红帽企业级 Linux 生态系统形成了 Fedora、RHEL 和 CentOS Linux 协同发展的局面
1. 上游：通过 Fedora 向广大开发工程师`提供桌面操作系统`的持续创新和技术架构整合，大约是每 6 个月发布一个版本
2. 中游：是红帽企业级 Linux，主要面向广大企业和应用开发商，特点是`稳定、安全和性能优化`
3. 下游：是社区领导的 CentOS Linux，面向成本敏感用户和生态开发者，特点是`无成本、易获取`，大约在红帽企业 Linux 发布的几个月后发布

## CentOS Stream
CentOS Stream 本身介于 Fedora 和 RHEL 之间，离 RHEL 更近，相当于 RHEL 上开发的所有功能都已经在 CentOS Stream 具备，该版本同样对所有人免费开放，可保证开发者提前获得 RHEL 新特性，在此基础上来做诸如开发第三方组件等工作，拓展他们对于 RHEL 生态的影响。相当于 CentOS Stream 是 RHEL 的试验田。2019 年 9 月，Red Hat 宣布了 CentOS Stream，它是 CentOS 的滚动发行版本，介于 Fedora Linux 的上游开发和 RHEL 的下游开发之间而存在，当官方明确表示未来不会再发布由 RHEL 代码编译而成的 CentOS 后，意味着 CentOS Stream 先行，稳定之后再发布 RHEL，所以不难理解众多开发者对这个决策的不满。

`PS:`
而且现在 CentOS 只提供大版本, 如 CentOS Stream 8 和 9, 没有之前的 7.X 这样的小版本.
虽然 RedHat 官方保证 CentOS Stream 足够稳定, 但是根据笔者和其他相关人士的沟通, 均不再选择其作为生产环境 OS.

## CentOS 替代品
### 1.RHEL Linux
RHEL 大家都熟悉, 就不做简介了. 不考虑钱的因素, RHEL Linux 是最完美的替代品

<font style="color:cyan">**特点：**
1. 不差钱
2. 对稳定性要求极高
3. 需要购买专业Linux维护服务
4. 金融行业
</font>

### 2.Rocky Linux
Rocky Linux 直接从 RHEL® 重建源代码，无论用例如何，您都将拥有超级稳定的体验。
使用 Rocky Linux 替代 CentOS, 代价也是最小的. 但是 Rocky Linux 的稳定性还需要经过更长时间的检验.

<font style="color:cyan">**特点：**
1. 开源
2. 期望寻找 CentOS 的平替
3. 熟悉 Fedora RHEL CentOS 生态
</font>

### 3.公有云Linux
如果您的全部或大部分资源都在公有云上托管, 那么还有一个可行的方案是选择: 公有云提供的 Linux. 如:
+ Amazon Linux
+ Alibaba Cloud Linux

<font style="color:cyan">**特点：**
1. 公有云用户
</font>

### 4.Debian系Linux
以上提到的几种选择, 都是基于 Fedora 这一系的 Linux 衍生而来. 切换代价也相对较小.
但是如果用户相比稳定性, 更追求创新, 追求更新的内核, 更新的功能, 那么也可以选择切换到 Debian 系 Linux, 推荐的选择有: Debian 和 Ubuntu

当然, 要切换到 Dbian 系 Linux, 代价还是相对较大的:
+ 包管理软件会从 yum/dnf 切换到 apt/dpkg
+ 大多数人的观点是 Debian 系统不像 RHEL/CentOS 那样稳定或无故障
+ Debian 的内核/软件相对更新

<font style="color:cyan">**特点：**
1. 相比稳定性, 更追求创新
2. 熟悉 Debian 生态
3. 已经较多使用容器/K8s (因为 Debian 在容器生态中更常见)
4. 需要使用较新内核或较新的功能, 如 eBPF 和 Cilium
</font>

### 5.信创系Linux(美脱钩)
如果您主要面向的是`国内的政企/金融`客户, 有信创的刚需, 那么推荐您切换到信创系 Linux, 主流的包括:  `华为主导的 openeuler` 和 `阿里主导的 anolis(龙蜥) os`.

这两款也是基于 Fedora 的生态系统, 其官网也往往会有用于一键迁移 OS 的实用工具. 但是软件生态还需要更多时间来培养成熟.

<font style="color:cyan">**特点：**
1. 信创需求
2. 面向政企/金融客户
</font>

## 总结
1. 不要选择 CentOS Stream;
2. RHEL Linux, 适合金融客户;
3. Rocky Linux, 适合需要开源免费且平替 CentOS Linux的客户;
4. 公有云 Linux, 适合资源都在公有云上的客户;
5. 信创 Linux, 适合面向国内政企/金融, 有信创需求的客户.

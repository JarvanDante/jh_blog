---
author: "jarvan"
title: "StarRocks使用"
date: 2023-06-25
description: "高性能分析型数据仓库，使用向量化、MPP 架构、可实时更新的列式存储引擎等技术实现多维、实时、高并发的数据分析"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- db
- categories:

---

## 什么是StarRocks
>官网地址 https://docs.starrocks.io/zh/docs/introduction/

StarRocks 是一款高性能分析型数据仓库，使用向量化、MPP 架构、可实时更新的列式存储引擎等技术实现多维、实时、高并发的数据分析。

## 适用场景
StarRocks 可以满足企业级用户的多种分析需求，包括 OLAP 多维分析、定制报表、实时数据分析和 Ad-hoc 数据分析等。

### OLAP 多维分析：

+ 用户画像、标签分析、圈人
+ 用户行为分析
+ 财务报表
+ 业务问题探查分析
+ 高维业务指标报表
+ 自助式报表平台
+ 系统监控分析
+ 跨主题业务分析 

### 实时数据仓库：
+ 电商大促数据分析
+ 物流行业的运单分析
+ 金融行业绩效分析、指标计算
+ 直播质量分析
+ 广告投放分析
+ 管理驾驶舱
+ 探针分析APM

### 高并发查询：
+ 广告主报表分析
+ 零售行业渠道人员分析
+ SaaS 行业面向用户分析报表
+ Dashbroad 多页面分析

### 统一分析：
+ 通过使用一套系统解决多维分析、高并发查询、预计算、实时分析查询等场景，降低系统复杂度和多技术栈开发与维护成本。
+ 使用StarRocks 来统一数据湖和数据仓库，将高并发和实时要求性很高的业务放在StarRocks中分析，把数据湖上的分析使用StarRocks外表查询，统一使用 StarRocks 管理湖仓数据。

### 系统架构
StarRocks的架构简洁，整个系统的核心只有FE（Frontend）、BE（Backend）两类进程，不依赖任何外部组件，方便部署与维护。同时，FE和BE模块都可以在线水平扩展，元数据和数据都有副本机制，确保整个系统无单点。

![/starrocks.png](/images/docImages/starrocks.png)

<font color="lightgreen">FE（Frontend）</font>是StarRocks的前端节点，<font color="lightgreen">负责管理元数据，管理客户端连接，进行查询规划，查询调度</font>等工作。FE根据配置会有两种角色：**Follower** 和 **Observer**。

+ Follower 会通过类Paxos的BDBJE协议选主出一个Leader（实现选主需要集群中有半数以上的Follower实例存活），只有Leader会对元数据进行写操作。非Leader节点会自动的将元数据写入请求路由到Leader节点。每次元数据写入时，必须有多数Follower成功才能确认是写入成功。
+ Observer 不参与选主操作，只会异步同步并且回放日志，主要用于扩展集群的查询并发能力。每个FE节点都会在内存保留一份完整的元数据，这样每个FE节点都能够提供无差别的服务。


<font color="lightgreen">BE（Backend）</font> 是StarRocks的后端节点，负责数据存储以及SQL执行等工作。
数据存储方面，StarRocks的BE节点都是完全对等的，FE按照一定策略将数据分配到对应的BE节点。在数据导入时，数据会直接写入到BE节点，不会通过FE中转，BE负责将导入数据写成对应的格式以及生成相关索引。

在执行SQL计算时，一条SQL语句首先会按照具体的语义规划成逻辑执行单元，然后再按照数据的分布情况拆分成具体的物理执行单元。物理执行单元会在数据存储的节点上进行执行，这样可以避免数据的传输与拷贝，从而能够得到极致的查询性能。

StarRocks整体对外暴露的是一个 <font color="lightgreen">MySQL协议接口</font> ，支持标准SQL语法。用户通过已有的MySQL客户端能够方便地对StarRocks里的数据进行查询和分析。

### 数据管理
StarRocks 使用 <font color="lightgreen">列式存储</font> ，采用<font color="lightgreen">分区分桶</font>机制进行数据管理。一张表可以被划分成多个分区，一个分区内的数据可以根据一列或者多列进行分桶，将数据切分成多个 Tablet。Tablet 是 StarRocks 中最小的数据管理单元。每个 Tablet 都会以多副本 (replica) 的形式存储在不同的 BE 节点中。用户可以自行指定 Tablet 的个数和大小，StarRocks 会管理好每个 Tablet 副本的分布信息。

下图展示了 StarRocks 的数据划分以及 Tablet 多副本机制。表按照日期划分为 4 个分区，第一个分区进一步切分成 4 个 Tablet。每个 Tablet 使用 3 副本进行备份，分布在 3 个不同的 BE 节点上。

![/table.png](/images/docImages/table.png)

在执行 SQL 语句时，StarRocks 可以对所有 Tablet 实现并发处理，从而充分利用<font color="lightgreen">多机、多核</font>提供的计算能力。用户也可以将高并发请求压力分摊到多个物理节点，通过增加物理节点的方式来扩展系统的高并发能力。

StarRocks 支持 Tablet 多副本存储（默认三个），多副本能够保证数据存储的高可靠以及服务的高可用。在三副本下，一个节点的异常不会影响服务的可用性，集群的读写服务仍然能够正常进行。增加副本数还有助于提高系统的高并发查询能力。

在 BE 节点数量发生变化时 （比如扩缩容时），StarRocks 可以自动完成节点的增减，无需停止服务。节点变化会触发 Tablet 的自动迁移。当节点增加时，一部分 Tablet 会自动均衡到新增的节点，保证数据能够在集群内分布的更加均衡；当节点减少时，待下线机器上的 Tablet 会被自动均衡到其他节点，从而自动保证数据的副本数不变。管理员能够非常容易的实现弹性伸缩，无需手工进行数据的重分布。

<font color="lightgreen">存算一体架构</font> 的优势在于极速的查询性能，但也存在一些局限性：
+ 成本高：需要使用三副本保证数据可靠性；随着用户存储数据量的增加，需要不断扩容存储资源，导致计算资源浪费。
+ 架构复杂：存算一体架构需要维护多数据副本的一致性，增加了系统的复杂度。
+ 弹性不够：存算一体模式下，扩缩容会触发数据重新平衡，弹性体验不佳。

### 数据模型
数据模型分为四类：明细模型（Duplicate Key）、聚合模型（Aggregate Key）、更新模型（Unique Key）、主键模型（Primary Key）
#### 明细模型（Duplicate Key）
关键字 DUPLICATE KEY
```sql
CREATE TABLE IF NOT EXISTS table1 (
    event_time DATETIME NOT NULL COMMENT "datetime of event",
    event_type INT NOT NULL COMMENT "type of event",
    user_id INT COMMENT "id of user",
    device_code INT COMMENT "device of ",
    channel INT COMMENT ""
)
DUPLICATE KEY(event_time, event_type)
DISTRIBUTED BY HASH(user_id) BUCKETS 8
```
#### 聚合模型
关键字 AGGREGATE KEY （默认全表排序，一般可省略）
```sql
CREATE TABLE IF NOT EXISTS table2 (
    site_id LARGEINT NOT NULL COMMENT "id of site",
    date DATE NOT NULL COMMENT "time of event",
    city_code VARCHAR(20) COMMENT "city_code of user",
    pv BIGINT SUM DEFAULT "0" COMMENT "total page views"
)
DISTRIBUTED BY HASH(site_id) BUCKETS 8;
```

#### 更新模型
关键字 UNIQUE KEY
```sql
CREATE TABLE IF NOT EXISTS table3 (
    create_time DATE NOT NULL COMMENT "create time of an order",
    order_id BIGINT NOT NULL COMMENT "id of an order",
    order_state INT COMMENT "state of an order",
    total_price BIGINT COMMENT "price of an order"
)
UNIQUE KEY(create_time, order_id)
DISTRIBUTED BY HASH(order_id) BUCKETS 8
```
StarRocks存储内部会给每一个批次导入数据分配一个版本号, 同一主键的数据可能有多个版本, 查询时最大(最新)版本的数据胜出。 

#### 主键模型 
目前primary主键存储在内存中，为防止滥用造成内存占满，限制主键字段长度全部加起来编码后不能超过127字节。

主要适用场景有：

1.实时对接 TP 数据至 StarRocks。

2.利用部分列更新轻松实现多流 JOIN

关键字 PRIMARY KEY
```sql
create table table4 (
    dt date NOT NULL,
    order_id bigint NOT NULL,
    user_id int NOT NULL,
    merchant_id int NOT NULL,
    good_id int NOT NULL,
    good_name string NOT NULL,
    price int NOT NULL,
    cnt int NOT NULL,
    revenue int NOT NULL,
    state tinyint NOT NULL
) PRIMARY KEY (dt, order_id)
PARTITION BY RANGE(`dt`) (
    PARTITION p20210820 VALUES [('2021-08-20'), ('2021-08-21')),
    PARTITION p20210821 VALUES [('2021-08-21'), ('2021-08-22')),
    ...
    PARTITION p20210929 VALUES [('2021-09-29'), ('2021-09-30')),
    PARTITION p20210930 VALUES [('2021-09-30'), ('2021-10-01'))
) DISTRIBUTED BY HASH(order_id) BUCKETS 4
PROPERTIES("replication_num" = "3");
```
>注意:
> 1.主键列仅支持类型: boolean, tinyint, smallint, int, bigint, largeint, string/varchar, date, datetime, 不允许NULL。
> 2.分区列(partition)、分桶列(bucket)必须在主键列中。
> 3.和更新模型不同，主键模型允许为非主键列创建bitmap等索引，注意需要建表时指定。
> 4.由于其列值可能会更新，主键模型目前还不支持rollup index和物化视图。
> 5.暂不支持使用ALTER TABLE修改列类型。 ALTER TABLE的相关语法说明和示例，请参见 ALTER TABLE。
> 6.在设计表时应尽量减少主键的列数和大小以节约内存，建议使用int/bigint等占用空间少的类型。暂时不建议使用varchar。建议提前根据表的行数和主键列类型来预估内存使用量，避免出现OOM。内存估算举例：
a. 假设表的主键为: dt date (4byte), id bigint(8byte) = 12byte
b. 假设热数据有1000W行, 存储3副本
c. 则内存占用: (12 + 9(每行固定开销) ) * 1000W * 3 * 1.5(hash表平均额外开销) = 945M



### 建表语句
```sql
CREATE TABLE kg_hr_ods.test
(
    `ds`            date           NULL COMMENT "etl时间",
    `business_unit` varchar(65533) NULL COMMENT "业务单位"
) ENGINE = OLAP DUPLICATE KEY(`ds`)
COMMENT "员工信息表"
PARTITION BY RANGE(`ds`)(
    PARTITION p20231130 VALUES LESS THAN ("2023-12-01"),
    PARTITION p20231201 VALUES LESS THAN ("2023-12-02"),
    PARTITION p20231202 VALUES LESS THAN ("2023-12-03"),
    PARTITION p20231203 VALUES LESS THAN ("2023-12-04")
)
DISTRIBUTED BY HASH(`ds`) BUCKETS 24
PROPERTIES (
"replication_num" = "3"
);
```

#### 创建分区
目前 StarRocks 只支持 range 分区，以下面的例子来介绍分区功能：
```sql
CREATE TABLE kg_hr_ods.test
(
    `ds`            date           NULL COMMENT "etl时间",
    `business_unit` varchar(65533) NULL COMMENT "业务单位"
) ENGINE = OLAP DUPLICATE KEY(`ds`)
COMMENT "员工信息表"
PARTITION BY RANGE(`ds`)(
    PARTITION p20231130 VALUES LESS THAN ("2023-12-01"),
    PARTITION p20231201 VALUES LESS THAN ("2023-12-02"),
    PARTITION p20231202 VALUES LESS THAN ("2023-12-03"),
    PARTITION p20231203 VALUES LESS THAN ("2023-12-04")
)
DISTRIBUTED BY HASH(`ds`) BUCKETS 24;
```
#### 批量创建分区
如下例，通过指定 <font color="lightgreen">START END EVERY</font> 语句可以自动创建分区。其中，START 的值将被 <font color="lightgreen">包括在内</font>，而 END 的值会被 <font color="cyan">排除在外</font>。
```sql
CREATE TABLE kg_hr_ods.test
(
    `ds`            date           NULL COMMENT "etl时间",
    `business_unit` varchar(65533) NULL COMMENT "业务单位"
) ENGINE = OLAP DUPLICATE KEY(`ds`)
COMMENT "员工信息表"
PARTITION BY RANGE(`ds`)(
    START ("2023-01-01") END ("2023-02-01") EVERY (INTERVAL 1 DAY)
)
DISTRIBUTED BY HASH(`ds`) BUCKETS 24;
```
查看所有分区：
```sql
show partitions from kg_hr_ods.test;
```
#### 管理分区
添加分区
```sql
ALTER TABLE kg_hr_ods.test ADD PARTITION p20230209 VALUES LESS THAN ("2023-02¬10")
```
删除分区
```sql
ALTER TABLE kg_hr_ods.test drop PARTITION p20230101;
```
修改分区
```sql
ALTER TABLE site_access SET("dynamic_partition.enable"="false");
ALTER TABLE site_access SET("dynamic_partition.enable"="true");
```
查看分区信息
```sql
show partitions from kg_hr_ods.test;
```


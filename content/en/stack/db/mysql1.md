---
author: "Karson"
title: "mysql调优"
date: 2022-08-25
description: "索引、索引优化、SQL优化"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- db
categories:
- mysql

---

## Mysql索引
### 索引定义 
索引 ：在<font color='cyan'>关系型数据库</font>中的一种 <font color='cyan'>数据结构</font>，把数据 <font color='cyan'>提前规则排序和组织</font>，有助于 <font color='cyan'>快速定位</font> 数据记录的位置，加快数据库中数据的 <font color='cyan'>查询和访问速度</font>。
>PS :以空间换时间的思想

### 索引种类
#### 按数据结构分类：
+ B+tree 索引：
+ Hash 索引：
+ Full-text 索引：

#### 按物理存储分类：
+ 聚集索引：
+ 非聚集索引：

#### 按字段特性分类：
+ 主键索引(Primary)：
+ 唯一索引(Unique)：
+ 普通索引(Index)：
+ 全文索引(Full-text)：

#### 按字段数量分类：
+ 单列索引：
+ 联合/组合索引：

### 索引的数据结构和区别
常见的数据结构：二叉树、红黑树、B树、B+树
区别：树的高度影响数据获取的性能（每个树节点都是I/O），越矮性能越高
#### 二叉树
特点：<font color='cyan'>每个节点最多有两个子节点，小在左，大在右</font>，数据随机的情况下树叉越明显 
场景：按顺序排的极端的情况下，数据会是一个链表，查最大的数时会扫描全表
> mysql不支持，因为性能很低



#### 红黑树(平衡二叉树)
特点：在二叉树的基础上，加入 <font color='cyan'>自旋平衡</font> 的优化，每当数据插入时会自动分配树的结构，达到平衡，提高检索效率
树的高度还是很高的
#### B树(多叉树)
特点：
1. 父节点可以存储<font color='cyan'>多个元素</font>
2. 父节点可以有<font color='cyan'>多个节点</font> 
3. <font color='cyan'>非叶子节点存数据和指针，叶子节点只存储数据</font>
   
缺点：不支持范围查询
>PS: 一个节点如果是3阶存2个元素
> mysql支持16阶

#### B+树(B树升级)
特点：
1. <font color='cyan'>叶子节点存所有数据</font>
2. <font color='cyan'>非叶子节点不存数据</font>，只存询址节点
3. <font color='cyan'>叶子节点上有双向指针</font>，解决范围查询


### Hash 索引
对字段通过Hash计算，得到的值存储为索引。
优点：'等于'类的值查询，效率非常高
缺点：不支持范围查询
>ps: 可能会冲突


### 聚集/非聚集索引
按存储方面分：
InnoDB是聚集索引：
`.ibd` : InnoDB data 表索引 + 数据
MyISAM是非聚集索引：
`.MYD` : MyISAM data 表索引 + 数据
`.MYI` : MyISAM index 表索引 + 数据


### 二级索引
二级索引：所有 <font color='cyan'>非主键索引</font> ，都叫 <font color='cyan'>二级索引</font>

### 回表
回表：查询条件中没用到主键索引，因此第一次查询确定主键索引id范围，之后再回到数据表中根据主键id索引查询的过程

### 覆盖索引
覆盖索引：<font color='cyan'>查询的 字段 都在索引列中</font>
所以不要用 `select *` 的操作

### 索引下推
条件：
+ 二级索引
+ 范围查询
+ 减少回表

针对二级索引的优化，用来在 <font color='cyan'>范围查询时减少回表次数</font> 的

### 单列/联合索引
#### 联合索引
最左优先原则：一定要按联合索引的从左到右的组合方式组合，如：(a),(a,b)，(a,b,c)
#### 联合索引优势：
1. 减少开销 ：联合索引优于多个单列索引
2. 覆盖索引：提高性能，减少回表
3. 效率高：

## 索引优化
### 联合索引失效场景：
<font color='cyan'>"左、右、函数、范围后、select */!=/not in/单引号"</font>
1. 最左前缀法则：<font color='cyan'>缺少最左索引，跳过中间</font>
2. 在索引列上操作（<font color='cyan'>计算、函数、or</font>）：函数 打断了索引的有序 固失效
3. 索引字段 <font color='cyan'>范围之后全失效</font> `where name = tom and age >10 and score=100`
4. 使用覆盖索引，不使用 `select *`
5. 使用不等于`!=`、`not in`索引失效
6. like% <font color='cyan'>百分写最右</font>
7. 字体符号不加单引号


>PS:有possiblity_key强制使用索引 `FORCE INDEX( XXX )`

## trace工具
步骤：
1. 开启trace: `set session optimizer_trace="enabled_on",end_markers_in_json=on;`
2. `select * from user where age=18 and name='张三';`
3. `SELECT * FROM information_schema.OPTIMIZER_TRACE;`

## SQL优化
`select` `from` `left` `group by` `batch` `limit`
1. <font color='cyan'>不用`select *`</font>:不会走覆盖索引
2. <font color='cyan'>小表驱动大表</font>： `from 小表 left join 大表`
3. <font color='cyan'>用连接查询代替子查询</font>
4. <font color='cyan'>group by 的列加索引</font>
5. <font color='cyan'>批量插入batch</font>
6. <font color='cyan'>使用limit</font>
7. <font color='cyan'>用union all 代替 union</font>: union all 不去重，union去重
8. <font color='cyan'>join表不宜过多</font>
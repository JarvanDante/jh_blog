---
author: "karson"
title: "StarRocks的Join 查询优化"
date: 2023-05-22
description: "介绍 StarRocks 在 Join 查询规划上的经验和探索。文章主要分为四个部分：Join 背景，Join 逻辑优化，Join Reorder，分布式 Join 规划。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags:
- db
- categories:

---

## 1.Join背景

### 1.1Join 类型
![/img_1.png](/images/docImages/joinType.png)
上图列举了常见的 Join 类型：
+ Cross Join：左表和右表的一个笛卡尔积。

+ Full / Left / Right Outer Join：Outer Join 需要根据语义，对两表/左表/右表上没有匹配上的行进行补 Null。

+ Anti Join：反连接，输出连接关系上 <b><font color="cyan"> 没有匹配上 </font></b>的数据行，通常 Anti Join 出现在not in 或者not exists 子查询的规划中。

+ Semi Join：与 Anti Join 相反，只输出在连接关系上匹配的数据行即可，一般是一行数据结果。

+ Inner Join：输出左表和右表的交集，根据连接条件产生一对多的结果行。

### 1.2Join 优化的难点
Join 的执行效率通常分成两部分来优化，一是 提高单机上 Join 算子的效率，二是 规划一个合理的 Join 计划，尽可能地减少 Join 的输入/执行成本。本文主要集中在后者的介绍上，那么接下来就从 Join 优化的难点开始讲起。
+ 难点一，Join 的实现方式多。
  ![/img_1.png](/images/docImages/joinWay.png)
  如上图所示，不同 Join 的实现方式在不同场景下效率不同，如 Sort-Merge Join 在 Join 有序数据时，效率可能远高于 Hash Join，但是在数据 Hash 分布的分布式数据库里，Hash Join 的效率可能远比 Sort-Merge 高。而数据库则需要针对不同的场景，选择合适的 Join 方式。
  starrocks中还是使用的Hash Join。

+ 难点二，多表 Join 的执行顺序。
  ![/img_1.png](/images/docImages/joinSort.png)
  
在多表 Join 的场景下，选择度高的 Join 先执行，会提高整个 SQL 的效率，但是怎么判断出 Join 的执行顺序呢？这却是十分困难的。

如上图所示，在 Left-Deep 模型下，N 个表 Join 可能的排列个数有2^n-1个，但是在 Bushy 模型下，排列个数高达2^(n-1) * C(n-1)个，对于数据库而言，查找一个最佳 Join 顺序的耗时和成本是指数级的增长。

+ 难点三，数据的基数/行数/相关度难以评估
  ![/img_1.png](/images/docImages/joinData.png)

在执行 SQL 之前，数据库难以准确评估一个 Join 实际的执行效果，通常我们都认为小表 Join 大表的选择度高于大表 Join 大表。但是实际情况下呢？显然并不是这样的，还有很多一对多的场景，甚至在更复杂的 SQL 中，存在各种聚合、过滤的算子，在数据经过一系列运算后，数据库系统对于 Join 的输入都会难以评估准确。
+ 难点四，单机最优的计划不等于分布式最优
  ![/img_1.png](/images/docImages/joinLocal.png)
  在分布式系统中，会通过 Re-Shuffle 或者广播数据的方式，将需要的数据发送到目的端参与计算，分布式数据库中 Join 也是如此。但这也带来了另外一个问题，一个单机数据库上最优的执行计划，因为没有考虑数据的分布 & 网络传输的开销，放在分布式数据库上未必是最优的执行计划。分布式数据库在规划 Join 的执行计划和执行方式时，需要考虑数据的分布和网络成本。

### 1.3SQL的优化流程
![/img_1.png](/images/docImages/joinSQL.png)
StarRocks 对于 SQL 的优化主要通过 <b><font color="cyan">优化器</font></b> 完成，主要集中在 <b><font color="cyan">Rewrite</font></b> 和 <b><font color="cyan">Optimize</font></b> 阶段。

### 1.4 Join 优化的原则
StarRocks 目前 Join 的算法主要是一个 Hash Join，默认使用右表去构建 Hash 表，在这个前提下，我们总结了五个优化方向：

1. 不同 Join 类型的算子，性能是不同的，尽可能使用性能高的 Join 类型，避免使用性能差的 Join 类型。根据 Join 输出的数据量，大致上的性能排序：Semi-Join/Anti-Join > Inner Join > Outer Join > Full Outer Join > Cross Join。

2. Hash Join 实现时，使用小表做 Hash 表，远比用一个大表做 Hash 表高效。

3. 多表 Join 时，优先执行选择度高的 Join，能大幅减少后续 Join 的开销。

4. 尽可能减少参与 Join 的数据量。

5. 尽可能减少分布式 Join 产生的网络成本。

PS:谓词的概念：
```

 谓词 ： 用于过滤和比较数据的条件。它们用于在查询中指定需要返回的数据行。
 常见的SQL谓词包括：
  
+ 比较谓词：用于比较两个值之间的关系，例如 等于（=），不等于（<>），大于（>），小于（<），大于等于（>=），小于等于（<=）等。
  
+ 逻辑谓词：用于将多个条件组合在一起进行逻辑操作，例如 AND，OR，NOT等。
  
+ 范围谓词：用于指定一个范围，例如 BETWEEN 和 IN 。
  
+ 空值谓词：用于检查是否为NULL值，例如 IS NULL 和 IS NOT NULL。
  
+ 模糊匹配谓词：用于在字符串中进行模式匹配，例如 LIKE 和 NOT LIKE。
  
+ 子查询谓词：用于在查询中嵌套另一个查询，例如 EXISTS 和 IN。

```
## 2.Join 逻辑优化
介绍一些 Join 上的启发式规则。
### 2.1 类型转换
低效率的 Join 类型转为高效的 Join 类型，主要包括以下三个转换规则：
+ 转换规则一：Cross Join 转换为 Inner Join
当 Cross Join 满足某个约束时，可以将 Cross Join 转为 Inner Join。
该约束为：Join 上至少存在一个表示连接关系的 谓词。
例如：
```
-- 转换前
Select *  From t1, t2 Where t1.v1 = t2.v1; 
 
-- 转换后, Where t1.v1 = t2.v1是连接关系谓词
Select *  From t1 Inner Join t2 On t1.v1 = t2.v1; 
```
+ 转换规则二：Outer Join 转换为 Inner Join

当满足以下约束时，可以将 Outer Join 转为 Inner Join：
1. Left / Right Outer Join 上存在一个 Right / Left 表的相关谓词；

2. 该相关谓词是一个严格（Restrick Null）谓词。
   例如：
```
 
-- 转换前                
Select *  From t1 Left Outer Join t2 On t1.v1 = t2.v1 Where t2.v1 > 0; 
-- 转换后， t2.v1 > 0 是一个 t2 表上的严格谓词
Select *  From t1 Inner Join t2 On t1.v1 = t2.v1 Where t2.v1 > 0;
```
需要注意的是，在 Outer Join 中，需要根据 On 子句的连接谓词进行 <b><font color="cyan"> 补 Null </font></b> 操作， 
<b><font color="cyan">而不是过滤 </font></b>，所以该转换规则不适用 On 子句中的连接谓词。
例如：
```shell
Select *  From t1 Left Outer Join t2 On t1.v1 = t2.v1 And t2.v1 > 1; 
-- 显然，上面的SQL和下面SQL的语义并不等价
Select *  From t1 Inner Join t2  On t1.v1 = t2.v1 And t2.v1 > 1;
```
这里需要提到一个概念，即 <b><font color="cyan">严格（Restrick Null）谓词 </font></b>.
StarRocks 把一个可以过滤掉 Null 值的谓词叫做 <b><font color="cyan">严格谓词 </font></b>，例如a > 0；
而不能过滤 Null 的谓词，叫做 <b><font color="cyan">非严格谓词 </font></b>，例如：a IS Null;
大部分谓词都是严格谓词，非严格谓词主要是 ```IS Null``` 、```IF```、 ```CASE WHEN``` 或 ```函数构成的谓词```
StarRocks 对于严格谓词的判断，用了一个简单的方法：
将需要检测的列全部替换成 Null，然后进行表达式化简。如果结果是 True，意味着输入为 Null 时，Where 子句无法过滤数据，那么该谓词是一个非严格谓词；反之，如果结果是 False 或 Null，那么是一个严格谓词。


+ 转换规则三：Full Outer Join 转为 Left / Right Outer Join
  
同样，当满足该约束时，Full Outer Join 可以转为 Left / Right Outer Join：存在一个可以 bind 到 Left / Right 表的严格谓词。
例如：
```
-- 转换前
Select *  From t1 Full Outer Join t2 On t1.v1 = t2.v1 Where t1.v1 > 0; 
-- 转换后， t1.v1 > 0 是一个左表上的谓词，且是一个严格谓词
Select *  From t1 Left Outer Join t2 On t1.v1 = t2.v1 Where t1.v1 > 0;
```

### 2.2 谓词下推
<b><font color="cyan"> 谓词下推 </font></b>是一个 Join 上非常重要，也是很常用的一个优化规则，其主要目的是 <b><font color="cyan">提前过滤 Join 的输入 </font></b>，从而提升 Join 的性能。

对于 Where 子句，当满足以下约束时，我们可以进行谓词下推，并且伴随着谓词下推，我们可以做 Join 类型转换：
1. 任意 Join 类型；
2. Where 谓词可以 bind 到其中一个输入上。
例如：
```
Select *  
From t1 Left Outer Join t2 On t1.v1 = t2.v1 
        Left Outer Join t3 On t2.v2 = t3.v2 
Where t1.v1 = 1 And t2.v1 = 2 And t3.v2 = 3;
```
其谓词下推的流程如下。

第一步，分别下推 (t1.v1 = 1 And t2.v1 = 2) 和 (t3.v2 = 3)，由于满足类型转换规则(t1 Left Outer Join t2) Left Outer Join t3 转换为 (t1 Left Outer Join t2) Inner Join t3。
![/img_1.png](/images/docImages/inner.png)
第二步，继续下推 (t1.v1 = 1) 和 (t2.v1 = 2)，且 t1 Left Outer Join t2 转换为 t1 Inner Join t2。
![/img_1.png](/images/docImages/inner1.png)

***需要注意的是***
对于 On 子句上的连接谓词，其下推的规则和 Where 子句有所不同，这里我们分为 
Inner  Join 和其他 Join 类型两种情况。

第一种情况是，对于 Inner Join，On 子句上的连接谓词下推，和 Where 子句相同，上面已经叙述过，这里不再重复。

第二种情况是，对于 Outer / Semi / Anti Join 的连接谓词下推，需要满足以下约束，且下推过程中无法进行类型转换：

必须为 [Left/Right] Outer/Semi/Anti Join；

连接谓词'只能' bind 到 [Right/Left] 输入上。

例如：
```
Select *  
From t1 Left Outer Join t2 On t1.v1 = t2.v1 And t1.v1 = 1 And t2.v1 = 2 
        Left Outer Join t3 On t2.v2 = t3.v2 And t3.v2 = 3;
```
其 On 连接谓词下推的流程如下。

第一步，下推 t1 Left Join t2 Left Join t3 上可以 bind 到右表的连接谓词 (t3.v2 = 3)，此时无法将 Left Outer Join 转换为 Inner Join。
![/img_1.png](/images/docImages/left.png)
第二步，下推 t1 Left Join t2 上可以 bind 到右表的连接谓词 (t2.v1 = 2)。由于t1.v1 = 1是 bind 到左表的，下推以后会过滤 t1 的数据，所以该行为与 Left Outer Join 语义不符，无法下推该谓词。
![/img_1.png](/images/docImages/left1.png)

### 2.3 谓词提取

在之前的谓词下推的规则中，只能下推满足 <b><font color="cyan">合取 </font></b>语义的谓词，例如 t1.v1 = 1 And t2.v1 = 2 And t3.v2 = 3 中，三个子谓词都是通过合取谓词连接，而无法下推 <b><font color="cyan">析取 </font></b>语义的谓词，例如t1.v1 = 1 Or t2.v1 = 2 Or t3.v2 = 3。

但是在实际场景中，析取谓词也十分常见，对此 StarRocks 做了一个提取谓词（列值推导）的优化。通过一系列的交并集操作，将析取谓词中的列值范围提取出合取谓词，继而下推合取谓词。例如：
```
-- 谓词提取前        
Select *  
From t1 Join t2 On t1.v1 = t2.v1 
Where (t2.v1 = 2 AND t1.v2 = 3) OR (t2.v1 > 5 AND t1.v2 = 4)
 
-- 利用(t2.v1 = 2 AND t1.v2 = 3) OR (t2.v1 > 5 AND t1.v2 = 4)进行列值推导，推导出（t2.v1 >= 2），（t1.v2 IN (3, 4)）两个谓词
Select *  
From t1 Join t2 On t1.v1 = t2.v1  
Where (t2.v1 = 2 AND t1.v2 = 3) OR (t2.v1 > 5 AND t1.v2 = 4) 
AND t2.v1 >= 2 AND t1.v2 IN (3, 4);
```
这里需要注意的是，提取出来的谓词范围可能是原始谓词范围的超集，所以不一定能直接替换原始谓词。
### 2.4 等价推导
在谓词上，除了上述的谓词提取，还有另一个重要的优化，叫 <b><font color="cyan">等价推导</font></b> 。等价推导主要利用了 Join 的连接关系，从左表/右表列的取值范围，推导出右表/左表对应列的取值范围。例如：
```
-- 原始SQL
Select *  
From t1 Join t2 On t1.v1 = t2.v1 
Where (t2.v1 = 2 AND t1.v2 = 3) OR (t2.v1 > 5 AND t1.v2 = 4) 
 
-- 利用(t2.v1 = 2 AND t1.v2 = 3) OR (t2.v1 > 5 AND t1.v2 = 4)进行列值推导，推导出（t2.v1 >= 2），（t1.v2 IN (3, 4)）两个谓词
Select *  
From t1 Join t2 On t1.v1 = t2.v1  
Where (t2.v1 = 2 AND t1.v2 = 3) OR (t2.v1 > 5 AND t1.v2 = 4) 
AND t2.v1 >= 2 AND t1.v2 IN (3, 4); 
 
-- 利用连接谓词(t1.v1 = t2.v1)和(t2.v1 >= 2)进行等价推导，推导出（t1.v1 >= 2）谓词
Select *  
From t1 Join t2 On t1.v1 = t2.v1 
Where (t2.v1 = 2 AND t1.v2 = 3) OR (t2.v1 > 5 AND t1.v2 = 4) 
AND t2.v1 >= 2 AND t1.v2 IN (3, 4) AND t1.v1 >= 2;
```
当然，等价推导的作用范围并不像谓词提取一样广泛，谓词提取可以在任意谓词上进行，但等价推导和谓词下推类似，在不同的 Join 上有不同的条件约束，这里同样分为 Where 谓词和 On 连接谓词来解析。

Where 谓词：

几乎没有约束，可以从左表的谓词推导出右表，反之亦可。

On 连接谓词：

在 Inner Join 上和 Where 谓词相同，没有条件约束；

除 Inner Join 外，仅支持 Semi Join 和 Outer Join，且仅支持与 Join 方向相反的单向推导。例如，Left Outer Join 可以从左表的谓词推导出右表的谓词，Right Outer Join 可以从右表的谓词推导出左表的谓词。

为什么在 Outer / Semi Join 上存在单向的限制呢？原因也很简单，以 Left Outer Join 为例，在谓词下推的规则中有提到，Left Outer Join 只能下推右表的谓词，而左表的谓词则由于违法语义导致无法下推。所以执行等价推导时，从右表谓词推导出的左表谓词，同样需要满足该约束。

那么在这个前提下，推导出来的左表谓词并不能起到提前过滤数据的作用，而且还会带来执行额外谓词的开销，所以 Outer / Semi Join 只支持单向推导。

关于等价推导的实现，StarRocks 是通过维护了两个 Map 实现的。一个 Map 用于维护 Column 和 Column 之间的等价关系，另一个 Map 则用来维护 Column 到 Value 或者表达式的等值关系，通过这两个 Map 相互查找，实现等价推导

### 2.5 Limit 下推 
除了谓词可以下推，Join 上也支持 Limit 的下推。当 SQL 是一个 Outer Join 或 Cross Join 时，可以将 Limit 下推到输出行数稳定的孩子上。其中，Left Outer Join 输出行数至少和左孩子一致，那么 Limit 可以下推到左表上，Right Outer Join 反之。
```angular2html
-- 下推前
Select * 
From t1 Left Outer Join t2 On  t1.v1 = t2.v1 
Limit 100; 
 
-- 下推后
Select * 
From (Select * From t1 Limit 100) t Left Outer Join t2 On t.v1 = t2.v1 
Limit 100;
```
比较特殊的是 Cross Join 和 Full Outer Join、Cross Join 的输出是一个笛卡尔积，行数是左表 x 右表；而 Full Outer Join 的输出行数，则至少是左表 + 右表，所以这两种 Join 可以在左表和右表上各下推一个 Limit。例如：
```angular2html
-- 下推前
Select * 
From t1 Join t2
Limit 100; 
 
-- 下推后
Select * 
From (Select * From t1 Limit 100) x1 Join 
     (Select * From t2 Limit 100)
Limit 100;
```
---
author: "jarvan"
title: "JVM 底层原理、调优配置、核心参数与项目常见坑总结"
date: 2025-01-21
description: "系统整理 JVM 运行时内存结构、类加载机制、GC 原理、常用调优参数、排查命令，以及项目中常见的内存、GC、线程和配置问题解决方法。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👤
tags:
- java
- jvm
- gc
- tuning
- performance
categories:
- java
---

# JVM 是什么

JVM 全称是 Java Virtual Machine，也就是 Java 虚拟机。

Java 程序不是直接运行在操作系统上，而是先编译成字节码，再交给 JVM 执行。

整体流程：

```shell
Java 源码
   ↓ javac
.class 字节码
   ↓
类加载器
   ↓
JVM 运行时数据区
   ↓
解释执行 / JIT 编译
   ↓
操作系统 / CPU
```

JVM 解决的核心问题：

```shell
一次编译，到处运行
自动内存管理
垃圾回收
运行时优化
线程管理
异常处理
安全隔离
```

但 JVM 自动管理不代表不需要调优。

项目出现内存溢出、频繁 Full GC、接口抖动、CPU 飙高、线程卡死时，最终经常都要回到 JVM 层面排查。

---

# JVM 运行时内存结构

JVM 运行时数据区可以分为几块：

```shell
线程共享：
  堆 Heap
  方法区 / 元空间 Metaspace

线程私有：
  虚拟机栈 JVM Stack
  本地方法栈 Native Method Stack
  程序计数器 Program Counter Register
```

常见问题大多集中在：

```shell
堆内存溢出
元空间溢出
栈溢出
直接内存溢出
GC 频繁
线程过多
```

---

# 一、堆内存 Heap

堆是 JVM 中最大的一块内存区域。

主要存放：

```shell
对象实例
数组
集合数据
业务缓存
字符串对象
```

例如：

```java
User user = new User();
List<Order> orders = new ArrayList<>();
```

这些对象大多数都在堆里。

堆内存也是 GC 重点管理区域。

常见参数：

```shell
-Xms     初始堆大小
-Xmx     最大堆大小
```

示例：

```shell
-Xms2g -Xmx2g
```

生产环境通常建议：

```shell
Xms 和 Xmx 设置成一样
```

这样可以减少运行过程中堆扩容和收缩带来的抖动。

---

# 二、年轻代和老年代

堆通常会按对象生命周期分代。

```shell
年轻代 Young Generation
  Eden
  Survivor S0
  Survivor S1

老年代 Old Generation
```

对象一般先进入 Eden 区。

如果经过多次 GC 后还存活，会晋升到老年代。

流程：

```shell
新对象创建
   ↓
进入 Eden
   ↓
Minor GC 后仍存活
   ↓
进入 Survivor
   ↓
多次存活后
   ↓
晋升老年代
```

年轻代适合存放“朝生夕死”的对象。

老年代适合存放生命周期较长的对象。

---

# 三、方法区 / 元空间 Metaspace

JDK 8 以后，永久代被元空间 Metaspace 替代。

元空间主要存放：

```shell
类元信息
方法信息
字段信息
常量池相关信息
运行时生成的代理类信息
```

常见参数：

```shell
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
```

如果项目中大量使用：

```shell
CGLIB
动态代理
Groovy
反射生成类
热部署
脚本引擎
```

就要关注元空间增长。

常见错误：

```shell
java.lang.OutOfMemoryError: Metaspace
```

解决思路：

- 检查是否动态生成类过多。
- 检查类加载器是否泄漏。
- 设置合理 `MaxMetaspaceSize`。
- 避免频繁热部署导致旧 ClassLoader 无法释放。

---

# 四、虚拟机栈 JVM Stack

每个线程都有自己的栈。

每次方法调用，都会创建一个栈帧。

栈帧中包含：

```shell
局部变量表
操作数栈
方法返回地址
动态链接
```

常见参数：

```shell
-Xss
```

示例：

```shell
-Xss512k
```

栈太小，递归或深层调用容易：

```shell
java.lang.StackOverflowError
```

栈太大，单个线程占用内存变多，可创建线程数量变少。

所以不要盲目把 `-Xss` 调得很大。

---

# 五、程序计数器

程序计数器用于记录当前线程执行到哪一条字节码指令。

它是线程私有的。

这个区域一般不会出现明显的内存问题。

可以理解为：

```shell
线程执行字节码时的位置指针
```

---

# 类加载机制

Java 类从 `.class` 文件到 JVM 中可用，要经过类加载过程。

主要阶段：

```shell
加载 Loading
验证 Verification
准备 Preparation
解析 Resolution
初始化 Initialization
使用 Using
卸载 Unloading
```

常见类加载器：

```shell
Bootstrap ClassLoader     加载 Java 核心类
Extension / Platform      加载扩展类
Application ClassLoader   加载应用 classpath
自定义 ClassLoader        框架或容器自定义
```

类加载核心机制：

```shell
双亲委派模型
```

它的基本逻辑：

```shell
先让父加载器尝试加载
父加载器加载不了
再由子加载器加载
```

这样可以避免核心类被随意覆盖。

例如你自己写一个：

```java
java.lang.String
```

正常情况下不会替换 JDK 自带的 String。

---

# GC 是什么

GC 是 Garbage Collection，也就是垃圾回收。

JVM 会自动识别不再使用的对象，并回收它们占用的内存。

GC 要解决两个问题：

```shell
哪些对象已经没用了？
如何把这些对象占用的内存回收？
```

判断对象是否存活，常见是从 GC Roots 出发做可达性分析。

GC Roots 常见来源：

```shell
栈中的局部变量引用
静态变量引用
常量引用
JNI 引用
正在运行线程相关引用
```

如果一个对象从 GC Roots 不可达，就可能被回收。

---

# 常见 GC 类型

## 1、Minor GC

年轻代 GC。

特点：

```shell
发生频繁
速度较快
主要回收 Eden 和 Survivor
```

## 2、Major GC / Old GC

老年代 GC。

通常比 Minor GC 更慢。

## 3、Full GC

Full GC 通常会回收整个堆和元空间相关数据。

特点：

```shell
停顿时间长
影响接口响应
频繁出现说明系统有问题
```

项目中最需要警惕：

```shell
频繁 Full GC
Full GC 后内存不下降
GC 停顿时间过长
```

---

# 常见垃圾收集器

## 1、Serial GC

单线程 GC。

适合小内存、客户端场景。

生产服务端一般很少主动选择。

## 2、Parallel GC

吞吐量优先。

适合后台任务、批处理，对停顿不太敏感的场景。

## 3、CMS GC

低停顿收集器，曾经常用于老年代。

但 CMS 已逐渐被 G1 替代。

## 4、G1 GC

G1 是现在很多服务端项目常用选择。

特点：

```shell
面向服务端
可预测停顿
把堆划分为多个 Region
优先回收收益高的 Region
适合较大堆内存
```

常用参数：

```shell
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

注意：

```shell
MaxGCPauseMillis 是目标，不是硬性保证。
```

## 5、ZGC

ZGC 是低延迟垃圾收集器。

适合大内存、低延迟场景。

常见于新版本 JDK。

参数：

```shell
-XX:+UseZGC
```

是否使用 ZGC，要结合 JDK 版本、业务延迟要求、机器资源综合评估。

---

# JVM 核心配置参数

## 1、堆内存

```shell
-Xms2g
-Xmx2g
```

建议：

```shell
生产环境 Xms 和 Xmx 一般设置一致。
```

## 2、栈大小

```shell
-Xss512k
```

线程很多的服务，可以适当降低，但要避免递归或复杂调用栈溢出。

## 3、元空间

```shell
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
```

建议设置上限，避免元空间无限占用系统内存。

## 4、选择 GC

G1：

```shell
-XX:+UseG1GC
```

ZGC：

```shell
-XX:+UseZGC
```

## 5、GC 停顿目标

```shell
-XX:MaxGCPauseMillis=200
```

主要用于 G1。

表示期望 GC 停顿控制在 200ms 左右。

## 6、OOM 时生成堆转储

```shell
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/data/logs/dump
```

这个非常重要。

没有 dump 文件，很多 OOM 问题只能靠猜。

## 7、打印 GC 日志

JDK 8：

```shell
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:/data/logs/gc.log
```

JDK 9+：

```shell
-Xlog:gc*:file=/data/logs/gc.log:time,uptime,level,tags
```

GC 日志是分析停顿和内存变化的关键。

## 8、容器环境内存

在 Docker / K8s 中，注意 JVM 对容器内存的识别。

常见参数：

```shell
-XX:MaxRAMPercentage=70
-XX:InitialRAMPercentage=70
```

不要让 `-Xmx` 接近容器 limit 的 100%。

因为进程还需要：

```shell
线程栈
元空间
直接内存
JIT Code Cache
Native 内存
```

---

# 推荐生产配置示例

普通 SpringBoot 服务，2C4G 容器示例：

```shell
-Xms2g
-Xmx2g
-Xss512k
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/data/logs/dump
-Xlog:gc*:file=/data/logs/gc.log:time,uptime,level,tags
```

如果是 JDK 8，GC 日志改成：

```shell
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:/data/logs/gc.log
```

注意：

```shell
配置不是越大越好，要结合机器内存、QPS、对象分配速度和 GC 日志调整。
```

---

# JVM 调优思路

JVM 调优不要一上来就改参数。

正确流程：

```shell
先观察现象
        ↓
收集数据
        ↓
定位瓶颈
        ↓
小步调整
        ↓
压测验证
        ↓
上线观察
```

重点看：

```shell
CPU 是否异常
内存是否持续增长
Full GC 是否频繁
GC 后内存是否下降
接口 P99 是否抖动
线程数是否异常
对象分配速度是否过快
```

不要凭感觉调 JVM。

---

# 常用排查命令

## 1、jps

查看 Java 进程：

```shell
jps -l
```

## 2、jstat

查看 GC 情况：

```shell
jstat -gcutil <pid> 1000 10
```

含义：

```shell
每 1 秒输出一次，一共输出 10 次
```

重点看：

```shell
YGC     年轻代 GC 次数
YGCT    年轻代 GC 总耗时
FGC     Full GC 次数
FGCT    Full GC 总耗时
GCT     GC 总耗时
```

## 3、jmap

查看堆信息：

```shell
jmap -heap <pid>
```

导出 dump：

```shell
jmap -dump:format=b,file=/tmp/heap.hprof <pid>
```

注意：

```shell
线上导 dump 可能会造成进程停顿，要谨慎操作。
```

## 4、jstack

查看线程栈：

```shell
jstack <pid> > thread.txt
```

常用于排查：

```shell
CPU 飙高
死锁
线程阻塞
接口卡死
线程池耗尽
```

## 5、arthas

Arthas 是非常实用的线上 Java 诊断工具。

常用命令：

```shell
dashboard
thread
jvm
watch
trace
stack
heapdump
```

例如查看最忙线程：

```shell
thread -n 5
```

---

# 常见坑一：频繁 Full GC

现象：

```shell
接口突然变慢
P99 延迟升高
CPU 不一定很高
GC 日志中 Full GC 很频繁
```

常见原因：

```shell
老年代空间不足
大对象太多
内存泄漏
缓存没有上限
一次性加载大量数据
对象生命周期过长
```

解决思路：

- 分析 GC 日志。
- 查看 Full GC 后内存是否下降。
- 导出 heap dump。
- 使用 MAT / VisualVM 分析大对象。
- 检查本地缓存是否设置最大容量。
- 检查大查询、大 List、大 Map。

如果 Full GC 后内存明显下降，可能是瞬时对象压力大。

如果 Full GC 后内存不下降，重点怀疑内存泄漏或长期持有引用。

---

# 常见坑二：内存泄漏

Java 有 GC，但仍然会内存泄漏。

原因是对象还被引用着，GC 认为它还活着。

常见场景：

```shell
static Map 无限增长
本地缓存没有容量限制
ThreadLocal 使用后不 remove
监听器注册后不注销
集合只加不删
连接对象未关闭
类加载器泄漏
```

典型问题：

```java
private static final Map<String, Object> CACHE = new HashMap<>();
```

如果这个 Map 一直 put，不清理，就会内存泄漏。

解决：

```shell
使用 Caffeine 设置 maximumSize
ThreadLocal finally 中 remove
集合增加清理机制
关闭资源
限制批量查询大小
```

ThreadLocal 正确写法：

```java
try {
    USER_CONTEXT.set(user);
    doBusiness();
} finally {
    USER_CONTEXT.remove();
}
```

---

# 常见坑三：一次性查询太多数据

问题代码：

```java
List<Order> orders = orderMapper.selectAll();
```

如果数据量很大，会导致：

```shell
堆内存暴涨
频繁 GC
接口超时
甚至 OOM
```

解决：

```shell
分页查询
游标查询
流式处理
限制导出数量
大任务异步化
```

示例：

```java
long lastId = 0L;
int pageSize = 1000;

while (true) {
    List<Order> orders = orderMapper.selectByLastId(lastId, pageSize);
    if (orders.isEmpty()) {
        break;
    }

    handle(orders);
    lastId = orders.get(orders.size() - 1).getId();
}
```

---

# 常见坑四：线程数过多

每个 Java 线程都有栈内存。

如果线程数过多，会占用大量内存，并增加上下文切换成本。

常见原因：

```shell
线程池无界
每个请求创建新线程
定时任务重复创建线程池
第三方 SDK 内部线程过多
阻塞 IO 导致线程堆积
```

错误示例：

```java
Executors.newCachedThreadPool();
```

风险：

```shell
线程数可能无限增长
```

建议使用有界线程池：

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
        10,
        50,
        60,
        TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(1000),
        new ThreadPoolExecutor.CallerRunsPolicy()
);
```

线程池一定要明确：

```shell
核心线程数
最大线程数
队列大小
拒绝策略
线程名称
监控指标
```

---

# 常见坑五：容器内存配置不合理

在 Docker / K8s 中，JVM 内存不能只看 `-Xmx`。

容器总内存还要包括：

```shell
堆内存
线程栈
元空间
直接内存
JIT 编译缓存
JNI 内存
操作系统开销
```

如果容器 limit 是 2G，你设置：

```shell
-Xmx2g
```

很容易被容器 OOM Kill。

建议：

```shell
Xmx 设置为容器内存的 60% ~ 75%
预留空间给非堆内存
监控 RSS 和 JVM Heap
```

---

# 常见坑六：直接内存泄漏

Netty、NIO、ByteBuffer、部分中间件会使用直接内存。

常见错误：

```shell
java.lang.OutOfMemoryError: Direct buffer memory
```

相关参数：

```shell
-XX:MaxDirectMemorySize=512m
```

排查方向：

- 是否大量创建 DirectByteBuffer。
- Netty ByteBuf 是否释放。
- 文件上传下载是否占用过多直接内存。
- 中间件客户端配置是否合理。

---

# 常见坑七：GC 日志没有打开

很多线上问题发生时，最尴尬的是没有 GC 日志，也没有 heap dump。

建议生产默认开启：

```shell
GC 日志
OOM heap dump
应用关键指标
线程池监控
接口耗时监控
```

否则出现问题时只能靠现象猜。

---

# 调优经验总结

## 1、先优化代码，再调 JVM 参数

很多 JVM 问题本质是代码问题。

例如：

```shell
大对象
大查询
缓存无限增长
线程池无界
SQL 太慢
对象创建太频繁
```

参数只能缓解，不能根治。

## 2、不要盲目加大堆

堆越大，不一定越好。

堆变大后：

```shell
GC 扫描成本可能增加
Full GC 停顿可能变长
问题可能被延后暴露
```

如果是内存泄漏，加大堆只是让 OOM 晚一点发生。

## 3、关注 P99，而不是只看平均值

平均响应时间正常，不代表系统稳定。

GC 停顿通常会体现在：

```shell
P95
P99
P999
```

所以线上服务要关注长尾延迟。

## 4、调优要有对比数据

每次调整参数前后，都要对比：

```shell
QPS
平均响应时间
P95 / P99
GC 次数
GC 总耗时
Full GC 次数
CPU
内存曲线
错误率
```

没有数据对比，就不知道调优是否有效。

---

# 总结

JVM 调优不是背几个参数，而是理解运行原理后，根据业务现象做定位。

核心要掌握：

```shell
JVM 内存结构
类加载机制
GC Roots 和可达性分析
年轻代 / 老年代
常见垃圾收集器
GC 日志分析
heap dump 分析
线程栈分析
容器内存模型
```

生产环境常用配置重点：

```shell
-Xms
-Xmx
-Xss
-XX:MetaspaceSize
-XX:MaxMetaspaceSize
-XX:+UseG1GC
-XX:MaxGCPauseMillis
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath
-Xlog:gc*
```

项目中最常见的问题：

```shell
频繁 Full GC
内存泄漏
一次性加载大量数据
线程池无界
容器内存配置不合理
直接内存泄漏
没有 GC 日志和 dump
```

我的经验是：

```shell
先看监控
再看 GC 日志
再看线程栈
再看 heap dump
最后再调参数
```

JVM 调优的目标不是把参数调得复杂，而是让系统在高并发、长时间运行、异常流量下依然稳定、可观测、可恢复。

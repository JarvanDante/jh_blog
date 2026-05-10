---
author: "jarvan"
title: "Java 本地缓存怎么选？Caffeine 与 Guava Cache 使用、原理、技巧总结"
date: 2025-01-16
description: "系统整理 Java 本地缓存的使用场景、核心原理、Caffeine 和 Guava Cache 的区别，并重点说明 Caffeine 在 SpringBoot 项目中的配置、代码示例和注意事项。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👤
tags:
- java
- cache
- caffeine
- guava
- springboot
categories:
- java
---

# Java 本地缓存是什么

Java 本地缓存，就是把热点数据直接缓存在当前应用 JVM 内存中。

请求进来后，优先从本地内存读取，读取不到再访问数据库、Redis 或第三方接口。

简单流程：

```shell
请求进入应用
        ↓
先查本地缓存
        ↓
命中：直接返回
        ↓
未命中：查询数据库 / Redis / 远程接口
        ↓
写入本地缓存
        ↓
返回结果
```

本地缓存的核心价值：

```shell
用内存换速度，减少重复查询和远程调用。
```

常见本地缓存框架：

```shell
Caffeine
Guava Cache
Ehcache
Map + 定时清理
```

现在新项目中，如果只是 Java 进程内本地缓存，我更推荐优先使用 `Caffeine`。

---

# 为什么需要本地缓存

在业务系统中，有很多数据并不需要每次都查数据库或远程服务。

例如：

```shell
系统配置
字典数据
地区信息
权限菜单
用户基础信息
商品分类
渠道配置
风控规则
接口 token
第三方 appId 配置
```

这些数据有几个特点：

```shell
读多写少
变化不频繁
访问频率高
允许短时间延迟
对响应速度敏感
```

如果每次请求都查数据库，会带来几个问题：

- 数据库压力变大。
- 接口响应变慢。
- 高并发下容易出现慢查询。
- 远程接口调用成本高。
- Redis 网络访问也会有额外开销。

本地缓存可以把热点数据放在应用内存里，访问速度非常快。

---

# 本地缓存和 Redis 的区别

很多人会问：既然有 Redis，为什么还要本地缓存？

区别如下：

```shell
本地缓存：数据在当前 JVM 内存中，速度最快，但多实例之间不共享。
Redis：数据在独立缓存服务中，多实例共享，但需要网络访问。
```

对比：

```shell
本地缓存
  优点：速度快、无网络开销、实现简单
  缺点：占用 JVM 内存、多实例数据不一致、应用重启后丢失

Redis
  优点：集中存储、多实例共享、支持持久化和分布式能力
  缺点：有网络开销、依赖外部服务、复杂度更高
```

常见组合方式：

```shell
本地缓存 Caffeine
        ↓ 未命中
Redis
        ↓ 未命中
数据库
```

也就是多级缓存：

```shell
L1：本地缓存
L2：Redis
DB：数据库
```

---

# 本地缓存核心原理

本地缓存本质上是一个增强版的 Map。

但它不只是简单的 `HashMap`，还要解决几个关键问题：

```shell
缓存容量如何控制？
缓存什么时候过期？
并发访问是否安全？
缓存未命中时如何加载？
热点数据如何保留？
冷数据如何淘汰？
缓存统计如何观察？
```

所以成熟缓存框架一般会提供：

```shell
最大容量
过期策略
淘汰策略
自动加载
异步刷新
并发安全
命中率统计
删除监听
```

核心目标：

```shell
让热点数据留在缓存里，让冷数据及时释放。
```

---

# 缓存常见淘汰策略

常见策略：

```shell
FIFO：先进先出
LRU：最近最少使用
LFU：最不经常使用
W-TinyLFU：Caffeine 使用的核心策略之一
```

## LRU

LRU 关注的是：

```shell
最近有没有被访问
```

最近没被访问的数据更容易被淘汰。

问题是：偶发的大量访问可能污染缓存。

## LFU

LFU 关注的是：

```shell
访问频率高不高
```

访问次数少的数据更容易淘汰。

问题是：历史热点可能长期占据缓存。

## Caffeine 的 W-TinyLFU

Caffeine 使用 Window TinyLFU 思想，兼顾近期访问和历史频率。

可以理解为：

```shell
新数据先进入窗口区
        ↓
经过访问竞争
        ↓
真正高价值的数据进入主缓存
        ↓
低价值数据被淘汰
```

它比传统 LRU 更适合真实业务里的热点访问。

---

# Caffeine 是什么

Caffeine 是一个高性能 Java 本地缓存库。

它的特点：

```shell
性能高
并发能力强
支持自动加载
支持异步加载
支持过期策略
支持最大容量
支持刷新策略
支持删除监听
支持命中率统计
SpringBoot 集成方便
```

依赖：

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

如果使用 Spring Cache：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

---

# Caffeine 基础使用

## 1、手动缓存

```java
Cache<String, String> cache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(Duration.ofMinutes(10))
        .recordStats()
        .build();

cache.put("user:1", "jarvan");

String value = cache.getIfPresent("user:1");
```

说明：

```shell
maximumSize       最大缓存数量
expireAfterWrite  写入后多久过期
recordStats       开启统计
```

## 2、缓存未命中自动加载

```java
String userName = cache.get("user:1", key -> {
    return queryUserNameFromDb(key);
});
```

含义：

```shell
缓存有值，直接返回
缓存没值，执行 queryUserNameFromDb
查询结果写入缓存
```

这个写法比自己先判断再 put 更简洁，也更安全。

---

# LoadingCache

如果缓存有固定加载逻辑，可以使用 `LoadingCache`。

```java
LoadingCache<Long, UserDTO> userCache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(Duration.ofMinutes(5))
        .build(userId -> userService.queryById(userId));

UserDTO user = userCache.get(1001L);
```

适合场景：

```shell
根据 key 查询固定数据
用户基础信息
配置详情
字典项
渠道信息
```

---

# expireAfterWrite 和 expireAfterAccess

Caffeine 常用两种过期方式。

## 1、expireAfterWrite

写入后固定时间过期。

```java
Caffeine.newBuilder()
        .expireAfterWrite(Duration.ofMinutes(10));
```

适合：

```shell
配置数据
字典数据
接口 token
短时间可接受延迟的数据
```

## 2、expireAfterAccess

最后一次访问后固定时间过期。

```java
Caffeine.newBuilder()
        .expireAfterAccess(Duration.ofMinutes(10));
```

适合：

```shell
热点用户信息
会话临时数据
短期频繁访问的数据
```

区别：

```shell
expireAfterWrite：从写入开始算
expireAfterAccess：从最后一次访问开始算
```

---

# refreshAfterWrite

`refreshAfterWrite` 表示写入一段时间后，下一次访问触发刷新。

```java
LoadingCache<Long, UserDTO> userCache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(Duration.ofMinutes(30))
        .refreshAfterWrite(Duration.ofMinutes(5))
        .build(userId -> userService.queryById(userId));
```

注意：

```shell
refreshAfterWrite 不是到时间立刻刷新
而是到时间后，下一次访问触发刷新
```

它适合需要尽量保持数据新鲜，但又不想频繁阻塞请求的场景。

---

# 删除监听

可以监听缓存删除原因。

```java
Cache<String, String> cache = Caffeine.newBuilder()
        .maximumSize(1000)
        .removalListener((String key, String value, RemovalCause cause) -> {
            System.out.println("key=" + key + ", cause=" + cause);
        })
        .build();
```

常见删除原因：

```shell
EXPLICIT    主动删除
REPLACED    被新值替换
COLLECTED   被 GC 回收
EXPIRED     过期
SIZE        超过容量被淘汰
```

---

# 缓存统计

开启统计：

```java
Cache<String, String> cache = Caffeine.newBuilder()
        .maximumSize(1000)
        .recordStats()
        .build();
```

查看：

```java
CacheStats stats = cache.stats();

System.out.println(stats.hitCount());
System.out.println(stats.missCount());
System.out.println(stats.hitRate());
System.out.println(stats.evictionCount());
```

重点关注：

```shell
命中率
未命中次数
淘汰次数
加载耗时
缓存大小
```

如果命中率很低，说明缓存 key、过期时间或业务场景可能不适合。

---

# SpringBoot 集成 Caffeine

## 1、开启缓存

```java
@SpringBootApplication
@EnableCaching
public class DemoApplication {
}
```

## 2、配置 Caffeine

```yml
spring:
  cache:
    type: caffeine
    cache-names:
      - userCache
      - configCache
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=10m,recordStats
```

## 3、使用 @Cacheable

```java
@Service
public class UserService {

    @Cacheable(cacheNames = "userCache", key = "#userId")
    public UserDTO getUserById(Long userId) {
        return queryFromDb(userId);
    }
}
```

流程：

```shell
第一次调用：查数据库，并写入缓存
第二次调用：直接从缓存返回
```

## 4、更新后清理缓存

```java
@CacheEvict(cacheNames = "userCache", key = "#userId")
public void updateUser(Long userId, UpdateUserRequest request) {
    updateDb(userId, request);
}
```

更新数据库后清理缓存，下一次查询重新加载。

## 5、写入或更新缓存

```java
@CachePut(cacheNames = "userCache", key = "#result.id")
public UserDTO updateAndReturn(UserDTO user) {
    updateDb(user);
    return user;
}
```

`@CachePut` 会执行方法，并把结果写入缓存。

---

# Guava Cache 是什么

Guava Cache 是 Google Guava 提供的本地缓存能力。

依赖：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.0.0-jre</version>
</dependency>
```

基础使用：

```java
LoadingCache<Long, UserDTO> cache = CacheBuilder.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .recordStats()
        .build(new CacheLoader<Long, UserDTO>() {
            @Override
            public UserDTO load(Long userId) {
                return userService.queryById(userId);
            }
        });

UserDTO user = cache.get(1001L);
```

Guava Cache 曾经使用非常广，但现在本地缓存新项目更推荐 Caffeine。

---

# Caffeine 和 Guava Cache 对比

## 1、性能

```shell
Caffeine 性能更强，并发场景更好。
Guava Cache 能满足普通场景，但高并发下不如 Caffeine。
```

Caffeine 的实现更现代，针对多核并发、缓存淘汰、写入读取都做了更多优化。

## 2、淘汰策略

```shell
Caffeine：W-TinyLFU，更适合真实热点访问
Guava Cache：传统 LRU 思路为主
```

如果业务有明显热点数据，Caffeine 更容易保留真正高价值缓存。

## 3、异步能力

Caffeine 原生支持异步缓存：

```java
AsyncLoadingCache<Long, UserDTO> cache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(Duration.ofMinutes(10))
        .buildAsync(userId -> userService.queryById(userId));

CompletableFuture<UserDTO> future = cache.get(1001L);
```

Guava Cache 的异步能力相对弱一些。

## 4、SpringBoot 集成

SpringBoot 对 Caffeine 支持很好。

配置简单：

```yml
spring:
  cache:
    type: caffeine
```

配合 `@Cacheable`、`@CacheEvict`、`@CachePut` 使用很方便。

## 5、选择建议

```shell
新项目：优先 Caffeine
老项目已经使用 Guava Cache：可以继续维护，后续逐步迁移
高并发热点缓存：优先 Caffeine
需要异步加载：优先 Caffeine
简单低频缓存：Guava Cache 也能满足
```

---

# 使用本地缓存的技巧

## 1、缓存 key 要稳定

key 设计非常重要。

推荐：

```shell
user:1001
config:site:1
dict:pay_status
channel:wechat:merchant_001
```

不要使用不稳定对象直接作为 key。

如果使用对象做 key，要确保 `equals` 和 `hashCode` 正确。

## 2、不要缓存大对象

本地缓存占用 JVM 内存。

不要缓存：

```shell
大文件
大列表
超大 JSON
大量图片
无限增长的数据集合
```

否则容易导致内存压力甚至 OOM。

## 3、一定要设置最大容量

不要只设置过期时间。

必须设置：

```java
maximumSize(10000)
```

否则缓存可能无限增长。

## 4、更新数据后及时失效

如果数据更新后缓存不清理，就会出现脏读。

常见策略：

```shell
更新数据库后删除缓存
定时过期
监听配置变更后主动刷新
后台手动清理缓存
```

## 5、不要缓存强一致数据

不适合本地缓存的场景：

```shell
余额
库存扣减
支付状态最终确认
实时风控结果
强一致权限判断
```

这些数据如果必须缓存，也要非常谨慎，最好结合 Redis、版本号、短 TTL 和主动失效机制。

## 6、防止缓存击穿

热点 key 过期瞬间，可能大量请求同时打到数据库。

处理方式：

```shell
使用 LoadingCache 的单 key 加载合并
合理设置 refreshAfterWrite
热点数据永不过期 + 后台刷新
加互斥锁
```

Caffeine 的 `cache.get(key, mappingFunction)` 对同一个 key 的并发加载有更好的保护。

## 7、空值缓存

如果某个 key 不存在，但请求很多，可能频繁查数据库。

可以缓存空结果：

```java
Cache<Long, Optional<UserDTO>> cache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(Duration.ofMinutes(3))
        .build();
```

空值缓存 TTL 要短，避免真实数据创建后长时间查不到。

---

# 项目实践建议

我一般会这样使用本地缓存：

```shell
配置类数据：Caffeine + expireAfterWrite
热点详情数据：Caffeine + maximumSize + refreshAfterWrite
第三方 token：Caffeine + expireAfterWrite，过期时间小于真实 token 过期时间
字典数据：Caffeine + 手动刷新
多实例强一致数据：不要只靠本地缓存
```

推荐分层：

```shell
Controller
    ↓
Service
    ↓
CacheService
    ↓
Repository / Remote Client
```

不要把缓存逻辑散落在各个 Controller 中。

可以封装成：

```java
@Service
public class UserLocalCache {

    private final Cache<Long, UserDTO> cache;

    public UserLocalCache(UserService userService) {
        this.cache = Caffeine.newBuilder()
                .maximumSize(10_000)
                .expireAfterWrite(Duration.ofMinutes(10))
                .recordStats()
                .build();
    }

    public UserDTO get(Long userId) {
        return cache.get(userId, this::loadUser);
    }

    public void invalidate(Long userId) {
        cache.invalidate(userId);
    }

    private UserDTO loadUser(Long userId) {
        return queryFromDb(userId);
    }
}
```

---

# 总结

Java 本地缓存适合读多写少、变化不频繁、对响应速度敏感的数据。

核心收益：

```shell
减少数据库查询
减少 Redis 网络访问
降低远程接口调用
提升接口响应速度
保护下游服务
```

Caffeine 和 Guava Cache 都能做本地缓存，但新项目更推荐 Caffeine：

```shell
Caffeine 性能更好
Caffeine 淘汰策略更先进
Caffeine 并发能力更强
Caffeine 异步支持更好
Caffeine 和 SpringBoot 集成方便
```

使用本地缓存时，要重点注意：

```shell
设置最大容量
设置合理过期时间
设计稳定 key
避免缓存大对象
更新后主动失效
监控命中率
不要缓存强一致核心数据
```

本地缓存不是为了替代 Redis 或数据库，而是作为应用内部的一级加速层。

用得好，它能明显提升性能；用得不好，也可能带来脏数据、内存膨胀和一致性问题。

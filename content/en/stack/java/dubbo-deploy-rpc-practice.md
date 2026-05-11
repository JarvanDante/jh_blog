---
author: "jarvan"
title: "Dubbo微服务部署、使用、RPC调用与注解实战总结"
date: 2026-05-11
description: "结合真实微服务项目，整理 Dubbo 的部署方式、API 契约分离、Nacos 注册中心接入、RPC 调用打通、常用注解使用，以及 BFF 层在微服务架构中的职责边界。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👤
tags:
- java
- dubbo
- rpc
- nacos
- microservice
categories:
- java
- microservice
---

# Dubbo不是简单的远程调用工具

在微服务项目里，Dubbo 最容易被理解成“像调用本地方法一样调用远程服务”的 RPC 框架。

这个理解没有错，但还不够。

真正落地时，Dubbo 解决的不只是调用问题，而是一整套服务协作问题：

```shell
服务接口如何定义？
接口和实现如何隔离？
服务如何注册和发现？
调用方如何引用远程服务？
版本如何控制？
超时和重试如何设置？
BFF 层应该调用谁？
服务之间如何避免互相乱调？
```

在我的实战项目里，微服务分层已经建立，服务之间不再直接通过 HTTP 地址硬编码调用，而是通过 Dubbo RPC 完成内部服务协作。

项目里几个关键点是：

```shell
微服务分层已经建立
API 契约分离非常关键
Dubbo RPC 调用已经打通
Nacos 作为注册中心
BFF 层负责面向前端编排，不负责沉淀核心业务能力
```

这篇文章就从部署、使用、RPC 调用、注解和项目分层几个角度，把 Dubbo 的实战落地方式整理清楚。

---

# 一、Dubbo在项目中的位置

一个比较清晰的微服务系统，可以按下面方式理解：

```shell
前端 / App / 管理后台
        ↓
BFF 层
        ↓
Dubbo RPC
        ↓
领域服务层：用户服务 / 订单服务 / 支付服务 / 内容服务
        ↓
数据层：MySQL / Redis / MQ / ES
```

这里 Dubbo 主要负责 BFF 与后端领域服务之间、领域服务与领域服务之间的内部 RPC 调用。

它不负责前端入口流量，不负责页面适配，也不应该承载太多接口聚合逻辑。

我更推荐把职责拆成这样：

```shell
Gateway：
统一入口、路由转发、鉴权、限流、跨域、安全控制

BFF：
面向具体前端场景做接口聚合、字段裁剪、数据组装

Dubbo Provider：
提供稳定的业务能力，例如用户查询、订单创建、余额扣减

Dubbo Consumer：
通过接口契约调用远程业务能力

Nacos：
服务注册、服务发现、配置管理
```

这样分层以后，系统不会因为一个前端页面需求，就把领域服务接口改得越来越像页面接口。

---

# 二、为什么API契约分离非常关键

Dubbo 项目里最容易踩的坑，就是把服务接口、DTO、业务实现全部写在同一个模块里。

这样一开始很方便，但后面会出现几个问题：

```shell
Consumer 被迫依赖 Provider 的实现模块
接口变更影响范围不可控
DTO 和数据库实体容易混用
Provider 内部代码被调用方感知
多服务之间 Maven 依赖越来越乱
```

所以实战项目里一定要做 API 契约分离。

推荐的模块结构如下：

```shell
user-api
  ├── UserRpcService.java
  ├── dto
  │   ├── UserDTO.java
  │   └── UserQueryRequest.java
  └── result
      └── RpcResult.java

user-service
  ├── UserRpcServiceImpl.java
  ├── domain
  ├── mapper
  ├── service
  └── repository

bff-service
  ├── controller
  ├── assembler
  └── remote
```

这里的核心原则是：

```shell
api 模块只放契约
service 模块负责实现契约
consumer 只依赖 api 模块
不要让 consumer 依赖 provider 的实现工程
```

比如用户服务对外暴露一个 Dubbo 接口：

```java
public interface UserRpcService {

    UserDTO getById(Long userId);

    List<UserDTO> listByIds(List<Long> userIds);
}
```

这个接口应该放在 `user-api` 模块，而不是放在 `user-service` 的业务实现包里。

DTO 也应该放在 API 模块：

```java
public class UserDTO implements Serializable {

    private Long id;

    private String username;

    private String nickname;

    private Integer status;

    // getter / setter
}
```

Dubbo RPC 会涉及网络传输，对象需要序列化，所以 DTO 要实现 `Serializable`。

---

# 三、Dubbo部署方式

Dubbo 的部署本质上不是单独部署一个 Dubbo Server，而是每个微服务应用作为 Dubbo Provider 或 Consumer 启动。

常见部署结构如下：

```shell
Nacos
  ├── nacos:8848

user-service
  ├── Spring Boot 应用
  ├── Dubbo Provider
  └── 注册到 Nacos

order-service
  ├── Spring Boot 应用
  ├── Dubbo Provider
  └── 注册到 Nacos

bff-service
  ├── Spring Boot 应用
  ├── Dubbo Consumer
  └── 从 Nacos 发现 Provider
```

启动顺序建议是：

```shell
1. 启动 Nacos
2. 启动 Provider 服务，例如 user-service、order-service
3. 确认 Provider 已注册到 Nacos
4. 启动 Consumer 服务，例如 bff-service
5. 通过 BFF 接口验证 Dubbo RPC 是否调用成功
```

如果是本地开发，Nacos 可以直接用 Docker 启动：

```shell
docker run -d \
  --name nacos-standalone \
  -e MODE=standalone \
  -p 8848:8848 \
  nacos/nacos-server
```

生产环境则建议至少考虑：

```shell
Nacos 集群部署
配置数据持久化到 MySQL
应用按环境隔离 namespace
服务按业务隔离 group
Provider 多实例部署
Consumer 设置合理超时、重试和熔断策略
```

---

# 四、Maven依赖配置

如果项目使用 Spring Boot + Dubbo + Nacos，Provider 和 Consumer 都需要引入 Dubbo Starter。

示例依赖如下：

```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-registry-nacos</artifactId>
</dependency>
```

如果使用 Spring Cloud Alibaba Nacos，也会有类似 Nacos 客户端依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

实际项目里要注意版本统一，建议通过父工程的 `dependencyManagement` 管理。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-bom</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

不要让不同服务各自随便指定 Dubbo、Nacos、Spring Boot 版本，否则后面排查兼容问题会非常痛苦。

---

# 五、Provider服务配置

Provider 是服务提供方，负责把接口实现注册到 Nacos。

一个用户服务的 `application.yml` 可以这样配置：

```yml
server:
  port: 9001

spring:
  application:
    name: user-service

dubbo:
  application:
    name: user-service
  protocol:
    name: dubbo
    port: 20881
  registry:
    address: nacos://127.0.0.1:8848
  scan:
    base-packages: com.example.user.rpc
```

这里有两个端口要区分：

```shell
server.port:
Spring Boot HTTP 端口

dubbo.protocol.port:
Dubbo RPC 协议端口
```

如果服务只是内部 RPC Provider，不一定需要暴露大量 HTTP Controller。

Provider 的接口实现示例：

```java
import org.apache.dubbo.config.annotation.DubboService;

@DubboService(version = "1.0.0", group = "user")
public class UserRpcServiceImpl implements UserRpcService {

    private final UserDomainService userDomainService;

    public UserRpcServiceImpl(UserDomainService userDomainService) {
        this.userDomainService = userDomainService;
    }

    @Override
    public UserDTO getById(Long userId) {
        User user = userDomainService.getById(userId);
        return UserAssembler.toDTO(user);
    }

    @Override
    public List<UserDTO> listByIds(List<Long> userIds) {
        List<User> users = userDomainService.listByIds(userIds);
        return UserAssembler.toDTOList(users);
    }
}
```

这里有几个重要点：

```shell
@DubboService 用来暴露 Dubbo 服务
实现类实现 api 模块里的接口
Provider 内部可以调用 domain service、repository、mapper
返回给 Consumer 的是 DTO，不是数据库 Entity
```

---

# 六、Consumer服务配置

Consumer 是服务调用方，例如 BFF 层。

BFF 的 `application.yml` 可以这样配置：

```yml
server:
  port: 8081

spring:
  application:
    name: bff-service

dubbo:
  application:
    name: bff-service
  registry:
    address: nacos://127.0.0.1:8848
  consumer:
    timeout: 3000
    retries: 0
    check: false
```

几个配置要特别注意：

```shell
timeout:
远程调用超时时间，不能无限等待

retries:
重试次数，查询接口可以谨慎开启，写接口通常建议 0

check:
启动时是否检查 Provider 存在，本地开发可设置 false，生产环境要结合发布策略判断
```

BFF 层引用远程服务：

```java
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.stereotype.Service;

@Service
public class UserBffService {

    @DubboReference(version = "1.0.0", group = "user", timeout = 3000, retries = 0)
    private UserRpcService userRpcService;

    public UserProfileVO getUserProfile(Long userId) {
        UserDTO user = userRpcService.getById(userId);
        return UserProfileVO.from(user);
    }
}
```

BFF Controller 对外提供 HTTP 接口：

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserBffService userBffService;

    public UserController(UserBffService userBffService) {
        this.userBffService = userBffService;
    }

    @GetMapping("/{userId}/profile")
    public UserProfileVO profile(@PathVariable Long userId) {
        return userBffService.getUserProfile(userId);
    }
}
```

这样调用链路就是：

```shell
前端
  ↓ HTTP
Gateway
  ↓ HTTP
BFF Controller
  ↓ Java 方法调用
BFF Service
  ↓ Dubbo RPC
UserRpcService Provider
  ↓
UserDomainService / Repository / DB
```

---

# 七、Dubbo RPC调用是如何打通的

Dubbo RPC 调用打通后，表面上看起来只是一行代码：

```java
UserDTO user = userRpcService.getById(userId);
```

但底层实际发生了很多事情：

```shell
1. Provider 启动
2. @DubboService 暴露接口
3. Provider 把服务元数据注册到 Nacos
4. Consumer 启动
5. @DubboReference 根据接口、group、version 查找 Provider
6. Consumer 从 Nacos 拉取 Provider 地址
7. Dubbo 为接口生成代理对象
8. 调用方调用接口方法
9. 代理对象把调用编码成 RPC 请求
10. 网络发送到 Provider
11. Provider 解码请求并执行真实实现类
12. 返回结果给 Consumer
```

可以用下面这个图理解：

```shell
Provider 启动
   ↓
扫描 @DubboService
   ↓
注册服务到 Nacos

Consumer 启动
   ↓
扫描 @DubboReference
   ↓
从 Nacos 订阅服务地址
   ↓
生成接口代理

BFF 调用接口方法
   ↓
Dubbo 代理发起远程调用
   ↓
Provider 执行业务实现
   ↓
返回 DTO
```

所以排查 Dubbo 调用问题时，不要只盯着一行调用代码，要顺着这条链路逐步看。

---

# 八、Dubbo常用注解

## 1、@DubboService

用于服务提供方，表示把当前实现类暴露成 Dubbo 服务。

```java
@DubboService(version = "1.0.0", group = "user", timeout = 3000)
public class UserRpcServiceImpl implements UserRpcService {
}
```

常用属性：

```shell
version:
接口版本，用于兼容升级

group:
服务分组，用于环境、业务或租户隔离

timeout:
服务方法默认超时时间

retries:
失败重试次数

interfaceClass:
显式指定暴露的接口类型
```

## 2、@DubboReference

用于服务消费方，表示注入一个远程 Dubbo 服务代理。

```java
@DubboReference(version = "1.0.0", group = "user", timeout = 3000, retries = 0)
private UserRpcService userRpcService;
```

常用属性：

```shell
version:
必须和 Provider 暴露的版本一致

group:
必须和 Provider 暴露的分组一致

timeout:
当前引用的调用超时时间

retries:
当前引用的失败重试次数

check:
启动时是否检查服务提供方存在

loadbalance:
负载均衡策略
```

## 3、@EnableDubbo

用于开启 Dubbo 注解扫描。

```java
@SpringBootApplication
@EnableDubbo
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

如果已经通过 `dubbo.scan.base-packages` 配置了扫描路径，有些项目可以不显式写 `@EnableDubbo`，但我更倾向于保持启动类语义清晰。

## 4、@DubboComponentScan

用于指定 Dubbo 组件扫描路径。

```java
@SpringBootApplication
@DubboComponentScan(basePackages = "com.example.user.rpc")
public class UserServiceApplication {
}
```

如果项目模块很多，扫描路径不要写得过大，否则启动时可能扫描到不该暴露的实现。

---

# 九、版本、分组和接口演进

API 契约分离以后，接口版本管理就很重要。

比如原接口是：

```java
UserDTO getById(Long userId);
```

后来要新增字段，可以优先通过 DTO 兼容扩展：

```java
public class UserDTO implements Serializable {

    private Long id;

    private String username;

    private String nickname;

    private String avatarUrl;
}
```

如果方法语义发生明显变化，就不要直接改老方法，而是新增方法或新版本接口：

```java
@DubboService(version = "2.0.0", group = "user")
public class UserRpcServiceV2Impl implements UserRpcService {
}
```

Consumer 也要明确引用版本：

```java
@DubboReference(version = "2.0.0", group = "user")
private UserRpcService userRpcService;
```

版本管理的基本原则：

```shell
兼容字段可以加
方法入参不要随意改
返回对象不要删除已有字段
语义变化用新方法或新版本
Provider 和 Consumer 不要同时强依赖一次上线
```

Dubbo 的 version 和 group 不是装饰性配置，而是服务治理的重要手段。

---

# 十、BFF层职责边界

在我的项目里，BFF 层非常关键。

BFF 不是简单的 Controller，也不是新的业务大泥球。它的核心职责是面向前端场景做编排。

BFF 可以做：

```shell
聚合多个 Dubbo 服务返回的数据
把领域 DTO 转成前端 VO
做字段裁剪和展示适配
处理前端页面维度的查询条件
组装轻量级页面状态
承接前端接口版本差异
```

BFF 不应该做：

```shell
沉淀核心业务规则
直接操作多个业务库
绕过领域服务修改数据
把所有远程接口简单透传给前端
替代订单、用户、支付等领域服务
```

一个常见 BFF 聚合示例：

```java
@Service
public class OrderBffService {

    @DubboReference(version = "1.0.0", group = "order", retries = 0)
    private OrderRpcService orderRpcService;

    @DubboReference(version = "1.0.0", group = "user", retries = 0)
    private UserRpcService userRpcService;

    public OrderDetailVO detail(Long orderId) {
        OrderDTO order = orderRpcService.getById(orderId);
        UserDTO user = userRpcService.getById(order.getUserId());

        return OrderDetailVO.builder()
                .orderId(order.getId())
                .orderStatus(order.getStatus())
                .buyerName(user.getNickname())
                .amount(order.getAmount())
                .build();
    }
}
```

这个逻辑放在 BFF 是合理的，因为它是页面展示聚合。

但如果这里出现了“订单状态流转”“扣库存”“扣余额”“发放权益”等规则，就应该下沉到对应领域服务，而不是留在 BFF。

---

# 十一、写接口时要注意超时、重试和幂等

RPC 调用不是本地方法调用，必须把网络不确定性考虑进去。

最容易忽略的是重试。

比如查询接口：

```java
@DubboReference(timeout = 2000, retries = 1)
private UserRpcService userRpcService;
```

查询接口通常可以适当重试。

但写接口要谨慎：

```java
@DubboReference(timeout = 3000, retries = 0)
private OrderRpcService orderRpcService;
```

比如创建订单、扣余额、发放奖励，如果开启重试，而接口没有幂等控制，就可能造成重复写入。

写接口应该优先保证：

```shell
业务幂等号
唯一索引
状态机校验
防重复提交
消息最终一致性
清晰的异常语义
```

Dubbo 的重试能力很强，但不能用它掩盖业务幂等设计。

---

# 十二、接口参数和返回值设计

Dubbo API 契约设计要稳定，不能把数据库实体直接暴露出去。

不推荐：

```java
UserEntity getById(Long userId);
```

推荐：

```java
UserDTO getById(Long userId);
```

复杂查询不要堆很多参数：

```java
List<OrderDTO> query(Long userId, Integer status, Long startTime, Long endTime, Integer pageNo, Integer pageSize);
```

更推荐封装请求对象：

```java
List<OrderDTO> query(OrderQueryRequest request);
```

请求对象示例：

```java
public class OrderQueryRequest implements Serializable {

    private Long userId;

    private Integer status;

    private LocalDateTime startTime;

    private LocalDateTime endTime;

    private Integer pageNo;

    private Integer pageSize;
}
```

返回值也建议统一包装，尤其是需要承载错误码、错误信息、链路追踪信息时：

```java
public class RpcResult<T> implements Serializable {

    private boolean success;

    private String code;

    private String message;

    private T data;
}
```

是否统一包装要看团队规范。

如果内部异常体系和全局治理已经足够成熟，也可以直接返回 DTO；但跨团队协作时，统一结果结构更容易稳定契约。

---

# 十三、Nacos注册中心的作用

在 Dubbo 架构里，Nacos 负责服务注册和服务发现。

Provider 启动后，会向 Nacos 注册类似下面的信息：

```shell
服务名
接口名
版本号
分组
IP
端口
权重
元数据
```

Consumer 不需要知道 Provider 的 IP 和端口，只需要知道接口契约：

```java
@DubboReference(version = "1.0.0", group = "user")
private UserRpcService userRpcService;
```

Consumer 会从 Nacos 获取可用 Provider 列表，然后由 Dubbo 根据负载均衡策略选择一个 Provider 调用。

这带来几个好处：

```shell
服务地址不写死
Provider 可以多实例扩容
实例下线后 Consumer 能感知
结合权重可以做灰度
结合 namespace 可以做环境隔离
结合 group 可以做业务隔离
```

本地联调时可以打开 Nacos 控制台检查：

```shell
服务管理
  ↓
服务列表
  ↓
检查 provider 是否注册成功
  ↓
检查实例 IP 和端口是否正确
```

如果 Consumer 报找不到 Provider，第一件事就是去 Nacos 看服务有没有注册成功。

---

# 十四、常见问题排查

## 1、No provider available

常见原因：

```shell
Provider 没启动
Provider 没注册到 Nacos
Consumer 和 Provider 的 group 不一致
Consumer 和 Provider 的 version 不一致
Nacos namespace 不一致
网络不通
接口全限定名不一致
```

排查顺序：

```shell
先看 Provider 启动日志
再看 Nacos 服务列表
再看 Consumer 配置
再看接口包版本
最后看网络和防火墙
```

## 2、启动时报 check failed

如果 Consumer 启动时 Provider 还没启动，可能会报检查失败。

本地开发可以设置：

```yml
dubbo:
  consumer:
    check: false
```

但这不是根治方案。

生产环境要结合发布顺序、健康检查和灰度策略处理。

## 3、序列化失败

常见原因：

```shell
DTO 没实现 Serializable
字段类型无法序列化
Provider 和 Consumer 依赖的 api 包版本不一致
枚举字段变更不兼容
LocalDateTime 序列化配置不一致
```

API 模块里的 DTO 要尽量保持简单稳定，不要把复杂业务对象直接塞进去。

## 4、调用超时

常见原因：

```shell
Provider 方法执行太慢
数据库慢 SQL
远程服务继续调用远程服务导致链路过长
timeout 设置太小
Provider 线程池耗尽
网络抖动
```

超时不能只靠调大 timeout，应该看真实耗时在哪里。

---

# 十五、项目落地建议

结合这次实战项目，我认为 Dubbo 落地最重要的是下面几条。

第一，先把分层建立起来。

```shell
Gateway 管入口
BFF 管前端场景
领域服务管业务能力
API 模块管服务契约
Nacos 管服务发现
Dubbo 管内部 RPC
```

第二，API 契约必须独立。

```shell
Provider 实现 api
Consumer 依赖 api
DTO 放 api
Entity 不出 service
接口变更要考虑兼容性
```

第三，BFF 不要越界。

```shell
BFF 可以聚合
BFF 可以裁剪字段
BFF 可以适配页面
BFF 不应该沉淀核心业务规则
BFF 不应该直接改多个领域数据
```

第四，RPC 参数要工程化设计。

```shell
请求对象替代散乱参数
DTO 替代 Entity
写接口关闭无脑重试
关键写操作设计幂等
超时要按接口粒度设置
```

第五，治理能力要前置。

```shell
Nacos namespace 区分环境
group/version 管理服务隔离和演进
日志里打印 traceId
异常语义要清晰
Provider 要有健康检查
慢调用要能被监控发现
```

---

# 十六、总结

Dubbo 的核心价值，不只是让 Java 服务之间能够远程调用，而是让微服务之间通过稳定契约进行协作。

在实战项目里，我认为最关键的不是把 `@DubboService` 和 `@DubboReference` 跑起来，而是建立下面这套工程习惯：

```shell
微服务分层清晰
API 契约独立
Provider 只暴露稳定业务能力
Consumer 只依赖 api 模块
Nacos 负责服务发现
BFF 负责前端场景编排
Dubbo RPC 负责内部高效调用
```

当这套结构建立起来以后，新增一个服务、扩展一个接口、替换一个 Provider、扩容一个实例，都会变得更可控。

Dubbo RPC 调通只是第一步。

真正重要的是：让服务之间的边界清晰、契约稳定、调用可治理、问题可定位。

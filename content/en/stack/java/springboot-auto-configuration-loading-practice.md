---
author: "jarvan"
title: "SpringBoot 配置自动加载流程、底层逻辑、相关注解与使用技巧总结"
date: 2024-12-02
description: "系统整理 SpringBoot 配置如何自动加载、自动配置的底层逻辑、核心注解、配置优先级、条件装配、配置绑定以及项目中的排查和使用技巧。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👤
tags:
- java
- springboot
- configuration
- autoconfiguration
categories:
- java
---

# SpringBoot 配置自动加载是什么

SpringBoot 最大的特点之一，就是“约定大于配置”和“自动配置”。

以前使用 Spring MVC、MyBatis、Redis、Web 容器时，经常需要写大量 XML 或 Java Config。

SpringBoot 做的事情是：

```shell
根据项目依赖
        ↓
判断当前环境中有哪些类
        ↓
读取 application.yml / application.properties 配置
        ↓
按条件自动创建 Bean
        ↓
把这些 Bean 放进 Spring 容器
```

简单理解：

```shell
你引入了什么 starter
SpringBoot 就猜测你大概率要用什么能力
然后根据条件帮你配置好默认 Bean
```

例如：

```shell
引入 spring-boot-starter-web
        ↓
自动配置 Tomcat、Spring MVC、Jackson、DispatcherServlet

引入 spring-boot-starter-data-redis
        ↓
自动配置 RedisConnectionFactory、RedisTemplate

引入 mybatis-spring-boot-starter
        ↓
自动配置 SqlSessionFactory、Mapper 扫描
```

---

# 为什么 SpringBoot 能做到自动配置

核心原因是 SpringBoot 在启动时会加载一批自动配置类。

这些自动配置类不是全部生效，而是通过条件注解判断：

```shell
类路径里有没有某个类？
配置文件里有没有开启某个配置？
容器里是否已经存在某个 Bean？
当前是不是 Web 环境？
```

满足条件才创建 Bean。

所以 SpringBoot 自动配置不是“无脑创建”，而是“按条件装配”。

---

# 一、启动入口 @SpringBootApplication

SpringBoot 项目一般从这个注解开始：

```java
@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

`@SpringBootApplication` 是一个组合注解。

它主要包含：

```shell
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

可以理解为：

```shell
@SpringBootApplication
        ↓
声明这是 SpringBoot 配置类
        ↓
开启自动配置
        ↓
开启组件扫描
```

---

# 二、@SpringBootConfiguration

`@SpringBootConfiguration` 本质上还是 `@Configuration`。

作用是告诉 Spring：

```shell
当前类是一个配置类
可以在里面声明 Bean
```

例如：

```java
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

它和普通 `@Configuration` 的区别不大，更多是 SpringBoot 语义上的标识。

---

# 三、@ComponentScan

`@ComponentScan` 负责扫描组件。

常见注解都会被扫描进 Spring 容器：

```shell
@Component
@Controller
@RestController
@Service
@Repository
@Configuration
```

默认扫描范围：

```shell
启动类所在包及其子包
```

例如启动类在：

```shell
com.jarvan.demo.DemoApplication
```

那么默认扫描：

```shell
com.jarvan.demo
com.jarvan.demo.controller
com.jarvan.demo.service
com.jarvan.demo.mapper
```

注意：

```shell
如果类放在启动类包外面，默认扫描不到。
```

这也是很多 Bean 找不到的常见原因。

---

# 四、@EnableAutoConfiguration

`@EnableAutoConfiguration` 是自动配置的核心入口。

它的作用：

```shell
开启 SpringBoot 自动配置机制
```

底层会通过 `AutoConfigurationImportSelector` 找到需要加载的自动配置类。

可以理解成：

```shell
@EnableAutoConfiguration
        ↓
AutoConfigurationImportSelector
        ↓
读取自动配置类清单
        ↓
按条件过滤
        ↓
导入符合条件的自动配置类
```

---

# 自动配置类从哪里来

不同 SpringBoot 版本加载位置略有差异。

SpringBoot 2.x 常见位置：

```shell
META-INF/spring.factories
```

里面会有类似配置：

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration
```

SpringBoot 3.x 常见位置：

```shell
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

里面是自动配置类列表：

```shell
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration
```

这些类不是全部生效，而是继续通过条件注解判断。

---

# 自动配置加载流程

整体流程可以概括为：

```shell
启动 main 方法
        ↓
SpringApplication.run()
        ↓
创建 SpringApplication
        ↓
准备 Environment
        ↓
加载 application.yml / properties
        ↓
创建 ApplicationContext
        ↓
执行 BeanDefinition 加载
        ↓
@SpringBootApplication 生效
        ↓
@EnableAutoConfiguration 导入自动配置类
        ↓
条件注解决定哪些配置生效
        ↓
创建 Bean
        ↓
执行 Bean 生命周期
        ↓
启动完成
```

再简单一点：

```shell
读配置
扫组件
找自动配置类
条件过滤
创建 Bean
启动容器
```

---

# 条件装配注解

SpringBoot 自动配置的关键是条件装配。

常见条件注解：

```shell
@ConditionalOnClass
@ConditionalOnMissingClass
@ConditionalOnBean
@ConditionalOnMissingBean
@ConditionalOnProperty
@ConditionalOnWebApplication
@ConditionalOnNotWebApplication
@ConditionalOnResource
```

## 1、@ConditionalOnClass

当 classpath 中存在某个类时，配置才生效。

例如：

```java
@ConditionalOnClass(RedisTemplate.class)
public class RedisAutoConfiguration {
}
```

含义：

```shell
项目里有 RedisTemplate，说明你可能要用 Redis，于是 Redis 自动配置可以生效。
```

## 2、@ConditionalOnMissingBean

当容器中没有某个 Bean 时，才创建默认 Bean。

例如：

```java
@Bean
@ConditionalOnMissingBean
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
```

含义：

```shell
如果用户没有自己定义 ObjectMapper，SpringBoot 就提供一个默认的。
如果用户自己定义了，就优先用用户的。
```

这就是 SpringBoot “默认配置 + 用户覆盖” 的核心逻辑。

## 3、@ConditionalOnProperty

根据配置项决定是否生效。

例如：

```java
@ConditionalOnProperty(prefix = "app.audit", name = "enabled", havingValue = "true")
public class AuditAutoConfiguration {
}
```

配置：

```yml
app:
  audit:
    enabled: true
```

只有配置打开时，自动配置才生效。

---

# 配置文件加载

SpringBoot 默认支持：

```shell
application.properties
application.yml
application.yaml
```

常见路径：

```shell
classpath:/application.yml
classpath:/config/application.yml
file:./application.yml
file:./config/application.yml
```

示例：

```yml
server:
  port: 8080

spring:
  application:
    name: user-service
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/demo
    username: root
    password: root
```

SpringBoot 启动时会把配置加载到 `Environment` 中。

业务代码、自动配置类、条件注解都可以从 `Environment` 读取配置。

---

# 配置优先级

SpringBoot 配置有优先级。

常见优先级可以这样理解：

```shell
命令行参数
        >
环境变量
        >
外部 config 目录配置
        >
外部 application.yml
        >
classpath:/config/application.yml
        >
classpath:/application.yml
        >
默认配置
```

例如命令行：

```shell
java -jar app.jar --server.port=9090
```

会覆盖 `application.yml` 中的：

```yml
server:
  port: 8080
```

所以线上排查配置问题时，要注意：

```shell
最终生效的配置不一定来自项目里的 application.yml。
```

它可能来自启动参数、环境变量、K8s ConfigMap、Nacos、Apollo 等外部配置。

---

# Profile 多环境配置

多环境常用 profile。

例如：

```shell
application.yml
application-dev.yml
application-test.yml
application-prod.yml
```

主配置：

```yml
spring:
  profiles:
    active: dev
```

启动时也可以指定：

```shell
java -jar app.jar --spring.profiles.active=prod
```

不同环境可以放不同配置：

```yml
# application-dev.yml
server:
  port: 8081

# application-prod.yml
server:
  port: 8080
```

注意：

```shell
生产环境不要把敏感配置直接写死在代码仓库里。
```

例如数据库密码、支付密钥、JWT Secret，建议放在配置中心或环境变量中。

---

# @Value 读取配置

简单配置可以用 `@Value`。

```yml
app:
  name: jh-user-service
```

使用：

```java
@Value("${app.name}")
private String appName;
```

如果需要默认值：

```java
@Value("${app.timeout:3000}")
private Integer timeout;
```

`@Value` 适合少量简单字段。

如果配置很多，不建议到处写 `@Value`。

---

# @ConfigurationProperties 配置绑定

复杂配置推荐使用 `@ConfigurationProperties`。

配置：

```yml
jh:
  mq:
    enabled: true
    exchange: order.exchange
    retry-count: 3
    timeout-ms: 5000
```

Java 类：

```java
@Component
@ConfigurationProperties(prefix = "jh.mq")
public class MqProperties {

    private Boolean enabled;
    private String exchange;
    private Integer retryCount;
    private Long timeoutMs;

    // getter setter
}
```

优点：

- 配置集中。
- 类型安全。
- 支持嵌套结构。
- 易于校验。
- 易于单元测试。

如果是自定义 starter，可以配合：

```java
@EnableConfigurationProperties(MqProperties.class)
```

---

# @EnableConfigurationProperties

`@EnableConfigurationProperties` 用于启用配置绑定类。

示例：

```java
@Configuration
@EnableConfigurationProperties(MqProperties.class)
public class MqAutoConfiguration {
}
```

它常用于自动配置类中。

这样即使 `MqProperties` 没有加 `@Component`，也可以被 Spring 管理。

---

# 自动配置类示例

假设我们要做一个审计日志组件。

配置：

```yml
jh:
  audit:
    enabled: true
    topic: audit-log
```

配置类：

```java
@ConfigurationProperties(prefix = "jh.audit")
public class AuditProperties {

    private Boolean enabled = false;
    private String topic = "audit-log";

    // getter setter
}
```

自动配置类：

```java
@Configuration
@EnableConfigurationProperties(AuditProperties.class)
@ConditionalOnProperty(prefix = "jh.audit", name = "enabled", havingValue = "true")
public class AuditAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public AuditService auditService(AuditProperties properties) {
        return new AuditService(properties.getTopic());
    }
}
```

这段逻辑表示：

```shell
如果 jh.audit.enabled=true
        ↓
并且容器中没有 AuditService
        ↓
就创建一个默认 AuditService
```

如果业务系统自己定义了 `AuditService`，SpringBoot 就不会创建默认的。

这就是自动配置中非常重要的扩展思想。

---

# Starter 机制

SpringBoot 的 starter 本质上是依赖聚合 + 自动配置。

例如：

```shell
spring-boot-starter-web
```

会帮你引入：

```shell
Spring MVC
Tomcat
Jackson
Validation
相关自动配置
```

所以我们不用一个个手动引依赖。

自定义 starter 通常包含：

```shell
xxx-spring-boot-starter
xxx-spring-boot-autoconfigure
```

starter 负责引入依赖。

autoconfigure 负责提供自动配置类。

---

# 配置自动加载和 Nacos 的关系

本地 `application.yml` 是 SpringBoot 基础配置来源。

如果接入 Nacos，配置加载会变成：

```shell
启动应用
        ↓
读取 bootstrap / application 基础配置
        ↓
连接 Nacos
        ↓
拉取远程配置
        ↓
合并到 Environment
        ↓
触发 Bean 创建或配置刷新
```

例如：

```yml
spring:
  config:
    import: optional:nacos:user-service.yml
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
      config:
        group: DEFAULT_GROUP
        file-extension: yml
```

注意：

```shell
Nacos 配置不是替代 SpringBoot 自动配置，而是作为配置来源参与 Environment 合并。
```

自动配置类仍然会基于最终的 Environment 判断是否生效。

---

# 常见使用技巧

## 1、优先使用 @ConfigurationProperties

少量配置可以用 `@Value`，复杂配置建议统一放到 Properties 类。

例如：

```java
@ConfigurationProperties(prefix = "payment")
public class PaymentProperties {
    private String gatewayUrl;
    private String appId;
    private Integer timeoutMs;
}
```

这样比在多个类里散落 `@Value` 更容易维护。

## 2、提供默认值

配置类中可以给默认值：

```java
private Integer timeoutMs = 3000;
private Integer retryCount = 3;
```

这样配置缺失时，系统也有合理默认行为。

## 3、用 @ConditionalOnMissingBean 留扩展点

自动配置类中建议使用：

```java
@ConditionalOnMissingBean
```

让业务系统可以覆盖默认实现。

这也是 starter 设计中非常重要的习惯。

## 4、用 @ConditionalOnProperty 做开关

例如：

```yml
jh:
  risk-control:
    enabled: true
```

对应：

```java
@ConditionalOnProperty(prefix = "jh.risk-control", name = "enabled", havingValue = "true")
```

这样可以用配置控制功能是否启用。

## 5、排查自动配置是否生效

可以开启 debug：

```yml
debug: true
```

启动后会输出自动配置报告。

也可以看 Actuator：

```shell
/actuator/conditions
/actuator/configprops
/actuator/env
/actuator/beans
```

这些接口可以帮助确认：

```shell
某个自动配置为什么生效？
某个自动配置为什么没生效？
某个配置最终值是什么？
某个 Bean 是否创建？
```

## 6、不要滥用 @ComponentScan

如果扫描范围过大，可能带来：

```shell
启动变慢
扫描到不该加载的 Bean
测试环境 Bean 冲突
多模块项目依赖混乱
```

建议把启动类放在项目根包下，保持包结构清晰。

## 7、敏感配置不要提交代码仓库

以下配置不要直接写在 Git 中：

```shell
数据库密码
Redis 密码
JWT Secret
支付密钥
短信密钥
第三方接口密钥
```

建议使用：

```shell
环境变量
K8s Secret
Nacos 加密配置
CI/CD 注入
配置中心权限控制
```

---

# 常见问题排查

## 1、为什么 Bean 没有被创建

常见原因：

```shell
类不在扫描路径下
条件注解不满足
配置开关没打开
缺少 starter 依赖
已经存在同类型 Bean
Profile 不对
```

排查方式：

```shell
检查包路径
检查 application.yml
检查 spring.profiles.active
开启 debug=true
查看 /actuator/conditions
```

## 2、为什么配置没有生效

常见原因：

```shell
配置文件位置不对
配置 key 写错
profile 没激活
命令行参数覆盖了配置
环境变量覆盖了配置
Nacos 配置覆盖了本地配置
配置类没有启用
```

排查重点：

```shell
看最终 Environment，而不是只看代码里的 application.yml。
```

## 3、为什么自动配置被覆盖

因为 SpringBoot 很多默认配置使用了：

```java
@ConditionalOnMissingBean
```

当你自己声明了同类型 Bean，默认 Bean 就不会再创建。

这通常是正常现象。

例如你自己定义：

```java
@Bean
public ObjectMapper objectMapper() {
    return new ObjectMapper();
}
```

那么 SpringBoot 默认的 `ObjectMapper` 配置可能就不会按原方式生效。

---

# 总结

SpringBoot 配置自动加载的核心流程可以概括为：

```shell
启动应用
        ↓
加载配置文件到 Environment
        ↓
扫描业务组件
        ↓
读取自动配置类清单
        ↓
按条件注解判断是否生效
        ↓
创建默认 Bean
        ↓
用户自定义 Bean 优先覆盖默认 Bean
```

核心注解：

```shell
@SpringBootApplication
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
@ConfigurationProperties
@EnableConfigurationProperties
@ConditionalOnClass
@ConditionalOnMissingBean
@ConditionalOnProperty
```

项目实践建议：

```shell
复杂配置用 @ConfigurationProperties
自动配置留 @ConditionalOnMissingBean 扩展点
功能开关用 @ConditionalOnProperty
多环境用 profile 管理
敏感配置走环境变量或配置中心
排查问题看 Environment 和自动配置报告
```

理解 SpringBoot 自动配置后，就能从“会用 starter”进一步变成“知道 starter 为什么生效、为什么不生效、如何扩展、如何排查”。

---
author: "jarvan"
title: "SpringCloud微服务-gateway网关之JwtToken鉴权"
date: 2025-01-02
description: "统一路由与转发，鉴权与安全，负载均衡，限流与熔断，日志与监控，协议转换与响应处理"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👤
tags:
- java
categories:

---

# 网关请求流程

```shell
d客户端请求
   ↓
HandlerMapping（路由匹配）
   ↓
WebHandler（核心处理器）
   ↓
Filter 链（核心控制点）
   ↓
NettyRoutingFilter（真正发请求到微服务）
   ↓
微服务返回结果
   ↓
Filter（POST 处理）
   ↓
返回客户端
```

## 1、HandlerMapping（路由映射器）

职责：

```shell
根据请求 URL → 找到匹配的 Route
```

做的事：

- 匹配路由规则（Path、Host、Header等）
- 把 Route 放进上下文
- 交给 WebHandler

## 2、WebHandler（请求处理器）

职责：

```shell
组织整个过滤器链执行
```

做的事：

- 把所有 Filter 按顺序串成链 → 依次执行

## 3、Filter（网关核心）

### Filter 分两阶段:

```shell
PRE（前置） → 请求发到微服务之前
POST（后置） → 微服务返回之后
```

执行顺序：

```shell
PRE 按顺序执行 →
调用微服务 →
POST 反向执行
```

### PRE 阶段可以做什么？

```shell
鉴权（JWT）
限流（Sentinel）
参数校验
日志记录
请求改写
灰度发布
```

> 注意：做的 JWT 拦截，就在这里

### POST 阶段可以做什么？

```shell
统一返回结构
日志记录
异常处理
响应修改
统计耗时
```

### 关键规则

```shell
只要一个 Filter 在 PRE 阶段“拦截”了
→ 后面的 Filter 全部不执行
→ 请求不会到微服务
```

这就是为什么：

> 限流 / 鉴权 一定要在最前面

## 4、NettyRoutingFilter（真正干活的）

职责：

````shell
把请求转发到微服务

执行完它：
```shell
请求才真正出网关 → 到你的 px-app-service / px-balance-service
````

## 完整执行顺序(重点)

```shell
1. HandlerMapping 找到 Route
2. 进入 WebHandler
3. 执行 Filter（PRE 顺序）
4. 执行 NettyRoutingFilter（调用微服务）
5. 微服务返回
6. 执行 Filter（POST 逆序）
7. 返回客户端
```

# 职责边界(必须清晰)

## Gateway 层（px-gateway）

```shell
PRE：
- JWT 校验
- 限流（Sentinel）
- IP 黑名单
- 请求日志

POST：
- 统一返回结构（可选）
```

## BFF 层（px-bff-service）

```shell
- 聚合接口
- 调用多个服务
- 熔断（Sentinel）
```

## 业务服务层

```shell
- 数据处理
- 用户逻辑
- 钱包逻辑
- 游戏逻辑
```

# 路由属性

<font color='cyan'>**网关路由**</font> 对应的 java 类型是<font color='cyan'>**RouteDefinition**</font>，其中常见的属性有：

- id :路由唯一标识
- uri :路由目标地址
- predicates :路由断言，判断请求是否符合当前路由

- filters : 路由过滤器，对请求或响应做特殊处理

# 网关过滤器

## GatewayFilter(本文示例)

路由过滤器,作用于任意指定的路由

自定义 GatewayFilter 比较简单，直接实现GatewayFilter接口即可

```java
@Component
public class JwtAuthFilter implements GlobalFilter, Ordered {

    private static final String SECRET = "your-secret-key-123456";

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        ServerHttpRequest request = exchange.getRequest();

        // 放行登录接口
        String path = request.getURI().getPath();
        if (path.contains("/login")) {
            return chain.filter(exchange);
        }

        String token = request.getHeaders().getFirst("Authorization");

        if (token == null || !token.startsWith("Bearer ")) {
            return unauthorized(exchange);
        }

        try {
            String realToken = token.replace("Bearer ", "");

            Claims claims = Jwts.parserBuilder()
                    .setSigningKey(SECRET.getBytes())
                    .build()
                    .parseClaimsJws(realToken)
                    .getBody();

            // 可选：把 userId 传给下游
            String userId = claims.getSubject();

            ServerHttpRequest newRequest = exchange.getRequest().mutate()
                    .header("X-User-Id", userId)
                    .build();

            return chain.filter(exchange.mutate().request(newRequest).build());

        } catch (Exception e) {
            return unauthorized(exchange);
        }
    }

    private Mono<Void> unauthorized(ServerWebExchange exchange) {
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return -100; // 越小越先执行
    }
}
```

## GlobalFilter

全局过滤器，作用范围是所有路由；

### 第一步：先加依赖（Gateway 模块）

在px_gateway的pom.xml加依赖：

```xml
<!--        jjwt-api-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.11.5</version>
        </dependency>
        <!--        jjwt-impl-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.11.5</version>
        </dependency>
        <!--        jjwt-jackson-->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.11.5</version>
        </dependency>
```

### 第二步：自定义 GlobalFilter

src/main/java/com/example/px_gateway/filter/JwtAuthFilter.java:

```java
package com.example.px_gateway.filter;

import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class JwtAuthFilter implements GlobalFilter, Ordered {

    private static final String SECRET = "your-secret-key-123456";

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        ServerHttpRequest request = exchange.getRequest();

        // 放行登录接口
        String path = request.getURI().getPath();
        if (path.contains("/login")) {
            return chain.filter(exchange);
        }

        String token = request.getHeaders().getFirst("Authorization");

        if (token == null || !token.startsWith("Bearer ")) {
            return unauthorized(exchange);
        }

        try {
            String realToken = token.replace("Bearer ", "");

            Claims claims = Jwts.parserBuilder()
                    .setSigningKey(SECRET.getBytes())
                    .build()
                    .parseClaimsJws(realToken)
                    .getBody();

            // 可选：把 userId 传给下游
            String userId = claims.getSubject();

            ServerHttpRequest newRequest = exchange.getRequest().mutate()
                    .header("X-User-Id", userId)
                    .build();

            return chain.filter(exchange.mutate().request(newRequest).build());

        } catch (Exception e) {
            return unauthorized(exchange);
        }
    }

    private Mono<Void> unauthorized(ServerWebExchange exchange) {
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return -100; // 越小越先执行
    }
}
```

# 项目实践

gateway加GlobalFilter,用于解析JwtToken

## 先加依赖（Gateway 模块）

```XML
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
</dependency>
```

## 写 GlobalFilter

src/main/java/com/example/px_gateway/filter/JwtAuthFilter.java:

```java
package com.example.px_gateway.filter;

import com.example.px_gateway.config.JwtProperties;
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

@Component
public class JwtAuthFilter implements GlobalFilter, Ordered {

    private final JwtProperties jwtProperties;


    public JwtAuthFilter(JwtProperties jwtProperties) {
        this.jwtProperties = jwtProperties;
    }

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();

        // 放行登录接口
        String path = request.getURI().getPath();
        if (jwtProperties.getWhiteList().contains(path)) {
            return chain.filter(exchange);
        }


        String token = request.getHeaders().getFirst("Authorization");

        if (token == null || !token.startsWith("Bearer ")) {
            return unauthorized(exchange);
        }

        try {
            String realToken = token.replace("Bearer ", "");

            Claims claims = Jwts.parserBuilder()
                    .setSigningKey(jwtProperties.getSecret().getBytes())
                    .build()
                    .parseClaimsJws(realToken)
                    .getBody();

            // 可选：把 userId 传给下游
            String userId = claims.getSubject();

            ServerHttpRequest newRequest = exchange.getRequest().mutate()
                    .header("X-User-Id", userId)
                    .build();

            return chain.filter(exchange.mutate().request(newRequest).build());

        } catch (Exception e) {
            return unauthorized(exchange);
        }
    }

    private Mono<Void> unauthorized(ServerWebExchange exchange) {
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return 0; // 越小越先执行
    }
}

```

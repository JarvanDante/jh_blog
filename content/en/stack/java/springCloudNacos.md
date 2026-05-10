---
author: "jarvan"
title: "SpringCloud微服务-nacos部署/配置/使用"
date: 2025-01-03
description: "易于构建云原生应用的动态服务发现、配置管理和服务管理平台"
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

# Nacos核心技术原理

服务注册与发现机制

- 客户端通过REST API注册服务到Nacos服务器
- Nacos维护服务列表，并使用心跳机制检测服务健康状态
- 客户端从Nacos获取可用服务列表，实现服务调用

配置管理机制

- 配置集中存储在Nacos中
- 采用长轮询机制，当配置变更时快速通知所有订阅该配置的服务
- 客户端接收变更通知后自动刷新应用配置，无需重启

# 第一步：px-gateway 加依赖

```XML
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

# 第二步：本地 application.yml 保留 Nacos 连接

```yml
server:
  port: 8080

spring:
  application:
    name: px-gateway

  config:
    import: optional:nacos:px-gateway.yml

  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
      username: nacos
      password: nacos123456
      config:
        group: DEFAULT_GROUP
        file-extension: yml
```

> 注意：之前遇到过 No spring.config.import，所以这里必须加：

```XML
spring:
  config:
    import: optional:nacos:px-gateway.yml
```

# 第三步： 在 Nacos 新建配置

Nacos 控制台：

```shell
配置管理 → 配置列表 → 新建配置
```

填写：

```shell
Data ID: px-gateway.yml
Group: DEFAULT_GROUP
配置格式: YAML
```

内容类似如下：

```yml
spring:
  cloud:
    gateway:
      server:
        webflux:
          routes:
            - id: px-bff-service
              uri: lb://px-bff-service
              predicates:
                - Path=/api/frontend/**

auth:
  jwt:
    secret: your-secret-key-12345678901234567890
    white-list:
      - /api/frontend/login
      - /api/frontend/register
      - /api/frontend/captcha
```

![/images/docImages/nacos.png](/images/docImages/nacos.png)
![/images/docImages/nacos2.png](/images/docImages/nacos2.png)
用的是新版 Gateway，要继续用：
`spring.cloud.gateway.server.webflux.routes`
不要再用旧的：
`spring.cloud.gateway.routes`

# 第四步：Gateway 代码读取白名单/JWT 配置

建配置类：
src/main/java/com/example/px_gateway/config/JwtProperties.java:

```java
package com.example.px_gateway.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

@Component
@ConfigurationProperties(prefix = "auth.jwt")
public class JwtProperties {
    private String secret;
    private List<String> whiteList = new ArrayList<>();

    public String getSecret() {
        return secret;
    }

    public void setSecret(String secret) {
        this.secret = secret;
    }

    public List<String> getWhiteList() {
        return whiteList;
    }

    public void setWhiteList(List<String> whiteList) {
        this.whiteList = whiteList;
    }
}
```

在 GlobalFilter 里注入：

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

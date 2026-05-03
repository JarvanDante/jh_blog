---
author: "jarvan"
title: "SpringCloud微服务-sentinel部署/配置/使用"
date: 2025-01-04
description: "面向分布式、多语言异构化服务架构的流量治理组件，主要以流量为切入点，从流量路由、流量控制、流量整形、熔断降级、系统自适应过载保护、热点流量防护等多个维度来帮助开发者保障微服务的稳定性"
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

# Sentinel

阿里巴巴的流量控制组件,官网：`https://sentinelguard.io/zh-cn/index.html`
多版本下载地址：
`https://github.com/alibaba/Sentinel/releases`

## 雪崩问题:

微服务链路中的某个服务故障，引起整个链路中的所有微服务都不可用。

解决方案：

### 一、请求限流

限制访问微服务的请求并发量，避免服务因流量激增出现故障。

### 二、线程隔离

也叫舱壁模式，模拟船舱隔板的防水原理。通过限定每个业务能使用的线程数量而将故障业务隔离，避免故障扩散。

### 三、服务熔断

由断路器统计请求的异常比例或慢调用比例，如果超出决阈值则会熔断该业务，则拦截接口的请求。熔断期间，所有请求快速失败，倒全都走+逻辑。

### 四、失败处理

定义fallback逻辑，让业务失败时不再抛出异常，而是返回默认数据或友好提示。

## 簇点链路：

就是单机调用链路。是一次请求进入服务后经过的每一个被Sentinel监控的资源链。默认Sentinel会监控SpringMVC的第一个Endpoint（http接口）。限流、熔断等都是针对簇点链路中的资源设置的。而资源名默认就是接口的请求路径：

## 一、安装Sentinel

镜像：

```shell
From bladex/sentinel-dashboard:1.8.8
```

我本地是docker-compose,所以docker-compose.yml

```yml
sentinel:
  build: ./sentinel
  container_name: sentinel
  ports:
    - "8858:8858"
  environment:
    - JAVA_OPTS=-Dserver.port=8858 -Dsentinel.dashboard.auth.username=sentinel -Dsentinel.dashboard.auth.password=sentinel
  restart: always
  networks:
    jarven:
      ipv4_address: 172.19.0.45
```

## 二、SpringBoot集成Sentinel

### 第一步：gateway中添加依赖

```XML
<!--        gateway-sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
        </dependency>
```

### 第二步：其它微服务添加依赖

```XML
<!--        sentinel-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-apache-dubbo3-adapter</artifactId>
            <version>1.8.9</version>
        </dependency>
```

### 第三步：gateway和其它微服务中添加sentinel的配置信息

```YMAL
spring:
    cloud:
        sentinel:
            transport:
                dashboard: 127.0.0.1:8858
                port: 8721
```

### 第四步：重启服务请求接口看控制台

![/images/docImages/sentinel.png](/images/docImages/sentinel.png)

## 限流测试-请求限流

### 接口配置流控(QPS/线程)

`在Sentinel控制台->簇点链路->选择接口/user/user-info->流控`
![/images/docImages/sentinel2.png](/images/docImages/sentinel2.png)

### jmeter设置(QPS/线程)

![/images/docImages/sentinel3.png](/images/docImages/sentinel3.png)

### 测试结果:1/2请求返回429

![/images/docImages/sentinel4.png](/images/docImages/sentinel4.png)
![/images/docImages/sentinel5.png](/images/docImages/sentinel5.png)
![/images/docImages/sentinel6.png](/images/docImages/sentinel6.png)

## 限流测试-线程隔离

与限流类似，把设置`并发线程数`对应的阈值即可

```XML
server:
  port: 8081
  #可以模拟限流
  tomcat:
    threads:
      max: 5
    accept-count: 5
    max-connections: 10
```

## 熔断测试

与限流类似

![/images/docImages/sentinel7.png](/images/docImages/sentinel7.png)

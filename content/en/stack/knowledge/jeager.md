---
author: "Karson"
title: "Jeager链路追踪系统"
date: 2023-05-20
description: "Jaeger是一个开源的分布式追踪系统，用于监控和诊断微服务架构中的应用程序。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- knowledge
categories:
- jeager
---

## 概述
Jaeger 是 `Uber` 开发并开源的一款 <font color='cyan'>**分布式链路追踪系统**</font> ，兼容 OpenTracing API。
### 适用场景：
+ <font color='cyan'>**分布式跟踪信息传递**</font>
+ <font color='cyan'>**分布式事务监控**</font>
+ <font color='cyan'>**问题分析**</font>
+ <font color='cyan'>**依赖分析**</font>
+ <font color='cyan'>**性能优化**</font>



## 特性
### 高扩展性
Jaeger后端的设计没有单点故障，可以根据业务需求进行扩展。
### 原生支持 OpenTracing
Jaeger后端，Web UI和工具库已完全设计为支持OpenTracing标准:
+ 通过跨度引用将迹线表示为 <font color='cyan'>**有向无环图**</font>（不仅是树）；
+ 支持强类型的跨度 <font color='cyan'>**标签 和 结构化日志**</font> baggage
+ 支持通用的分布式 <font color='cyan'>**上下文传播机制**</font>

### 多存储后端
+ Jaeger支持两个流行的开源 <font color='cyan'>**NoSQL数据库**</font> 作为跟踪存储后端：<font color='cyan'>**Cassandra 3.4+**</font> 和 <font color='cyan'>**Elasticsearch 5.x / 6.x / 7.x**</font>。
+ Jaeger还附带了一个简单的内存存储区，用于测试设置。

### 现代化的UI
+ Jaeger Web UI是使用流行的开源框架（如React）以Javascript实现的。
+ v1.0中发布了几项性能改进，以允许UI有效处理大量数据，并显示具有成千上万个跨度的跟踪

### 云原生部署
+ Jaeger后端作为Docker映像的集合进行分发。
+ 这些二进制文件支持各种配置方法，包括命令行选项，环境变量和多种格式（yaml，toml等）的配置文件。
+ Kubernetes模板和Helm图表有助于将其部署到Kubernetes集群。

### 可观察性
+ 所有Jaeger后端组件都公开Prometheus指标（也支持其他指标后端）。
+ 使用结构化日志库zap将日志写到标准输出。

## 系统架构
![/images/docImages/jg1.png](/images/docImages/jg1.png)

+ <font color='cyan'>**Jaeger Client**</font>
为不同语言实现了符合 <font color='cyan'>**OpenTracing 标准的 SDK**</font>。应用程序通过 API 写入数据，client library 把 trace 信息按照应用程序指定的采样策略 <font color='cyan'>**传递给 jaeger-agent**</font>。

+ <font color='cyan'>**Agent**</font>
是一个监听在 UDP 端口上 <font color='cyan'>**接收 span 数据的网络守护进程，它会将数据批量发送给 collector**</font>。它被设计成一个基础组件，推荐部署到所有的宿主机上。Agent 将 client library 和 collector 解耦，为 client library 屏蔽了路由和发现 collector 的细节。

+ <font color='cyan'>**Collector**</font>
  <font color='cyan'>**接收 jaeger-agent 发送来的数据**</font>，然后将数据写入后端存储。Collector 被设计成无状态的组件，因此您可以同时运行任意数量的 jaeger-collector。

+ <font color='cyan'>**Data Store**</font>
  <font color='cyan'>**后端存储**</font>被设计成一个可插拔的组件，支持将数据写入  <font color='cyan'>**cassandra、elastic search**</font>。

+ <font color='cyan'>**Query**</font>
  <font color='cyan'>**接收查询请求，然后从后端存储系统中检索 trace**</font> 并通过 UI 进行展示。Query 是无状态的，您可以启动多个实例，把它们部署在 nginx 这样的负载均衡器后面。

## 采样速率
如果所有的请求都开启 Trace 显然会带来比较大的压力，另外，大量的数据也会带来很大存储压力。为此，jaeger 支持设置采样速率，根据系统实际情况设置合适的采样频率。
Jaeger 官方提供了 多种采集策略:
1. <font color='cyan'>**const - 全量采集**</font>，采样率设置 0,1 分别对应打开和关闭
2. <font color='cyan'>**probabilistic - 概率采集**</font>，默认万份之一，0~1之间取值，
3. <font color='cyan'>**rateLimiting - 限速采集**</font>，每秒只能采集一定量的数据
4. <font color='cyan'>**remote - 动态采集策略**</font>，根据当前系统的访问量调节采集策略


## 安装
为方便演示，使用官方推荐的Docker快速启动方式：
```shell
docker run -d --name=jaeger -p6831:6831/udp -p16686:16686 jaegertracing/all-in-one:latest
```

浏览器Web UI： `http://localhost:16686/`
> PS:注意：这个docker镜像封装的jaeger是把数据放在<font color='cyan'>**内存中**</font> 的，仅用于测试，正式使用需指定<font color='cyan'>**后端存储**</font>

docker-compose部署如下：
docker-compose.yml
```yaml
version: "3"
services:
  jeager:
    build: ./jeager
    container_name: jeager
    environment:
      - COLLECTOR_ZIPKIN_HTTP_PORT=9411
    ports:
      - "5775:5775/udp"
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
      - "16686:16686"
      - "14268:14268"
      - "9411:9411"
    volumes:
      - /Users/wangdante/D/kugou/:/var/www/html/
    restart: always
    networks:
      jarven:
        ipv4_address: 172.19.0.20
...
```

![/images/docImages/jg3.png](/images/docImages/jg3.png)

## 使用
现在需要一点数据让我们把页面操作起来。
### 启动一个应用
这个应用是Jaeger仓库中的一个示例
这里我通过源码启动:
```shell
git clone git@github.com:jaegertracing/jaeger.git jaeger
cd jaeger/examples/hotrod
go run ./main.go all
```
同样提供了docker镜像启动方式，参考示例链接。

访问hotrod的webUI：`http://127.0.0.1:8080`

### 发送请求
点击一个按钮，发送下单请求，效果见下图
![/images/docImages/jg4.png](/images/docImages/jg4.png)

分别是 <font color='cyan'>**车牌号**</font>，<font color='cyan'>**预计到达时间**</font>，<font color='cyan'>**请求序列号**</font>，<font color='cyan'>**请求耗时**</font>。

### Jaeger查看服务架构
切换到Jaeger webUI，点击上面的System Architecture --> DAG （directed acyclic graph，有向无环图）
![/images/docImages/jg5.png](/images/docImages/jg5.png)
这就是我们启动的hotrod的微服务架构图，可以看到有几个服务，依赖关系如何。
首先，这里有四个服务（真实的），两个存储(组件模拟的)，其中的数字就是请求调用次数，图中展示了redis调用的次数最多，有点纳闷，回到Jaeger 主页面看看。

### 查看一个trace
![/images/docImages/jg6.png](/images/docImages/jg6.png)
通过前面的架构图可以指定，frontend是最上层服务，通过它应该可以查询到所有的调用记录。
上面的查询结果就是点击了查询按钮之后的效果，在第一个记录中，可以看到3个Errors，后面是一次请求经历的几个服务或阶段。
点进去看详细
![/images/docImages/jg7.png](/images/docImages/jg7.png)

这里提到一个词 Span，这里先介绍什么是Trace，一个 trace 代表了一个事务或者流程在（分布式）系统中的执行过程；
一个 span 代表在分布式系统中完成的单个工作单元，也包含其他 span 的 “引用”，这允许将多个 spans 组合成一个完整的 Trace。

比如说，调用一次查询用户信息的请求 /get_user?uid=1，这里分几步：
+ 第一步，请求到达路由服务，路由服务根据路径调用handler
+ 第二步，handler内部调用service服务
+ 第三步，service根据路由调用其handler
+ 第四步，service-handler内调用数据库

这里就有四个耗时阶段，每个阶段都要耗时，将每个阶段称作是一个span，但其实你会发现上一个阶段会包含下面的所有阶段，这里解释了前面提到的span引用问题；
每个span都有一些元信息：
+ span-id
+ 操作名
+ 开始时间和结束时间，以及耗时
+ tag，用户自定义标签便于查询过滤和理解数据
+ log，记录 Span 内特定时间或事件的日志信息，以及应用程序本身的其他调试或信息输出
+ span context，跨越进程边界，传递到子级 Span 的状态。常在追踪示意图中创建上下文时使用

这里通过上面的图解释一下调用过程：

1. frontend服务接收到外部HTTP GET请求，路由是/dispatch
2. frontend服务调用customer服务的/customer接口
3. customer服务内执行mysql查询，结果返回再返回frontend服务
4. 然后frontend服务向driver服务发起RPC调用，接口是Driver::findNearest
5. driver服务内再调用多次redis，并且可以看到发生错误
6. 然后frontend服务又向route服务发起了多次HTTP GET调用，路由是/route
7. 最后frontend服务返回结果。

点开每个span都可以看到一些细节，包含tag和log；比如发送错误的redis调用，在log部分可以看到更多细节，这都是我们在代码中打的
![/images/docImages/jg8.png](/images/docImages/jg8.png)

### Contextualized logging（情景化日志）
简单说就是，每个span下的log部分都是针对这一次调用产生的记录，这对我们的帮助简直不要太大；以往我们都是直接去tail -f x.log，如果一秒有多个并发请求，我们很难从那么多日志记录中找出问题。

### Span Tags & Logs
这个前面也说了，tag自己定义，kv格式，jaeger中可以用来筛选trace，而log则是记录与这个span这次请求密切相关的事件，一般会带有时间戳，比如redis timeout
>注意：OpenTracing的规范仓库中的semantic data conventions(语义数据约定)描述了一些能够应对多数情况的tag名和log字段，因此我们还是应该参考一下这个约定，少搞特殊

### 链路拆解分析
现在以frontend服务为主角，把以它为root的span都折叠，得到下图
![/images/docImages/jg9.png](/images/docImages/jg9.png)

此图可以让我们更清晰的这个trace过程，哪些span是顺序执行的，哪些是并行的。

顺序执行的span：
+ /customer
+ /driver.DriverService/FindNearest

并行的span:
+ route (每次并发3个请求，图中显示的蓝条在时间线上重合)

> PS:这有多方便不用我多说了，代码中的并发调用看的一清二楚。

### 筛选&排序
查询时筛选是通过service, tag, min/max duration；
这里说一下排序功能，我们可以对搜索结果排序，有几种方式，见下图:
![/images/docImages/jg10.png](/images/docImages/jg10.png)

### 比较多个Trace
在搜索结果中选中至少两个trace，点击右上角的比较按钮
![/images/docImages/jg11.png](/images/docImages/jg11.png)

下图是比较的页面

![/images/docImages/jg12.png](/images/docImages/jg12.png)

下面介绍怎么看对比图；
首先，忽略方块颜色，这些箭头连起来的方块图就是一个trace的调用链，一个方块就是一个span。

一般我们只会选择 <font color='cyan'>**两个相同**</font>的请求来比较，<font color='cyan'>**上图**</font>其实就是两个trace重合之后的图；
如果你选择 <font color='cyan'>**两个不同**</font> 的请求比较，看到的是<font color='cyan'>**下图**</font>
![/images/docImages/jg13.png](/images/docImages/jg13.png)
顺便就讲一下这图，很明显，两个不同路由的请求trace无法重合，其实没有比较意义，但可以解释一下这个颜色，我们放大来看；
![/images/docImages/jg14.png](/images/docImages/jg14.png)
+ 深绿色，表示这个span在只存在于trace-B中，A没有这个span
+ 深红色，表示这个span在只存在于trace-A中，B没有这个span

不过这不是绝对的，因为我看到了这种比较情况
![/images/docImages/jg15.png](/images/docImages/jg15.png)
同样是深绿色，但是两个trace种都有这个span，只是B多于A，我想可以通过数值+8%得出结论，B的span数是14，A是13，算起来就是多了大约8%，看来我们更应该关注这个数值，不过我想颜色的深浅仍然可以代表差距的大小。

下面看另一张图：
![/images/docImages/jg16.png](/images/docImages/jg16.png)
这里还有两种颜色;

+ 浅绿色，表示这个span在trace-B（右边这个）的数量多余trace-A
+ 浅红色，表示这个span在trace-A（左边这个）的数量多于trace-B

最后就是看到的更多的灰色，表示两个trace中都有这个span，且数量一致。

那么到底如何根据比较得出结论？ 看下面这张图
![/images/docImages/jg17.png](/images/docImages/jg17.png)

### 推断：
首先A和B在靠近root span的部分有重合，但大量的child span显示<font color='cyan'>**深红色**</font>表示 <font color='cyan'>**trace-B缺少这些深红色的span**</font>，一般<font color='cyan'>**表示在灰色span处发生了调用失败事件**</font>，导致一连串的span消失。

这样的比较可以在调查事件时提供非常及时和细致的线索。我们可以快速而自信地缩小搜索范围。

如果发生上图这样的事件，我们不应该直接去看深红色的span细节，而应该<font color='cyan'>查看**靠近它们的灰色span的log信息**</font>，以快速定位问题。


## kratos集成jeager
前提docker下已经部署了jeager，之后开放了端口 http://localhost:14268
### 下载安装包
```shell
# 主要是otel ==> go get -u go.opentelemetry.io/otel
$ go get -u go.opentelemetry.io/otel/exporters/jaeger
```

### 添加jeager到grpc服务
文件地址：`balance/internal/server/grpc.go`
```go
package server

import (
...
	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/attribute"
	"go.opentelemetry.io/otel/exporters/jaeger"
	"go.opentelemetry.io/otel/sdk/resource"
	"go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.4.0"
...
)

// 设置全局trace
func initTracer(url string) error {
	// 创建 Jaeger exporter
	exp, err := jaeger.New(jaeger.WithCollectorEndpoint(jaeger.WithEndpoint(url)))
	if err != nil {
		return err
	}
	tp := trace.NewTracerProvider(
		// 将基于父span的采样率设置为100%
		trace.WithSampler(trace.ParentBased(trace.TraceIDRatioBased(1.0))),
		// 始终确保再生成中批量处理
		trace.WithBatcher(exp),
		// 在资源中记录有关此应用程序的信息
		trace.WithResource(resource.NewSchemaless(
			semconv.ServiceNameKey.String("kratos-balance"),
			attribute.String("exporter", "wjb"),
			attribute.Float64("float", 388.66),
		)),
	)
	otel.SetTracerProvider(tp)
	return nil
}
// NewGRPCServer new a gRPC server.
func NewGRPCServer(c *conf.Server, balanceService *service.BalanceService, logger log.Logger) *grpc.Server {
...
	err := initTracer("http://localhost:14268/api/traces")
	if err != nil {
		panic(err)
	}
...
}
```

![/images/docImages/jgr1.png](/images/docImages/jgr1.png)
![/images/docImages/jgr2.png](/images/docImages/jgr2.png)
---
author: "Karson"
title: "OpenTracing"
date: 2023-05-20
description: "分布式调用链标准 OpenTracing"
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


## OpenTracing概念
### OpenTracing
<font color='cyan'>**分布式调用链标准**</font>（OpenTracing）
为了让分布式链路追踪技术有一个行业标准，<font color='cyan'>**CNCF（云原生计算基金会）**</font> 推出了 OpenTracing 项目

OpenTracing 项目是 <font color='cyan'>**与平台厂商无关的链路解决方案**</font>，它不提供具体的实现代码，只制定规范，让接入它的系统能有个一致的协议。

<font color='cyan'>**Jaeger**</font> 、 <font color='cyan'>**Skywalking**</font> 等分布式链路追踪产品都是根据这个标准实现的。

OpenTracing 是一个中立的分布式追踪的 <font color='cyan'>**API 规范**</font>，提供了统一接口方便开发者在自己的服务中集成一种或者多种分布式追踪的实现，使得开发人员能够方便的添加或更换追踪系统的实现。

OpenTracing 可以解决 <font color='cyan'>**不同的分布式追踪系统 API 不兼容的问题**</font> ，各个分布式追踪系统都来实现这套接口。

### OpenTracing数据模型
OpenTracing 的数据模型，主要有三个：
#### 1.Trace：
可以理解为 <font color='cyan'>**一个完整请求链路**</font> ，也可以认为是由 <font color='cyan'>多个 `span` 组成的有向无环图(DAG) </font>
#### 2.Span：
span 代表系统中具有开始时间和执行时长的逻辑运行单元，只要是 <font color='cyan'>**一个完整生命周期的程序访问**</font> 都可以认为是一个 span，比如一次数据库访问，一次方法的调用，一次 MQ 消息的发送等；
每个 **span 包含**: <font color='cyan'>**操作名称**</font> 、<font color='cyan'>**起始时间**</font>、<font color='cyan'>**结束时间**</font>、<font color='cyan'>**阶段标签集合（Span Tag**</font>）、<font color='cyan'>**阶段日志（Span Logs）**</font>、<font color='cyan'>**阶段上下文（SpanContext）**</font>、<font color='cyan'>**引用关系（Reference）**</font>；
#### 3.SpanContext
<font color='cyan'>**Trace 的全局上下文信息**</font>，span 的状态通过 SpanContext 跨越进程边界进行传递，比如包含 trace id，span id，Baggage Items（一个键值对集合）

<font color='cyan'>**TraceId 是这个请求的全局标识。内部的每一次调用就称为一个 Span，每个 Span 都要带上全局的 TraceId，这样才可把全局 TraceId 与每个调用关联起来**</font>。
这个 TraceId 是通过 SpanContext 传输的，既然要传输，显然都要遵循协议来调用。
> 如果我们把传输协议比作车，把 SpanContext 比作货，把 Span 比作路应该会更好理解一些。

## 设计方案
OpenTracing的设计方案包括以下几个核心组件：
### Tracer:
跟踪器是OpenTracing的核心组件，用于创建和管理跟踪span。它还提供了一组标准API，用于在应用程序中嵌入跟踪代码

### API:
API是一组标准化的接口，用于跨语言和跨平台地记录和传递跟踪数据

### Span：
Span是跟踪中的基本单位，用于描述操作的开始和结束。它可以包含事件、标签和日志，并可以与上下文关联。

### Context:
Context是OpenTracing中用于传递上下文信息的关键组件.它可以包含操作ID、Span和其他跟踪数据，用于跨越不同服务和调用之间的信息传递。Context通常需要与RPC系统和HTTP框架进行集成，以便在不同的服务之间传递跟踪信息。

### Carrier:
Carrier是OpenTracing中用于传递跟踪数据的载体。它可以是HTTP请求头、RPC参数、日志文件等等。通过使用标准化的Carrier格式，不同的跟踪系统可以互相兼容并集成。

在设计OpenTracing时，还考虑了可扩展性和可插拔性。跟踪器和跟踪系统可以根据需要进行定制和更改，而不会影响应用程序中的跟踪代码

## 底层原理:
在实现OpenTracing时，通常会使用两个核心组件：Tracer和Span

### Tracer
Tracer是OpenTracing中的核心组件之一，它充当了跟踪系统和应用程序之间的桥梁。
<font color='cyan'>**Tracer主要负责管理和创建Span和Trace**</font>。Span是代表代码执行时间和相关上下文信息的对象，而Trace是包含所有Span的集合。Tracer可以通过将跟踪数据发送到跟踪系统进行存储和分析，以便开发人员能够更好地了解应用程序的行为和性能。
在OpenTracing中，<font color='cyan'>**Tracer对象是 线程安全的**</font>，因此可以在多个线程中使用。通常，Tracer应该在应用程序中只有一个实例。创建Tracer实例时，通常需要提供一些配置选项，例如 跟踪系统的地址、<font color='cyan'>**采样率**</font>等。

### Span
Span是OpenTracing中的另一个核心组件，它代表了一段代码的执行时间和相关的上下文信息。Span通常被嵌套在Trace中，并且可以包含其他Span。

Span通常由以下几个<font color='cyan'>**部分组成**</font>：
#### SpanContext：
包含Span的元数据，例如Span的唯一标识符、父Span的标识符等

#### Operation Name：
Span的名称，用于描述Span代表的操作

#### Start Time：
Span的开始时间，用于记录Span开始执行的时间

#### End Time：
Span的结束时间，用于记录Span执行结束的时间

#### Tags：
用于添加Span的标签，例如Span的名称、开始时间、结束时间等

#### Logs：
用于向Span中添加日志和事件信息，例如请求开始和结束的时间、传输的数据等

Span通常由以下几个<font color='cyan'>**操作组成**</font>：
#### StartSpan：
用于开始一个新的Span，并返回一个Span对象

#### SetTag：
用于添加Span的标签

#### Log：
用于向Span中添加日志和事件信息

使用Tracer和Span可以帮助开发人员更好地了解应用程序的行为和性能。Tracer可以帮助开发人员将应用程序的跟踪数据发送到跟踪系统进行存储和分析，而Span则可以用来描述整个请求的跟踪信息

一个Span可以和一个或者多个Span间存在因果关系。

Open Tracing定义了两种关系：<font color='cyan'>**ChildOf (子节点)**</font> 和 <font color='cyan'>**FollowsFrom (父节点)**</font>

在使用OpenTracing时，开发人员需要在代码中创建Span，并通过Tracer将Span与跟踪操作相关联。
下面是一些使用OpenTracing的python示例代码：
```python
import opentracing  
# 创建一个新的Tracer对象
tracer = opentracing.Tracer()  
# 开始一个新的Spanwith 
tracer.start_active_span('my_operation') as scope:    span = scope.span    
# 在Span中添加一些标签    
span.set_tag('key', 'value')    
# 记录一些日志    
span.log_kv({'event': 'my_event', 'data': 'my_data'})    
# 执行一些操作    
do_something()    
# 结束Span    
scope.close()
```
上面的代码使用Python的OpenTracing库创建了一个新的Tracer对象，并使用start_active_span方法开始一个新的Span。在Span中，开发人员可以添加标签、记录日志、执行操作等。当跟踪操作完成后，Span会被自动关闭并提交给Trace。

### Log的概念
每个Span可以进行多次Logs操作，每一个Logs操作，都需要一个带时间戳的时间名称、以及可选的任意大小的存储结果。

### Tags的概念
每个Span可以有多个键值对（key:value）形式的Tags，Tags是没有时间戳的，支持简单的对Span进行注解和补充。

## OpenTracing理论的开源项目

### Jaeger
Jaeger是一个开源的分布式追踪系统，支持OpenTracing规范，并提供了一个用于<font color='cyan'>**收集、存储和查询**</font> 跟踪数据的平台。
并支持多种语言和平台。Jaeger可以帮助用户了解服务之间的依赖关系，找到性能瓶颈，进行故障排除等。在Jaeger中，开发人员可以使用OpenTracing API创建Span，并将它们与Jaeger进行交互。

Jaeger的设计与OpenTracing的原则非常一致。
Jaeger的架构包括以下组件：
#### Agent：
运行在每个主机上的进程，用于 <font color='cyan'>**接收Span数据并将其发送到Collector**</font>

#### Collector：
<font color='cyan'>收集Agent发送的Span数据</font> ，并存储在数据库中

#### Query：
提供一个Web界面，用于<font color='cyan'>**查询和分析**</font> 存储在数据库中的<font color='cyan'>Span数据</font>

#### Storage：
用于 <font color='cyan'>存储Span数据的后端存储</font>，支持Cassandra、Elasticsearch和Memory三种存储方式

### Zipkin
Zipkin是Twitter开源的分布式跟踪系统，它也实现了OpenTracing规范，并支持多种语言和平台。Zipkin可以帮助用户追踪请求的路径，分析服务之间的依赖关系，以及找到性能瓶颈。

### SkyWalking
SkyWalking是Apache基金会孵化的分布式APM系统，它也支持OpenTracing规范。SkyWalking可以帮助用户追踪分布式系统中的请求，分析服务之间的依赖关系，以及监控服务的性能指标。

## 方案对比
![/images/docImages/otr1.png](/images/docImages/otr1.png)
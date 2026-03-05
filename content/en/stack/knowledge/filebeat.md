---
author: "jarvan"
title: "Filebeat日志采集组件"
date: 2022-05-16
description: "一个轻量级的日志数据收集工具。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- knowledge
categories:

---

## 概述
Filebeat是一个轻量级的日志数据收集工具，属于Elastic公司的Elastic Stack（ELK Stack）生态系统的一部分。

## 目的
从各种来源收集日志数据，将数据发送到 Elasticsearch 、 Logstash 或 其他目标，以便进行 搜索、分析和可视化

## 特点
### 轻量级：
Filebeat是一个 <font color='cyan'>轻量级的代理</font>，对系统资源的<font color='cyan'>消耗非常低</font>。它设计用于<font color='cyan'>高性能和低延迟</font>，可以在各种环境中运行，包括 <font color='cyan'>服务器、容器和虚拟机</font>。
### 多源收集：
Filebeat支持从各种来源收集数据，包括 <font color='cyan'>日志文件、系统日志、Docker容器日志、Windows事件日志</font> 等。它<font color='cyan'>具有多个输入模块</font>，可以轻松配置用于不同数据源的数据收集。
### 模块化：
Filebeat <font color='cyan'>采用模块化的方式组织配置</font>，每个输入类型都可以作为一个模块，<font color='cyan'>易于扩展和配置</font>。这使得添加新的数据源和日志格式变得更加简单。
### 自动发现：
Filebeat支持 <font color='cyan'>自动发现服务</font>，可以在容器化环境中自动识别新的容器和服务，并开始收集其日志数据。
### 安全性：
Filebeat支持 <font color='cyan'>安全传输</font>，可以使用 <font color='cyan'>TLS/SSL加密协议</font>  将数据安全地传输到目标。它还支持基于令牌的身份验证。
### 数据处理：
Filebeat可以对数据进行简单的处理，如 <font color='cyan'>字段分割、字段重命名和数据过滤</font> ，以确保数据适合进一步处理和分析。
### 目标输出：
Filebeat可以 <font color='cyan'>将数据发送到多个目标</font> ，最常见的是将数据发送到 <font color='cyan'>Elasticsearch</font>，以便进行全文搜索和分析。此外，还可以将数据发送到 <font color='cyan'>Logstash、Kafka</font>等目标。
### 实时性：
Filebeat可以以 <font color='cyan'>实时方式收集和传输数据</font>，确保日志数据及时可用于分析和可视化。
### 监控和管理：
Filebeat具有 <font color='cyan'>自身的监控功能</font>，可以监视自身的状态和性能，并与 <font color='cyan'>Elasticsearch、Kibana</font>  等工具集成，用于管理和监控数据收集

![/images/docImages/fb1.png](/images/docImages/fb1.png)

## 采集原理的主要步骤
### 一、数据源检测：
Filebeat首先配置要监视的数据源，这可以是日志文件、系统日志、Docker容器日志、Windows事件日志等。Filebeat可以通过输入模块配置来定义数据源。
### 二、数据收集：
一旦数据源被定义，Filebeat会定期轮询这些数据源，检查是否有新的数据产生。
如果有新数据，Filebeat将读取数据并将其发送到后续处理阶段。
### 三、数据处理：
Filebeat可以对采集到的数据进行一些简单的处理，例如字段分割、字段重命名、数据解析等。这有助于确保数据格式适合进一步的处理和分析。
### 四、数据传输：
采集到的数据将被传输到一个或多个目标位置，通常是Elasticsearch、Logstash或Kafka等。
Filebeat可以配置多个输出目标，以便将数据复制到多个地方以增加冗余或分发数据。
### 五、安全性和可靠性：
Filebeat支持安全传输，可以使用TLS/SSL协议对数据进行加密。
它还具有数据重试机制，以确保数据能够成功传输到目标位置。
### 六、数据目的地：
数据被传输到目标位置后，可以被进一步处理、索引和分析。目标位置通常是Elasticsearch，用于全文搜索和分析，或者是Logstash用于进一步的数据处理和转换，也可以是Kafka等其他消息队列。
### 七、实时性和监控：
Filebeat可以以实时方式监视数据源，确保新数据能够快速传输和处理。
Filebeat还可以与监控工具集成，以监控其自身的性能和状态，并将这些数据发送到监控系统中。

总结，
Filebeat采集原理是通过轮询监视数据源，将新数据采集并发送到目标位置，同时确保数据的安全传输和可靠性。它提供了一种高效且灵活的方式来处理各种类型的日志和事件数据，以便进行后续的分析和可视化。

## 组件组成
Filebeat由两个主要组件组成：inputs 和  harvesters
### harvester
一个harvester负责读取一个单个文件的内容。harvester逐行读取每个文件（一行一行地读取每个文件），并把这些内容发送到输出。
每个文件启动一个harvester。
harvester负责打开和关闭这个文件，这就意味着在harvester运行时文件描述符保持打开状态。
在harvester正在读取文件内容的时候，文件被删除或者重命名了，那么Filebeat会续读这个文件。这就有一个问题了，就是只要负责这个文件的harvester没用关闭，那么磁盘空间就不会释放。默认情况下，Filebeat保存文件打开直到close_inactive到达。
### input
一个input负责管理harvesters，并找到所有要读取的源。
如果input类型是log，则input查找驱动器上与已定义的glob路径匹配的所有文件，并为每个文件启动一个harvester。
每个input都在自己的Go例程中运行。
下面的例子配置Filebeat从所有匹配指定的glob模式的文件中读取行：
```yaml
filebeat.inputs:
- type: log
  paths:


    - /var/log/*.log
    - /var/path2/*.log
```

## Filebeat安装

### 步骤一：下载
下载：`wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.5.1-linux-x86_64.tar.gz`
### 步骤二：解压
解压：`tar -zvxf filebeat-7.5.1-linux-x86_64.tar.gz`
### 步骤三：编辑
在解压出来的目录中，编辑filebeat.yml（缩进比较严格）
+ 复制一份用于备份
+ 清空内容并输入，如下：
```yaml
filebeat.inputs:
- type: log
  ignore_older: 3600
  paths:
   - /data/www/tmewbgl-api/runtime/logs/error/error.log
  fields:
        sys_name: wbglerror
        logtopic: topic-logstash-flow
  processors:
   - drop_fields:
       fields: ["beat.hostname","beat.name","beat.version","input_type","beat","@timestamp","source"]

- type: log
  ignore_older: 3600
  paths:
   - /data/www/tmewbgl-api/runtime/schedule/info/info.log
  fields:
        sys_name: wbglinfo
        logtopic: topic-logstash-flow
  processors:
   - drop_fields:
       fields: ["beat.hostname","beat.name","beat.version","input_type","beat","@timestamp","source"]

output.kafka:
    hosts: ["10.130.64.159:9092"]
    topic: '%{[fields.logtopic]}'
    max_message_bytes: 1000000
    required_acks: 1
    compression: none
```
### 步骤四：启动
启动
检验是否成功：
```shell
./filebeat -e -c filebeat.yml -d "publish"
```
![/images/docImages/fb2.png](/images/docImages/fb2.png)
参数解释：
`./filebeat -e -c filebeat.yml`
+ -c：配置文件位置
+ -path.logs：日志位置
+ -path.data：数据位置
+ -path.home：家位置
+ -e：关闭日志输出
+ -d 选择器：启用对指定选择器的调试。 对于选择器，可以指定逗号分隔的组件列表，也可以使用-d“*”为所有组件启用调试.例如，-d“publish”显示所有“publish”相关的消息。

### 步骤五：后台启动
后台启动
```shell
nohup ./filebeat -e -c filebeat.yml >/dev/null 2>&1 & 
```
将所有标准输出及标准错误输出到/dev/null空设备，即没有任何输出
```shell
nohup ./filebeat -e -c filebeat.yml > filebeat.log &
```

### 步骤六：停止
```shell
ps -ef|grep filebeat,kill -9 pid
```

## 配置多个日志地址：
```yaml
filebeat.inputs:
- type: log 　　#日志目录一
  paths:
    - /var/log/system.log
- type: log 　　#日志目录二
  paths:
    - "/var/log/apache2/*"
```
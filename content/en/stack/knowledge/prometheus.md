---
author: "jarvan"
title: "prometheus"
date: 2023-05-20
description: "Prometheus是由go语言开发的一套开源的监控&报警&时间序列数 据库的组合。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags:
- knowledge
categories:
- prometheus
---

## 概述
prometheus 是一个 <font color='cyan'>**时间序列**</font> 数据库

## prometheus特点
1. <font color='cyan'>**多维度数据模型**</font>
   Prometheus 使用多维度的数据模型来存储时间序列数据
2. <font color='cyan'>**强大的查询语言**</font>
   PromQL 是 Prometheus 的查询语言，支持丰富的操作符和函数，可以实现复杂的数据分析和统计
3. <font color='cyan'>**灵活的警报机制**</font>
   Prometheus 支持基于查询语言的警报规则，可以实现灵活的警报策略和通知方式
4. <font color='cyan'>**易于扩展和集成**</font>
   Prometheus 提供了丰富的客户端库和插件，可以轻松地集成到各种应用和系统中，并支持水平扩展和高可用部署

prometheus整体架构图：
![/images/docImages/pr1.png](/images/docImages/pr1.png)

## 组件介绍
三台服务器
1. prometheus服务器
2. 被监控的服务器
3. grafana服务器

<font color='cyan'>**Prometheus 负责数据收集处理**</font>，<font color='cyan'>**Grafana 负责前台展示数据**</font>。其中采用 Prometheus 中对接的各 Exporter 包含：
+ Node Exporter（核心组件），负责收集所属节点的硬件和操作系统数据，可外挂客制化收集数据文件。它将以容器方式运行在所有节点上;
+ 其他各专属类型Exporter，例如上篇介绍的HPC高性能计算环境下，有针对调度系统专用的Exporter；
+ Alertmanager（可选组件），负责告警，它将以容器方式运行在所有节点上;
+ 其他组件，如cadvisor监控容器等

### Prometheus
![/images/docImages/pr2.png](/images/docImages/pr2.png)

### Node-exporter
![/images/docImages/pr3.png](/images/docImages/pr3.png)

### Grafana
![/images/docImages/pr4.png](/images/docImages/pr4.png)

## docker和docker-compose安装
### 安装docker
```shell
# 安装依赖包
yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加Docker软件包源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 安装Docker CE
yum install docker-ce -y
# 启动
systemctl start docker
# 开机启动
systemctl enable docker
# 查看Docker信息
docker info
```

### 安装docker-compose
```shell
# 直接下载到/usr/local/bin/下即可
curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 这是一个基于python编写的可执行脚本文件，下载后需要赋予可执行权限，运行需要依赖python环境
chmod +x /usr/local/bin/docker-compose
```

### 添加配置文件
```shell
mkdir -p D/mydocker/prometheus/config
cd D/mydocker/prometheus/config
```

### 添加 prometheus.yml 配置文件
```shell
# 添加 prometheus.yml 配置文件
vim prometheus.yml
```
```yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  scrape_timeout: 10s # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["172.19.0.22:9093"]
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "node_down.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    static_configs:
      - targets: ["172.19.0.21:9090"]

  - job_name: "cadvisor"
    static_configs:
      - targets: ["172.19.0.24:8080"]
    # 以下为各节点类型分组
    # 管理节点组
  - job_name: "mgt"
    scrape_interval: 8s
    static_configs:
      - targets: ["172.19.0.25:9100"]

  - job_name: "io"
    scrape_interval: 8s
    static_configs:
      - targets: ["172.19.0.25:9100"]

  - job_name: "login"
    scrape_interval: 8s
    static_configs:
      - targets: ["172.19.0.25:9100"]    
```

### 添加邮件告警配置文件
添加配置文件 alertmanager.yml，配置收发邮件邮箱
`vim alertmanager.yml`
```yml
global:
  smtp_smarthost: 'smtp.163.com:25'　　        #163服务器
  smtp_from: 'xxxxxx@163.com'　　　　　　　　    #你的发邮件的邮箱
  smtp_auth_username: 'xxxxxx@163.com'　　     #你的发邮件的邮箱用户名，也就是你的邮箱
  smtp_auth_password: '*********'　　　　　　　　#发邮件的邮箱密码
  smtp_require_tls: false　　　　　　　　        #不进行tls验证

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10m
  receiver: live-monitoring

receivers:
- name: 'live-monitoring'
  email_configs:
  - to: 'xxxxxxxxxx@qq.com'　　　　　　　　       #收邮件的邮箱
```

### 添加报警规则
添加一个 node_down.yml 为 prometheus targets 监控
`vim node_down.yml`
```shell
groups:
  - name: node_down
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          user: test
        annotations:
          summary: 'Instance {{ $labels.instance }} down'
          description: '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes.'
```

### 编写 docker-compose
`vim docker-compose-monitor.yml`
```yml
...
prometheus:
   build: ./prometheus
   container_name: prometheus
   hostname: prometheus
   ports:
      - "9090:9090"
   volumes:
      - /Users/wangdante/D/kugou/:/var/www/html/
      - /Users/wangdante/D/mydocker/prometheus/config/prometheus.yml:/etc/prometheus/prometheus.yml
      - /Users/wangdante/D/mydocker/prometheus/config/node_down.yml:/etc/prometheus/node_down.yml
   restart: always
   networks:
      jarven:
         ipv4_address: 172.19.0.21
alertmanager:
   build: ./alertmanager
   container_name: alertmanager
   hostname: alertmanager
   ports:
      - "9093:9093"
   volumes:
      - /Users/wangdante/D/mydocker/prometheus/config/alertmanager.yml:/etc/alertmanager/alertmanager.yml
   restart: always
   networks:
      jarven:
         ipv4_address: 172.19.0.22
grafana:
   build: ./grafana
   container_name: grafana
   hostname: grafana
   ports:
      - "3000:3000"
   restart: always
   networks:
      jarven:
         ipv4_address: 172.19.0.23
cadvisor:
   build: ./cadvisor
   container_name: cadvisor
   hostname: cadvisor
   ports:
      - "8080:8080"
   # volumes:
   # - /:/rootfs:ro
   # - /var/run:/var/run:rw
   # - /sys:/sys:ro
   # - /var/lib/docker/:/var/lib/docker:ro
   restart: always
   networks:
      jarven:
         ipv4_address: 172.19.0.24
node-exporter:
   build: ./node-exporter
   container_name: node-exporter
   hostname: node-exporter
   ports:
      - "9100:9100"
   restart: always
   networks:
      jarven:
         ipv4_address: 172.19.0.25
...
```

### 启动 docker-compose
```shell
# 使用docker-composer命令启动yml里配置好的各容器
docker-compose -f /usr/local/src/config/docker-compose-monitor.yml up -d

# 删除容器：
docker-compose -f /usr/local/src/config/docker-compose-monitor.yml down
#重启容器：
docker restart id
```

实际场景示例：
#### 服务器节点列表及资源监控
![/images/docImages/pr5.png](/images/docImages/pr5.png)
#### 用户登录情况监控
![/images/docImages/pr6.png](/images/docImages/pr6.png)
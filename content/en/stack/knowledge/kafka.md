---
author: "Karson"
title: "zookeeper、kafka、kafka-manager安装"
date: 2022-05-16
description: "zookeeper、kafka、kafka-manager安装"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags:
- knowledge
categories:
- kafka
---

## 新建Dockerfile
新建文件夹zookeeper、kafka、kafka-manager，并添加相应的Dockerfile
zookeeper目录下的Dockerfile内容如下：
```dockerfile
FROM wurstmeister/zookeeper
```

kafka目录下的Dockerfile内容如下：
```dockerfile
FROM wurstmeister/kafka
RUN apt-get update && apt-get install -y vim
```
kafka-manager目录下的Dockerfile内容如下：
```dockerfile
FROM sheepkiller/kafka-manager
RUN apt-get update && apt-get install -y vim
```

## 配置docker-compose.yml文件
内容如下：
```dockerfile
version: "3"
services:
    ...
    ...
    ...
  zookeeper:
    build: ./zookeeper
    container_name: zookeeper
    ports:
      - "2181:2181"
    volumes:
     - /Users/wangdante/D/kugou/:/var/www/html/
    restart: always
    networks:
      jarven:
        ipv4_address: 172.19.0.18
  kafka:
    build: ./kafka
    container_name: kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1
      KAFKA_MESSAGE_MAX_BYTES: 2000000
      KAFKA_CREATE_TOPICS: "Topic1:1:3,Topic2:1:1:compact"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
     - /Users/wangdante/D/kugou/:/var/www/html/
    restart: always
    networks:
      jarven:
        ipv4_address: 172.19.0.16
  kafka-manager:
    build: ./kafka-manager
    container_name: kafka-manager
    ports:
      - "9020:9000"
    environment:
      ZK_HOSTS: zookeeper:2181
    volumes:
     - /Users/wangdante/D/kugou/:/var/www/html/
    restart: always
    networks:
      jarven:
        ipv4_address: 172.19.0.17
...
```

## 进入kafka容器修改配置
进入容器：
`docker-compose exec kafka /bin/bash`
修改配置：
```shell
cd /opt/kafka/config/
# ls
connect-console-sink.properties    connect-file-source.properties   consumer.properties  server.properties
connect-console-source.properties  connect-log4j.properties        kraft         tools-log4j.properties
connect-distributed.properties       connect-mirror-maker.properties  log4j.properties     trogdor.conf
connect-file-sink.properties       connect-standalone.properties    producer.properties  zookeeper.properties
```

修改zookeeper的配置文件：zookeeper.properties
```shell
#新建目录 kafkaLogs 和 zooLogs
mkdir /opt/kafka/kafkaLogs
mkdir /opt/kafka/zooLogs
cd /opt/kafka/config/
vim zookeeper.properties
#配置如下：
dataDir=/opt/kafka/zooLogs
clientPort=2181
maxClientCnxns=0
admin.enableServer=false
```

修改kafka的配置文件：server.porperties
```shell
############################# Server Basics #############################
broker.id=0 
############################# Socket Server Settings #############################
//9092:因为9093和下面冲突
listeners=PLAINTEXT://127.0.0.1:9093
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
 
socket.request.max.bytes=104857600
############################# Log Basics #############################
log.dirs=/opt/kafka/kafkaLogs
num.partitions=1
num.recovery.threads.per.data.dir=1
############################# Internal Topic Settings  #############################
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1
############################# Log Retention Policy #############################
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
############################# Zookeeper #############################
#zookeeper.connect=zookeeper:2181
#jarvenwang update
zookeeper.connect=127.0.0.1:2181
zookeeper.connection.timeout.ms=18000
############################# Group Coordinator Settings #############################
group.initial.rebalance.delay.ms=0
#port=9092
#jarvenwang update
port=9093
advertised.host.name=127.0.0.1
message.max.bytes=2000000
#jarvenwang add
advertised.port=9093
```

## 启动zookeeper 和 启动kafka
启动zookeeper:
```shell
cd /opt/kafka/config
zookeeper-server-start.sh zookeeper.properties &
```

启动kafka:
```shell
cd /opt/kafka/config
kafka-server-start.sh server.properties &
```

## 生产消息 和 消费消息
### 创建一个主题：
PS：如果已经创建了主题可以，直接查看:
`kafka-topics.sh --list --zookeeper 127.0.0.1:2181`
前提必须要开启zookeeper的服务==》kafka-topics.sh --create --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 1 --topic test
```shell
kafka-topics.sh --create --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 1 --topic test
#kafka-topics.sh --create --zookeeper 127.0.0.1:2181 --replication-factor 1 --partitions 1 --topic test2
```

### 查看创建的主题：
```shell
$ kafka-topics.sh --list --zookeeper 127.0.0.1:2181
jarvenwang
test
```

### 生产者：
```shell
$ kafka-console-producer.sh --broker-list 127.0.0.1:9093 --topic test
>第一个消息
>第二个消息
>
```
![/images/docImages/kk2.png](/images/docImages/kk2.png)
### 消费者：
```shell
$ kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9093 --topic test --from-beginning
第一个消息
第二个消息
```

![/images/docImages/kk1.png](/images/docImages/kk1.png)

### 查看版本：
```shell
ps -ef|grep '/libs/kafka.\{2,40\}.jar'
```


## 进入kafka-manager容器修改配置
进入容器（可能要尝试多次）
> 提示：qemu: uncaught target signal 11 (Segmentation fault) - core dumped （镜像的问题）
`docker-compose exec kafka-manager /bin/bash`
```shell
[root@65567f1ceed6 kafka-manager-1.3.1.8]# ls
README.md                      bin              lib
RUNNING_PID                    conf             share
application.home_IS_UNDEFINED  hs_err_pid1.log  start-kafka-manager.sh
```

### 修改配置信息zkhosts
修改配置
`vi conf/application.conf`
```shell
# 修改
kafka-manager.zkhosts="172.19.0.18:2181"
# 如果是集群，参考如下
# kafka-manager.zkhosts="10.0.0.50:2181,10.0.0.60:2181,10.0.0.70:2181"
```

### 启动kafka-manager
确保自己本地的ZK已经启动了之后，我们来启动Kafka-manager, kafka-manager 默认的端口是9000
>如果想修改端口：
> -Dhttp.port=8081，指定端口; 如：bin/kafka-manager -Dhttp.port=8081
> -Dconfig.file=conf/application.conf 指定配置文件；如：nohup bin/kafka-manager -Dconfig.file=conf/application.conf -Dhttp.port=8081 >/dev/null 2>&1 &

#### 前台启动
```shell
$ bin/kafka-manager
05:18:59,993 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback.groovy]
05:18:59,995 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Could NOT find resource [logback-test.xml]
...
```
如果要结束进程：
`rm -rf /kafka-manager-1.3.1.8/RUNNING_PID`
![/images/docImages/kfkm.png](/images/docImages/kfkm.png)

#### 后台启动
```shell
$ nohup bin/kafka-manager -Dconfig.file=conf/application.conf >/dev/null 2>&1 &
[1] 3722
```
#### 访问
测试端口是否成功：
nc -zv 一下
`nc -zv 127.0.0.1 9020`
由于9000端口映射的端口为9020
输入地址：`127.0.0.1:9020` 即可访问

启动完毕后可以查看端口是否启动，由于启动过程需要一段时间，端口起来的时间可能会延后
![/images/docImages/kfkm2.png](/images/docImages/kfkm2.png)
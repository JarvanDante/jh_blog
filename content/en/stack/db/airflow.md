---
author: "karson"
title: "airflow调度系统"
date: 2023-05-24
description: "Apache Airflow是一个开源的任务调度和工作流管理平台，它使用Python编写，是一个以编程方式编写，安排和监视工作流的平台。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags:
- db
- categories:

---
# Airflow概念
Apache Airflow是一个开源的任务调度和工作流管理平台，它使用Python编写。Airflow调度器是Airflow的一个核心组件，负责管理和调度任务的执行。

## 1.Airflow调度器的主要功能

1. 任务调度：
Airflow调度器根据预定义的任务调度依赖关系和时间表，确定何时执行每个任务。它可以处理复杂的工作流调度需求，如依赖关系、并发性和重试策略等。

2. 任务执行：
Airflow调度器将任务分配给可用的执行器（如本地执行器、Celery执行器或Kubernetes执行器），并跟踪任务的状态和执行结果。

3. 任务监控和日志记录：
Airflow调度器监控任务的执行状态，并将任务的日志和元数据存储在后端数据库中，以便后续查询和分析。

4. 任务重试和错误处理：
如果任务执行失败，Airflow调度器可以根据预定义的重试策略自动重试任务。它还提供了一套灵活的错误处理机制，可以定义任务失败时的行为。

5. 调度器高可用：
Airflow调度器支持多个调度器实例之间的高可用性配置，以确保调度器的可用性和容错性。

核心概念：
+ Dynamic：Airflow配置需要使用Python，允许动态生产管道。这允许编写可动态、实例化管道的代码。
+ Extensible：轻松定义自己的运算符，执行程序并扩展库，使其适合于您的环境
+ Elegant：Airflow是精简的，使用功能强大的Jinjia模板引擎，将脚本参数化内置于airflow的核心中
+ Scalable：Airflow具有模板块架构，并使用消息队列来安排任意数量的工作任务。

## 2.安装Miniconda
conda概念：
一个开源的包、环境管理器，可用于在同一个机器上切换不同的Python版本的软件包及其依赖，并能在不同的python环境之间切换。
Anaconda包括Conda、Python 以及一大堆安装好的工具包，比如：numpy、pandas等，Miniconda包括Conda、Python。此外没有如此多的工具包，所以选择Miniconda
### 2.1下载
`
anaconda官网：
https://repo.anaconda.com/
`
https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
### 2.2安装
```shell
bash Miniconda3-latest-Linux-x86_64.sh
```
安装过程中，指定安装路径
### 2.3加载环境变量配置文件，使之生效
```shell
source ~/.bashrc
# 退出环境
(base) $conda deactivate
```

### 2.4取消激活base环境
miniconda安装完成后，每次打开终端都会激活其默认的base环境，用以下命令，禁止激活默认base环境
```shell
conda config --set auto_activate_base false 
```
### 2.5创建环境
```shell
$conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
$conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
$conda config --set show_channel_urls yes
```
创建python3.8环境
```shell
conda create --name airflow python=3.8
```
`说明：conda环境管理常用命令`
创建环境：
```shell
conda create -n env_name
```

查看所有环境：
```shell
conda info --envs
```

删除一个环境：
```shell
conda remove -n env_name --all
```

激活airflow环境：
```shell
conda activate airflow
```

退出airflow环境：
```shell
conda deactivate
python -V
```
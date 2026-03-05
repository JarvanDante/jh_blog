---
author: "Karson"
title: "shell知识点"
date: 2021-01-01 00:00:01
description: "shell知识点"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: Karson
authorEmoji: 👻
tags: 
- linux
- categories:

---

## 常用命令

### 系统所有shell版本
`cat /etc/shells`
```shell
ubuntu@k3s:~$ cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/usr/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/usr/bin/dash
/usr/bin/screen
/usr/bin/tmux
```
### 系统家目录
`echo $HOME`
```shell
ubuntu@k3s:~$ echo $HOME
/home/ubunt
```

### 系统命令查找目录
`echo $PATH`
```shell
ubuntu@k3s:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

### 系统默认shell
echo $SHELL
```shell
ubuntu@k3s:~$ echo $SHELL
/bin/bash
```
### 查看当前执行脚本名称
echo $0
```shell
ubuntu@k3s:~$ echo $0
-bash
# 例子：
切换sh
ubuntu@k3s:~$ /bin/sh （切换sh，退出exit）
$ echo $0
/bin/sh
$ exit
ubuntu@k3s:~$
```

### shell脚本编写
`vim hello.sh`
```shell
#!/bin/bash

echo "Hello Shell"
date
whoami
```
`chmod a+x hello.sh`
`./hello.sh`

举例：
```shell
#!/bin/bash

echo "请输入您的姓名："
name=$1
channel=$2
echo "你好，$name,欢迎来到 $channel!"
```
```shell
ubuntu@k3s:~$ ./game.sh 王大大  学习
请输入您的姓名：
你好，王大大,欢迎来到 学习!
```

#### 常用语法符号
+ $# : 传递给脚本或函数的位置参数的个数
+ $? : 上一命令的退出状态码。0 通常表示没有错误，非 0值表示有错误
+ $* : 传递给脚本或函数的位置参数，双引号包围时作为一个整体
+ $@ : 传递给脚本或函数的位置参数
+ $$ : 当前shell进程的进程ID（PID）
+ $! : 最后一个后台命令的进程ID
+ $0 : 当前脚本的名称
+ $1-n : 脚本或函数的位置参数

#### if 语句
```shell
if condition; then
  echo "猜对了"
elif [[ $guess -lt $number]]; then
  echo "小了"
else 
  echo "大了"
fi
```
上面的condition可以是[]，[[]]，或(()):
+ []是最基本的条件测试表达式
+ [[]]是扩展测试命令，提供了比[]更强大的功能
+ (())是数学表达式，支持常见的+-*/数学运算


### .profile/.bashrc
都要存储一些环境变量
#### .bash_profile
在用户登录的时候执行的并且只执行一次
#### .bashrc
每次新打开一个终端或新建一个shell对话执行的
通常.bash_profile里面会调用.bashrc ，所以推荐把环境变量放到.bashrc文件中

>PS : 修改.bashrc配置文件之后要生效
> `source .bashrc`
>ps:不同shell的配置文件不同：
> zsh 的配置文件 .zshrc 文件
> bash 的配置文件 .bashrc 文件
> fish 的配置文件 .config.fish 文件

### /目录下的配置文件 
/etc文件夹下的 profile 和 bashrc 文件

### alias 别名
在 .bashrc 定义别名
`alias la='ls -A'`


### 随机数

echo $((RANDOM * 10 + 1 ))
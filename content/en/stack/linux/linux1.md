---
author: "karson"
title: "基础知识"
date: 2021-01-01 00:00:01
description: "Linux是一种开源的操作系统内核."
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags: 
- linux
- categories:

---

# 命令
格式：
`command` `[-options]` `[parameter]`
+ command: 命令本身
+ -options: [可选，非必填] 命令的一些选项，可以通过选项控制命令的行为细节
+ parameter: [可选，非必填] 命令的参数，多数用于命令的指向目标等
## HOME目录 和 工作目录：
HOME目录 ：每个Linux操作用户在linux系统的个人账户目录，默认在: `/home/用户名`
工作目录 ： 打开命令终端时的目录，默认设置的工作目录就是HOME目录

## ls
`ls` `[-a -l -h]` `[linux路径]`
+ -a 选项，表示all意思，列出全部文件（包括隐藏文件/文件夹）
+ -l 选项，表示以列表示展示内容
+ -h 选项，易查看k,m,g

## cd 和 pwd
`cd`  `[linux路径]`
cd 有参数，去到目录 
cd 无参数，默认回到home目录
`pwd`  
输出当前目录 

特殊路径符：
+ `.`  当前目录
+ `..` 上一级目录 
+ `~`  HOME目录

## mkdir
`mkdir` `[-p]` `linux路径`
-p 表示自动创建不存在的父目录
注意：创建文件夹需要修改权限，请确保操作均在HOME目录内，不要在HOME外操作涉及到权限问题，HOME外无法成功

## touch/cat/more
+ `touch` `linux路径`
```shell
touch test.txt
```
+ `cat` `linux路径`
直接将内容全部显示出来
```shell
cat test.txt
```
+ `more` `linux路径`
```shell
more /etc/services
```
`f` : front翻上一页
`b` : back翻上一页
`空格` ：翻下一页
`q`  ：退出

## cp
`cp` `[-r]` `参数1` `参数2`
+ -r 选项，可选：用于复制文件夹使用，表示递归

## mv
`mv`  `参数1` `参数2`

## rm
语法：`rm` `[-r -f]` `参数1` `参数2` ... `参数N`
+ `-r` 同cp命令一样，-r 选项用于删除文件夹
+ `-f` 表示force，强制删除（不会弹出提示确认信息）
+ `参数1` `参数2` ... `参数N` 一次删除多个，空格隔开

通配符：
+ 符号* --表示通配符，匹配做任意内容
+ test* --表示匹配任何以test开头的内容
+ *test --表示匹配任何以test结尾的内容
+ *test * --表示匹配任何包含test的内容

## which
语法：`which` `查看命令位置`

## find
+ 语法：`find` `起始路径` `-name`  `"被查找文件名"`
```shell
find / -name "*test"
find / -name "*test*"
```
+ 语法：`find` `起始路径` `-size`  `"+|-n[kMG]"`
```shell
find / -size -10k
find / -size +100M
```

## grep
语法：`grep` `[-n]` `关键字`  `"文件路径"`
+ -n ：可选，显示行数
```shell
grep "abc" test.txt
 grep -n "xyz" test.txt
 
```

## wc
语法：`wc` `[-c -m -l -w]` `文件路径`
+ -c ，统计bytes数量
+ -m , 统计字符数量
+ -l ,统计行数
+ -w , 统计单词数量
```shell
wc test.txt 
wc -l test.txt 
```

## 管道符
`|` `左边命令的结果，作为右边的输入`
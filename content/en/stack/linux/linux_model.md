---
author: "Karson"
title: "linux的架构"
date: 2022-01-01 00:00:01
description: "linux的架构"
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

## linux的架构
组成：
1. 内核
2. 系统库
3. shell
4. 应用程序
![/images/docImages/lx.png](/images/docImages/lx.png)
### 内核
组件包括：
+ 设备驱动程序
+ 进程管理
+ 内存管理
+ 文件系统
+ 网络协议系统 

### 系统库
接口和函数包括：
+ C标准库（libc）
+ 数学库（libm）
+ 动态链接库（libdl）
+ 线程库（libpthread）
+ 第三方库（...）

### shell:
是内核kernel的外壳，接受用户输入的命令，传递给操作系统来执行

### 应用程序
软件程序：如 nginx/mysql等

## 发行版
由 内核、应用软件、系统工具、库文件、图形界面、shell、包管理器 组成的操作系统
常风的发行版本：
+ RedHat
+ CentOS
+ KALI(深透)
+ ubuntu(个人桌面)
+ fedora
+ debian
+ alpine(容器化)

## 虚拟机工具
常见的虚拟机工具：
windows上：
+ vmware
+ virtualBox
+ hyper-v
+ Multipass（推荐，只支持ubuntu）
+ parallelsDesktop
+ UTM
+ QEMU

>ps: ubuntu 官网： `https://ubuntu.com/`
### multipass
multipass常见的命令：

#### 安装虚拟机镜像实例：
`multipass launch`
如： `multipass launch -n [实例名] -c [CPU核数] -m [内存大小] -d [磁盘大小]`
#### 查看已经安装的虚拟机实例：
`multipass list/ls`
#### 进入虚拟机实例
`multipass shell vm-name`
如： `multipass shell master`

#### 启动虚拟机实例
`multipass start vm-name`

#### 停止虚拟机实例
`multipass stop vm-name`

## vi/vim
三种模式：
+ 命令模式：
+ 插入模式：
+ 尾行模式：
  ![/images/docImages/lx2.png](/images/docImages/lx2.png)

`i` ：插入
`I` ：行首
`a` : 后面插入
`A` : 行尾
`O` : 上一行插入
`o` : 下一行插入
移动：
`H` : 左
`J` : 下
`K` : 上
`L` : 右
复制粘贴：
`yy` : 复制一行内容，2yy 复制2次
`p` : 粘贴复制或删除的一行内容，3p复制3次
`dd` : 删除一行内容

`ctrl+f`: 前一页
`ctrl+b`：后一页
`ctrl+u`：上一半页
`ctrl+d`：下一半页
`G`：文章最后一行
`gg`：文章第一行
`100G`：文章第一行 或 `:100`

查看：
`:/内容`：严格大小写，`\c`不会区分大小写 或`:set ic`

替换：
`:n1,n2s/old/new/g`：`:40,50s/替换/被替换/g` ： `/s` 替换，`/g` 全局
`u` : 撤销

vi .vimrc文件：保存设置
`:set nu` ：行数
`:syntx on` ： 忽略大小写

## 常用命令
ls
cat hello.txt

`ls -l` : 详情
`ls -i` : inode ，i节点
`ls -a` 隐藏文件
`ls -h -t -r -l` ：大小 、时间、排序

`ln` : (link)创建链接文件
`ln -s` : 软链接
如： `ln -s hello.txt link.txt`

### 文件权限：
lrwxrwxrwx:
前组：文件所有者的权限(user)
中组：同组用户的权限(group)
后组：其他用户的权限(other)
r （4）: 可读
w （2）：可写
x （1）：可执行

#### chmod
chmod 修改权限
添加权限 + 号
`chmod +x 1.txt`
`chmod +rw 1.txt`

删除权限 - 号
`chmod -x 1.txt`

只有文件的所有者添加权限 + 号
`chmod u+x 1.txt`

只有文件的所有者或组成员添加权限 + 号
`chmod ug+x 1.txt`

#### touch 
#### pwd
#### cd -
./
..

#### cp
cp 1.txt 2.txt
cp -r a b  目录

#### mv
mv 1.txt 2.txt

#### rm
rm file.txt
rm -r 删除目录递归的删除
#### mkdir
mkdir a
mkdir -p a/b/c 层级目录

#### du
查看文件的大小
也可以查看目录多文件






## 目录

### /bin
基本命令 + 二进行可执行 文件

### /etc
软件
配置文件 ：nginx 、 mysql 的配置文件 

### /home
家目录，进入系统的默认目录

### /dev 
设备文件 

### /lib
系统库文件 

### /opt
可选的第三方软件包

### /tmp
临时文件

### /usr
用户程序

### /var
可变文件，如日志

### /boot
启动加载器文件

### /proc
进程信息

### /sys
系统文件
---
author: "jarvan"
title: "时区"
date: 2021-01-01 00:00:01
description: "Linux系统时区的修改"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags: 
- linux
- categories:

---

## 方法一：使用命令修改时区
### 1. 打开终端
### 2. 运行命令
```shell
sudo timedatectl set-timezone <时区>
```
将<时区>替换为你要设置的时区，例如 **Asia/Shanghai**

### 3.输入用户密码确认修改
### 4.通过运行命令timedatectl可以验证时区是否已成功修改

## 方法二：通过编辑/etc/timezone文件修改时区

### 1. 打开终端

### 2. 运行命令
```shell
sudo nano /etc/timezone
```
使用合适的文本编辑器打开/etc/timezone文件
### 3. 在文件中输入你要设置的时区，例如Asia/Shanghai
### 4. 保存并关闭文件

### 5. 运行命令
```shell
sudo dpkg-reconfigure -f noninteractive tzdata
```
来更新时区配置
### 6. 输入用户密码确认修改
### 7. 通过运行命令date或者timedatectl可以验证时区是否已成功修改

## 方法三：通过软链接修改时区

### 1. 打开终端
### 2. 运行命令cd /etc进入/etc目录
### 3. 运行命令
```shell
sudo ln -sf /usr/knowledge/zoneinfo/<时区> localtime
```
将<时区>替换为你要设置的时区，例如Asia/Shanghai
### 4. 输入用户密码确认修改
### 5. 通过运行命令date或者timedatectl可以验证时区是否已成功修改

注意：以上方法需要使用管理员权限，确保在修改时区时谨慎操作，避免因设置错误导致系统时间混乱。

## 时间同步
```shell
mount /dev/sr0 /mnt
yum install ntpdate -y
ntpdate cn.ntp.org.cn
```

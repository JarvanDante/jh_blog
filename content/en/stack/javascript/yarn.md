---
author: "karson"
title: "yarn包管理工具"
date: 2022-07-02
description: "yarn是一个包管理工具，它的作用是帮助开发者管理项目中的依赖包。使用yarn可以方便地安装、更新、卸载、查看依赖包。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags:
- javascript
- categories:

---
## 支持Node版本
支持的node 版本： `^4.8.0` || `^5.7.0` || `^6.2.2` || `>=8.0.0`

## yarn的优点
1. 速度快：
yarn使用并行安装和缓存机制，可以大大提高安装依赖包的速度。

2. 稳定性好：
yarn使用lockfile来锁定依赖包的版本，保证了项目的稳定性。

3. 安全性高：
yarn使用yarn.lock文件来锁定依赖包的版本，避免了由于依赖包版本不一致导致的安全问题。

4. 易于维护：
yarn提供了一些方便的命令，如`yarn upgrade`，`yarn outdated`等，可以方便地更新和查看依赖包。

5. 支持离线安装：
yarn可以将依赖包缓存到本地，支持离线安装，避免了网络不好的情况下安装依赖包失败的问题。
## yarn 安装和使用
### 安装： 
可以用yarn代替npm，yarn更快、对开发者更友好。但是在用yarn之前需要先通过npm安装它。
` npm i yarn -g` 或 `npm install --global yarn`
### 命令使用：
1. 初始化项目：<font color="lightgreen">yarn init</font> 该命令会在当前目录下创建一个新的package.json文件，用于管理项目的依赖和配置。

2. 安装依赖：<font color="lightgreen">yarn install</font> 该命令会根据package.json文件中的依赖列表，下载并安装项目所需的依赖包。

3. 添加依赖：<font color="lightgreen">yarn add [package]</font> 该命令会将指定的依赖包添加到项目中，并更新package.json文件的依赖列表。

4. 更新依赖：<font color="lightgreen">yarn upgrade [package]</font> 该命令会将指定的依赖包更新到最新版本，并更新package.json文件的依赖列表。

5. 移除依赖：<font color="lightgreen">yarn remove [package]</font> 该命令会将指定的依赖包从项目中移除，并更新package.json文件的依赖列表。

6. 运行脚本：<font color="lightgreen">yarn run [script]</font> 该命令会运行项目中定义的脚本命令，例如启动开发服务器、打包代码等。

7. 清理缓存：<font color="lightgreen">yarn cache clean</font> 该命令会清理yarn的缓存，可以解决一些依赖包版本冲突或缓存问题。

## yarn改国内镜像
1.查看镜像源：
`yarn config get registry`
如果显示：
` https://registry.yarnpkg.com`
则表示不是国内镜像源。可以通过以下步骤设置
2.设置为淘宝镜像源：
` yarn config set registry https://registry.npm.taobao.org/`

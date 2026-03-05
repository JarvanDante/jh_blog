---
author: "jarvan"
title: "debug相关"
date: 2022-12-03 00:00:01
description: "debug调试"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: jarvan
authorEmoji: 👻
tags: 
- golang
categories:
- go




---

## 常见问题解决
### version of Delve is too old for Go
>问题：
> goland开启debug一直connected的问题 undefined behavior - version of Delve is too old for Go
>解决：
> golang的调试器是delve，Goland内置有一个delve，这个问题表面上看，就是内置delve的版本过低了
> 步骤一：升级到最新版本的dlvgo install github.com/go-delve/delve/cmd/dlv@latest
> 步骤二：执行完上面的命令，其实是安装到了GOBIN目录下，我们执行go env 查看go的配置参数，找到gobin，并将dlv的路径添加到'help'->'Edit Custom Properties'
> 步骤三：在文件中添加以下配置dlv.path:/Users/zz/app/goApp/bin/dlv 重启goland即可解决问题
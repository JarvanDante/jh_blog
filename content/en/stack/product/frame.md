---
author: "karson"
title: "Figma-图层和编组类型"
date: 2025-02-05
description: "在Figma中，图层和编组是非常核心的概念，它们帮助设计师组织和管理设计元素"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags:
- 产品
categories:

---
## 左侧的图层列表
figma的画布中可以填入不同元素，每类元素会映射在左侧的图层列表中
选择相应的图层可以点击左侧图层列表
```text
alt + 左键 （拖拽复制）
空格 + 左键（不影响包含关系，如：一半包含）
```
<font color='cyan'>**图层包含上下的叠加关系，以及编组的包含层级关系**</font>
+ 上下叠加关系：
左侧图层列表按上下顺序优先显示，越往上显示优先级越高。show
+ 包含层级关系：
元素置入到frame里面，一起控制

## 多种编组类型

### 基础编组
#### 编辑Group
将多个图层成组作为一个整体进行排版、管理
```text
Command/Ctrl + G
```
子元素选择方法：
+ 双击
+ 左侧
+ 成组后可以用 `Com/Ctrl + 左键`选择下级图层


#### 画板/画框Frame
用于放置、展示图形元素的"视窗、容器"
Figma中创建界面绘制区域使用Frame实现
```text
Command/Ctrl + Alt + G
```
Group和Frame的区别：
+ Group会放大元素，Frame不会放大，可适配
#### 分区Section
用于容纳多个画板的最上级编组
常用于将多个画板导出为一张大图
```text
Command/Ctrl + S
```

<font color='cyan'>**图层可以进行命名、锁定、隐藏**</font>
命名Rename ：
```text
Command/Ctrl + R
```
锁定Lock：
```text
Command/Ctrl + Shift + L
```

隐藏Hide：
```text
Command/Ctrl + Shift + H
```
<font color='cyan'>**功能操作失败往往和图层有直接联系**</font>

<font color='cyan'>**碰到操作失败优先从图层处进行排查**</font>
### 特殊编组
#### 自动布局Autolayout

#### 矢量布尔Boolean operation

#### 界面组件Commponent





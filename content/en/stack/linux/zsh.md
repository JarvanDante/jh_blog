---
author: "karson"
title: "zsh下安装主题Powerlevel9k"
date: 2022-01-02 00:00:01
description: "Powerlevel9k主题可以用于 vanilla ZSH 或 ZSH 框架，如 oh-my-ZSH、 Prezto、 Antigen 等等。"
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: karson
authorEmoji: 👻
tags: 
- linux
- categories:
#  -
#image: images/feature3/go.png
---


## 安装 Powerlevel9k


###  步骤一：安装 Powerline 字体库
  ```shell
$git clone https://github.com/powerline/fonts.git
  $./install.sh
  Copying fonts...
  Powerline fonts installed to /Users/wangdante/Library/Fonts  （字段安装的目录）
```

`brew tap` 更新第三方库，才能用 homebrew 安裝字型。执行过可以跳过
brew tap caskroom/fonts (旧版本，新版本用：brew tap homebrew/cask)
```shell
# 获取homebrew-cask-completion
brew install brew-cask-completion
# 获取homebrew-cask-fonts
brew tap homebrew/cask-fonts
# 获取homebrew-cask-drivers
brew tap homebrew/cask-drivers

# 查看满足nerd格式的字段有哪些，选择下载
brew search nerd
==> Formulae
container-diff                                       nerdctl

==> Casks
font-go-mono-nerd-font
font-sauce-code-pro-nerd-font ✔
...
# 下载/安裝指令
brew install homebrew/cask-fonts/font-sauce-code-pro-nerd-font

```

###  步骤二：安装 Powerlevel9k 主题
```shell
# 安装P9k
$git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k

# 配置 oh-my-zsh ：编辑 ~/.zshrc 修改zsh的主题
ZSH_THEME="powerlevel9k/powerlevel9k"

# 显示在左边的提示元素（分段位于括号中并以空格隔开）
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(context
# 显示在右边的提示元素（分段）
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status)
# 左侧提示符是否显示两行（光标显示在下一行）
POWERLEVEL9K_PROMPT_ON_NEWLINE=true
```

###  步骤三：修改终端字体： 修改终端所使用的字体为 你安装的某 nerd font 字体。
装完后，依次Preferences > Profiles > Text > Change Font，将字体改成`SauceCodePro Nerd Font`或你自己下载的字体：

###  步骤四：source ~/.zshrc 使配置生效

### 步骤五：修改配色方案
```shell
git clone https://github.com/mbadolato/iTerm2-Color-Schemes.git
之后import 'Tomorrow Night Eighties'
monokai remastered (个人比较喜欢)
```
配置好后，如下图：

![/images/docImages/zsh.png](/images/docImages/zsh.png)


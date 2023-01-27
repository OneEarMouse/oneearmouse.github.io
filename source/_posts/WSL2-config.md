---
title: WSL2相关配置（更新）
date: 2023-01-27 16:43:46
categories:
tags:
description: 介绍WSL2在Windows10和Windows11上的配置。以及配置Windows Terminal，国内源替换、ZSH等
---

本文是对WSL2相关配置的更新与补充

# WSL官方配置教程
参考[微软官方配置教程](https://learn.microsoft.com/zh-cn/windows/wsl/install)
以下以wsl开头的命令建议都使用Windows PowerShell运行

## Windows版本
建议Windows版本都更新到22H2以上

## 启用WSL子系统功能
- 打开"启用或关闭Windos功能"
- 勾选"适用于Linux的Windows子系统"
- 重启电脑完成安装

## Windows10更新内核
参考[旧版 WSL 的手动安装步骤](https://learn.microsoft.com/zh-cn/windows/wsl/install-manual#step-4---download-the-linux-kernel-update-package)。windows10的wsl子系统可能需要更新内核才能使用WSL2，Windows11 22h2以后的版本则没有这个问题

因此，windows 10 电脑需要下载并安装[适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

## WSL2设为默认版本
```bash
wsl --set-default-version 2
```

## 安装Linux分发
查看可用版本
```bash
wsl -l -o
```

下载安装 ```Ubuntu-22.04```
```bash
wsl --install -d Ubuntu-22.04
```

## 检查版本
```bash
wsl -l -v
```
确保安装的wsl的version为2，则成功安装了基于wsl2的Ubuntu-22.04

# 配置
## Windows Terminal相关配置
目前最新版的Windows Terminal已经支持自动配置wsl到Windows Terminal中

如为老版本的Windows Terminal，则需要自行配置setting.json，具体可参考老版本说明

## Ubuntu替换国内镜像源
需要修改WSL子系统的```/ect/apt/sources.list```，可以在WSL的命令行中使用VIM进行编辑，也可以在Windows资源管理器中直接打开该文件进行修改

1. 修改前进行备份

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

2. 修改


```bash
sudo vim /etc/apt/sources.list
```

替换为清华镜像源，参考[Ubuntu 镜像使用帮助](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)
```txt
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

3. 更新

```bash
sudo apt-get update
sudo apt-get upgrade
```

## 配置oh-my-zsh
### 安装ZSH
```bash
$ sudo apt-get install zsh
$ zsh --version
zsh 5.8 (x86_64-ubuntu-linux-gnu)
```

### 安装oh-my-zsh
```bash
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

### 配置主题
需要编辑.zshrc文件，然后使用source刷新
```bash
vim ~/.zshrc
source ~/.zshrc
```
例如我喜欢dst或者ys主题，修改```ZSH_THEME="dst"```或者```ZSH_THEME="dst"```即可

### 配置插件
我喜欢使用的插件有：
- zsh-autosuggestions
- zsh-syntax-highlighting

下载：
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

在zshrc中配置，将其中的plugins行改为
```txt
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

更新配置
```bash
vim ~/.zshrc
source ~/.zshrc
```

## 宿主机上控制WSL中的Docker
目前的Docker DeskTop直接提供了该功能，直接安装后会自动接管WSL中的docker。因此在配置完毕WSL2后直接下载windows版本即可。参考(https://www.docker.com/products/docker-desktop/)
---
title: WSL2相关配置
date: 2021-03-04 22:25:20
categories:
tags:
description: 介绍WSL2在Windows上的配置。以及配置Windows Terminal，国内源替换、ZSH等
---

# WSL配置
参考[微软官方配置教程](https://docs.microsoft.com/zh-cn/windows/wsl/install-win10)
以下的bash命令都需要用管理员身份打开PowerShell运行
## Windows版本
确保windows版本大于1903.18362

## 启用WSL子系统功能
```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

## 启用虚拟机功能
```bash
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## 下载Linux内核包
[适用于 x64 计算机的 WSL2 Linux 内核更新包](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

## WSL2设为默认版本
```bash
wsl --set-default-version 2
```
## 安装Linux分发
打开Microsoft Store并选择Linux分发版，根据默认版本会直接下载运行

## 查看版本
```bash
wsl -s -v
```

## 启动
启动后自动打开cmd窗口，设置用户名和密码


# 配置
## Windows Terminal
在PowerShell中输入如下命令，获取一个guid
```bash
new-guid
```
直接在Windows Terminal的setting.profile.list中加入如下配置，就可以把wsl加入配置中
```json
{
    "guid": "{新获取的guid}",
    "hidden": false,
    "name": "Ubuntu",
    "source": "Windows.Terminal.Wsl"
    }
```

## 默认启动路径
在Windows Terminal的Ubuntu配置中加一句
```json
    "startingDirectory": "//wsl$/Ubuntu/home/<yourname>/"
```

## Ubuntu替换清华源
1. 首先需要使用root用户
```bash
sudo -i 
```

2. 进行原来源的备份
```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak 
```

3. 清空source.list
```bash
echo '' > /etc/apt/sources.list
```

4. 进行源列表的修改
复制并添加源
```bash
vim /etc/apt/sources.list 
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

5. 更新列表
```bash
apt-get update
```

## oh-my-zsh配置
1. 安装ZSH
```bash
$ sudo apt-get install zsh
$ zsh --version
zsh 5.8 (x86_64-ubuntu-linux-gnu)
```

2. 安装oh-my-zsh
```bash
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

3. 配置主题
需要编辑.zshrc文件，然后使用source刷新
```bash
vim ~/.zshrc
source ~/.zshrc
```
我喜欢dst主题，修改```ZSH_THEME="dst"```即可

## git status错误
git在wsl下读取windows中的repo会出现所有文件都modified的错误，是因为linux与windows的文件结尾不同导致的。使用如下命令解决
```bash
git config --global core.autocrlf true
git config --global core.filemode false
```
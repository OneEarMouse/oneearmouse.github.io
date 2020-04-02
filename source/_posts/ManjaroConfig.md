---
title: Manjaro 配置方法个人记录
date: 2019-11-09 19:21:24
categories: ArchLinux
tags:
description: 配置KDE版桌面的Manjaro操作系统的相关记录
---

Manjaro是一个目前广泛是使用的Arch Linux发行版，在各种方面都十分优秀。其最大优势在在于丰富的官方Wiki给出各种组件的介绍与问题解决办法，各种问题都可以轻松解决。其他优势网上有更相近描述。这里不再赘述。Manjaro的官方提供使用不同启动器的多种版本，有XFCE，KDE，GNOME等。还有使用国人操作系统桌面的Deepin桌面。其中，XFCE较轻便，但功能不够丰富。GNOME桌面十分华丽，但响应速度上没有优势。而Deepin桌面是类macos桌面，界面十分优秀，也比较符合国人操作习惯。但在我个人的电脑上出现了许多问题，如触摸板灵敏度无法调节到比较顺手的程度，Wifi断连等问题。因此个人最后选择了KDE桌面，在响应速度和操作上都有不错的表现。

# 系统安装
这一步类似其他操作系统，直接从官方下载操作系统ISO文件并写入U盘进行安装即可。Manjaro的设置界面非常直观，基本不需要过多介绍。这里我跳过不是重点的这部分。记得把\boot\efi挂到efi分区就行。同时我这里选择安装english-us系统，后期再在系统中添加中文字体。

# Pacman的使用
pacman软件包管理器是 Arch Linux 的一大亮点。它将一个简单的二进制包格式和易用的构建系统结合了起来(参见makepkg和ABS)。不管软件包是来自官方的 Arch 库还是用户自己创建，pacman 都能方便地管理。pacman 通过和主服务器同步软件包列表来进行系统更新。这种服务器/客户端模式可以使用一条命令就下载或安装软件包，同时安装必需的依赖包。更具体的操作可以查阅Arch Linux官方Wiki。
## 用法
### 安装软件包
安装或者升级单个软件包，或者一列软件包（包含依赖包），使用如下命令：
```
pacman -S package_name1 package_name2 ...
```
安装本地安装包
```
pacman -U file_name
```
### 删除软件包
删除单个软件包，保留其全部已经安装的依赖关系
```
pacman -R package_name
```
删除指定软件包，及其所有没有被其他已安装软件包使用的依赖关系：
```
pacman -Rs package_name
```
### 升级系统
一个 pacman 命令就可以升级整个系统。花费的时间取决于系统有多老。这个命令会同步非本地(local)软件仓库并升级系统的软件包：
```
pacman -Syu
```
升级内核
```
pacman -Syyuu
```
### 查询软件包
要查询已安装的软件包：
```
pacman -Qs string1 string2 ...
```
### 删除多余
要罗列所有明确安装而且不被其它包依赖的软件包：
```
pacman -Qet
```
### 清理缓存
pacman 将下载的软件包保存在 /var/cache/pacman/pkg/ 并且不会自动移除旧的和未安装版本的软件包，因此需要手动清理，以免该文件夹过于庞大。

使用内建选项即可清除未安装软件包的缓存：
```
pacman -Sc
```
pacman-contrib 提供的 paccache 命令默认会删除近3个版本前的软件包
```
paccache -r
```

# 系统升级
使用该命令升级系统。
```
pacman -Syu
```
或
```
pacman -Syyuu
```

# 添加archlinuxcn源
打开/etc/pacman.conf，在底部添加：
```
[archlinuxcn]
SigLevel = Optional
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
Server可以自己测速选择别的服务器。这里SigLevel为默认的Optional，需要签名认证，因此使用如下命令更新签名。最后更新源
```
sudo pacman -S archlinuxcn-keyring
sudo pacman -Syy
```

# 软件配置
## AUR相关配置
Manjaro的软件源包含的软件非常丰富，安装也非常简单，官方源里无法找到软件，也可以在AUR里搜索。这里我们安装yay来获取aur软件包。注：yaurt已经过时，不建议安装。
```
pacman -S yay
```
用法与pacman类似。注：使用yay时，不需要sudo权限
```
yay -Ss <关键字>	#搜索含有关键字的软件
yay -S <软件名>	#安装软件
yaourt <关键字> #搜索软件包
```
## Vim及文本编辑
```
sudo pacman -S vim
sudo pacman -S gedit
```

## 中文输入法
我们使用fcitx框架来配置输入法。这里我们选用google拼音，也可以自行选择其他。
```
sudo pacman -S fcitx-im
sudo pacman -S fcitx-configtool
sudo pacman -S fcitx-googlepinyin
```
之后按照官方文档，在‘~/.pam_environment’中，加入以下内容，并重启使环境变量生效。
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```
之后系统会自动启动fcitx，在输入法中选择添加输入法，选择显示所有输入法，将google拼音加切换输入法（注：第一个输入法需设置为键盘格式）

## Chrome
安装Chrome或chromium
```
sudo pacman -S google-chrome
sudo pacman -S chromium
```

## QQ
使用如下命令可直接安装deepin移植的tim
```
sudo pacman -S deepin.com.qq.office
```
注意，KDE版直接安装无法启动。需要先安装gnome-settings-daemon，然后在settings->start up and shutdown中添加/usr/lib/gsd-xsettings设置为自动启动。重启系统即可使用。


## 微信
微信的源没有直接加入pacman库，在<https://aur.archlinux.org/packages/deepin-wine-wechat/> 下载source code。使用
```
sudo pacman -U #下载的包名
```
直接安装后无法使用。需要在/opt/deepinwine/apps/Deepin-WeChat/run.sh 中，将WINE_CMD属性改为：WINE_CMD="deepin-wine"，用来调用deepin-wine桌面。此时能够正常启动，但大约1分钟后会闪退。参照<https://github.com/countstarlight/deepin-wine-wechat-arch/issues/32>。因为新版微信的小程序员会导致闪退，使用空文件替换~/.deepinwine/Deepin-WeChat/drive_c/Program Files/Tecent/WeChat/WeChatApp.exe，之后即可正常使用微信

## PDF
```
yay -S foxitreader
yay -S evince
```

### ZSH
```
sudo pacman -S zsh
sudo pacman -S oh-my-zsh-git
cp /usr/share/oh-my-zsh/zshrc ~/.zshrc #ohmyzsh配置文件
chsh -s /bin/zsh #替换默认shell
```
### 网易云音乐
直接可用
```
sudo pacman -S netease-cloud-music
```

至此，manjaro使用环境基本完成。进一步配置请期待后文
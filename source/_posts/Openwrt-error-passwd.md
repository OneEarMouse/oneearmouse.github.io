---
title: Openwrt升级后密码错误
date: 2022-05-21 22:06:37
categories: Openwrt
tags:
description: 记一次Openwrt系统升级之后密码错误
---

事情的起因：使用Openwrt系统一年后，突发奇想需要升级系统。于是重新刷了基于Lean的Openwrt固件，并导入了之前备份的配置。然而出现了问题，无法使用密码登录。

# 现象
1. 导入配置后，ip地址等设置都变为了我备份配置中的配置项。功能正常
2. 使用ssh，可以远程登录到后台，并执行各种命令行操作
3. 使用ip地址能够登录到管理页面，但无法使用之前配置的密码登录页面

根据1与2确认系统正常，备份的配置项正常。但3表示肯定出现了什么问题。期间进行了各种重装、复位、格式化等操作，均无法使其正常工作

# 原因
参考资料:[](https://www.right.com.cn/forum/thread-6188175-1-1.html)和[](https://www.bilibili.com/read/cv14768081/)。按照这两位探路先行者的说法，是更新后的rpcd多了几行代码，导致旧密码无法使用。

# 修复
1. 使用ssh登录
2. 找到```/etc/config/rpcd```，使用vim编辑这个文件。```vim /etc/config/rpcd```
3. 最后重启软路由
4. 成功，可以使用密码正常登录web管理界面啦

又水了一篇文章，真开心
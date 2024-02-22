---
title: Linux手动与自动挂载nfs
date: 2024-02-22 22:19:55
categories: Linux
tags: Linux
description: 简单记录Linux系统如何使用命令行挂载nfs存储，以及如何开机自动挂载nfs存储
---

# 介绍
本文简介如何在命令行界面Linux系统中配置NFS服务器与客户端并实现自动挂载

例子中，服务器ip为192.168.2.31，客户端ip为192.168.2.10，子网掩码255.255.255.0

# NFS Server配置（提供NFS服务）
如果安装的系统提供nfs服务的配置图形界面，那么直接用图形界面操作，别搞命令行

针对无图形界面配置：

## 安装
```bash
sudo apt install mfs-kernel-server -y
```
安装时，会自动在系统文件冲创建```/etc/exports/。

## 准备共享目录
准备共享目录，例如我希望共享```/mnt/storage```
```bash
sudo mkdir /mnt/storage
# 注：这里允许任何人读写nfs，如果有特殊需求则自己设置用户
sudo chown -R nobody:nogroup /mnt/storage
sudo chmod 777 /mnt/storage
```

## 配置共享

编辑/etc/exports。写法1和写法2仅供参考，也可以查阅文档查阅其他控制写法
```bash
vim /etc/exports
```
```txt
# 写法1：仅允许192.168.2.10/24 ip地址挂载读取/mnt/storage，允许读写，sync，无子文件夹检查
/mnt/storage  192.168.2.10/24(rw,sync,no_subtree_check)
# 写法2：允许所有ip地址挂载读取/mnt/storage，允许读写
/mnt/storage_main_ssd/data      (rw)
```

## 重启服务
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

# NFS Client配置
## 安装
```bash
sudo apt install nfs-common -y
```

## 检查
检查目标主机是否正常启动
```bash
showmount -e 192.168.2.31
```
如果正常，则输出如下：
```
Export list for 192.168.2.31:
/mnt/storage_main_ssd/data *
```

## 挂载
创建挂载点
```bash
sudo mkdir -p /mnt/storage
```

挂载
```bash
mount 192.168.2.31:/mnt/storage /mnt/storage
```
如成功，则会显示create symbol link xxx

## 验证
客户端使用```df -h```命令，查看挂载是否成功。正常则会显示：
```bash
Filesystem                                   Size  Used Avail Use% Mounted on
192.168.2.31:/mnt/storage                    454G   55G  381G  13% /mnt/storage
```
客户端创建文件
```bash
echo Hello,world >>/mnt/storage/hello.txt
```
检查服务器中是否成功创建该文件
```bash
cat /mnt/storage/hello.txt
```

## 自动挂载配置
编辑```/etc/fstab```
```bash
# 来源                              本地挂载目录    挂载协议    选项    第一个数字表示是否备份，第二个表示是否fsck磁盘检测
192.168.2.31:/mnt/storage         /mnt/storage   nfs     defaults        0 0
```

设置完成后，reboot，检查挂载点是否存在，验证自动挂载是否成功


# 注：
如果希望在PVE的LXC容器中挂载nfs或smb，需要将LXC容器设为特权容器，否则无法正确加载nfs相关程序
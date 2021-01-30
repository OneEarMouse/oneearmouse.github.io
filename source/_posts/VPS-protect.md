---
title: 个人VPS服务器防护指南
date: 2021-01-30 19:34:14
categories: 服务器
tags:
description: 简单介绍和记录常用的个人服务器防护方法。包括改变SSH端口，防火墙拦截，DenyHosts自动拒绝等。方便自己和网友查阅
---

# 服务器（VPS）防护简易指南
初次自己搭服务器的同学，不久以后一般会发现自己的```/var/log/```文件夹下充满了各种各样的访问错误日志。这是因为网上充斥了各种各样的SSH攻击。直接暴力破解你的SSH密码。我曾经在一个月内有过被同一个IP连接4000多次的记录。所以出来搭服务器的同学们要学会保护自己的服务器啊。接下来就以centos为例记录一下我是如何保护我的服务器的。用其他发行版的同学可以参考。
主要思路就是改变SSH的登陆端口，开启防火墙，以及利用DenyHosts自动分析拦截的组合，达成防护效果。


参考来源：[链接][https://segmentfault.com/a/1190000021752790]

# 查看攻击记录
大部分linux发行版都会把ssh连接的log放在```/var/log/```文件夹下。centos是secure文件，ubuntu是auth.log文件。通过查阅这些文件以及简单的使用grep等命令，我们可以知道谁在攻击（虽然这并没有什么卵用，除非你想一个个把他们拉黑名单）。

## 我们可以通过如下命令查看被攻击的次数
```
grep "Failed password" /var/log/secure | wc -l
```

## 查看所有认证成功
```
grep "password" /var/log/secure | grep -v Failed | grep -v Invalid
```
只有自己的ip，说明最近没有别人登录成功过。

## 统计攻击者ip
利用awk统计SSH连接记录与次数
```
awk '{if($6=="Failed"&&$7=="password"){if($9=="invalid"){ips[$13]++;users[$11]++}else{users[$9]++;
ips[$11]++}}}END{for(ip in ips){print ip, ips[ip]}}' secure* | sort -k2 -rn | head
```
显示
```
67.5.7.148 133
170.106.153.18 66
64.227.115.46 56
174.138.5.184 56
54.38.65.215 54
159.65.41.159 51
152.136.133.109 46
210.212.172.182 27
186.146.76.6 27
124.207.221.66 26
```

## 查看攻击者尝试用户名
```
awk '{if($6=="Failed"&&$7=="password"){if($9=="invalid"){ips[$13]++;users[$11]++}else{users[$9]++;
ips[$11]++}}}END{for(user in users){print user, users[user]}}' secure* | sort -k2 -rn | headawk '{if($6=="Failed"&&$7=="
password"){if($9=="invalid"){ips[$13]++;users[$11]++}else{users[$9]++;ips[$11]++}}}END{for(user in users){print user, u
sers[user]}}' secure* | sort -k2 -rn | head
```
可以看到尝试的一般是root或者admin的组合

# 配置SSH登陆
SSH的配置文件在```/etc/ssh/ssh_config```。首先，记得把本地生成的SSHkey放到```/当前用户~目录/.ssh/```的authorized_keys中，使自己的SSH公钥存放在服务器端。然后，对该配置文件进行修改。
- 找到 Port,修改为其他你想使用的接口
- 找到 PasswordAuthentication, 设置为no。使SSH不能使用密码登陆
- 找到 PubkeyAuthentication，确保为yes。使SSH能直接通过key验证登陆

然后重启SSH服务
```
service sshd restart
```
在本地通过```ssh -p 新端口 ipaddress```便可以直接连接到新端口了。

注：如果发现自己配置不对，怎么都连不上，无法修改配置，不要慌张。可以到VPS供应商提供的web版console接口中，一般能绕过防火墙和SSH配置直接登录。

# 配置服务器防火墙
根据我们服务器的实际用途，通常我们只需要使几个部分端口便能够完成所有服务。剩余的开放端口极其容易被后台程序用于后面连接。因此推荐使用防火墙禁用其他端口。不同linux发行版的防火墙用法不同，我这里以我目前用的centos7的firewall为例。你应该根据自己情况选用适当的防火墙，如果VPS自带防火墙，使用VPS提供的自带防火墙也是不错的选择。

## firewall简单使用教程
注意：centos7中的防火墙为firewall，是centos6中iptable的包装版。如果发现自己机器没有附带，使用```yum install firewalld```安装。

firewall功能十分强大，平常的使用分为临时与永久。另外还可以将配置区域设定为public,home,work等。这里不涉及这些设置，毕竟我们只需要默认的public一个配置。就简单写一下端口控制相关命令

注：直接开启防火墙很可能直接把你的SSH挤掉线，请尽量改好list再开启或使用vps提供的web console操作。
### 查看状态
```cmd
systemctl status firewalld
```

### 查看版本
```
firewall-cmd --version
```

### 启动和停止
1. 启动
```
systemctl start firewalld
```

2. 停止
```
systemctl stop firewalld
```

3. 重启
```
systemctl restart firewalld
```

### 查看防火墙规则
```
firewall-cmd --list-all
```

### 端口控制
#### 查看所有端口
```
firewall-cmd --list-ports
```

#### 添加端口
如果不加--permanent则只临时添加，重启后失效
```
firewall-cmd --add-port=2888/tcp --permanent  
```

#### 删除端口
```
firewall-cmd --remove-port=2888/tcp --permanent
```

## 端口选择
我的服务器只需要web服务，以及SSH控制。所以我只留了80，443和刚才设置的SSH端口。杜绝了其他一切连接。

# DenyHosts
如果你非常不幸，换了端口，加了防火墙，还是有无聊的家伙在用SSH暴力破解你的新端口。那你可以考虑安装DenyHosts这个软件。DenyHosts是基于Python的一个程序，它会分析SSHD的日志文件（/var/log/secure等），发现同一IP在进行多次SSH密码尝试失败时就会记录该IP到/etc/hosts.deny文件，从而达到自动屏蔽该IP的目的。

## DenyHosts安装
### 下载DenyHosts安装包。
```
wget http://imcat.in/down/DenyHosts-2.10.tar.gz
```
 

### 解压DenyHosts安装包
```
[root@mylnx04 ~]# tar -zxvf DenyHosts-2.10.tar.gz
```
 

### 开始DenyHosts的安装
安装DenyHosts前必须安装Python，当然现在绝大部分Linux主机应该都默认安装了Python。
```
cd DenyHosts-2.10/
python setup.py install
```
 
## DenyHosts配置

复制配置文件denyhosts.cfg
```
cp /usr/share/denyhosts/denyhosts.cfg-dist /usr/share/denyhosts/denyhosts.cfg
```
 
设置/usr/share/denyhosts/denyhosts.cfg相关参数。

```
PURGE_DENY 
 
多久清除屏蔽的IP的记录。
 
########################################################################
#
# PURGE_DENY: removed HOSTS_DENY entries that are older than this time
#             when DenyHosts is invoked with the --purge flag
#
#      format is: i[dhwmy]
#      Where 'i' is an integer (eg. 7) 
#            'm' = minutes    #分钟
#            'h' = hours      #小时
#            'd' = days       #天
#            'w' = weeks      #周
#            'y' = years      #年
#
# never purge:
PURGE_DENY =              #表示所有条目永远不删除（这里才是实际的设置）
#
# purge entries older than 1 week
#PURGE_DENY = 1w        #表示删除记录超过一周的条目
#
# purge entries older than 5 days
#PURGE_DENY = 5d        #表示删除记录超过5天的条目
#######################################################################
 
PURGE_THRESHOLD
 
定义某个host最多被清除几次。 超过PURGE_THRESHOLD值就不会被清理了。
#######################################################################
#
# PURGE_THRESHOLD: defines the maximum times a host will be purged.  
# Once this value has been exceeded then this host will not be purged. 
# Setting this parameter to 0 (the default) disables this feature.
#
# default: a denied host can be purged/re-added indefinitely
#PURGE_THRESHOLD = 0
#
# a denied host will be purged at most 2 times. 
#PURGE_THRESHOLD = 2 
#
#######################################################################
 
BLOCK_SERVICE   表示阻止的服务名。
 
    默认为sshd，也可以设置FTP、SMPT等。
 
#######################################################################
#
# BLOCK_SERVICE: the service name that should be blocked in HOSTS_DENY
# 
# man 5 hosts_access for details
#
# eg.   sshd: 127.0.0.1  # will block sshd logins from 127.0.0.1
#
# To block all services for the offending host:
#BLOCK_SERVICE = ALL
# To block only sshd:
BLOCK_SERVICE  = sshd   #禁止的服务名，当然DenyHost不仅仅用于SSH服务，还可用于SMTP等等。
# To only record the offending host and nothing else (if using
# an auxilary file to list the hosts).  Refer to: 
# http://denyhosts.sourceforge.net/faq.html#aux
#BLOCK_SERVICE =    
#
#######################################################################
 
DENY_THRESHOLD_INVALID 
 
允许无效用户登录失败的次数
 
#######################################################################
#
# DENY_THRESHOLD_INVALID: block each host after the number of failed login 
# attempts has exceeded this value.  This value applies to invalid
# user login attempts (eg. non-existent user accounts)
#
DENY_THRESHOLD_INVALID = 1  #允许无效用户登录失败的次数
#
#######################################################################
 
 
DENY_THRESHOLD_VALID
 
    允许有效（普通用户）用户登陆失败的次数
#######################################################################
#
# DENY_THRESHOLD_VALID: block each host after the number of failed 
# login attempts has exceeded this value.  This value applies to valid
# user login attempts (eg. user accounts that exist in /etc/passwd) except
# for the "root" user
#
DENY_THRESHOLD_VALID = 5 #允许普通用户登陆失败的次数
#
#######################################################################
 
 
DENY_THRESHOLD_ROOT
 
允许root登录失败的次数。
 
#######################################################################
#
# DENY_THRESHOLD_ROOT: block each host after the number of failed 
# login attempts has exceeded this value.  This value applies to 
# "root" user login attempts only.
#
DENY_THRESHOLD_ROOT = 1  #允许root登陆失败的次数
#
#######################################################################
 
 
DENY_THRESHOLD_RESTRICTED 设定DenyHost 写入到该资料夹
#######################################################################
#
# DENY_THRESHOLD_RESTRICTED: block each host after the number of failed 
# login attempts has exceeded this value.  This value applies to 
# usernames that appear in the WORK_DIR/restricted-usernames file only.
#
DENY_THRESHOLD_RESTRICTED = 1
#
#######################################################################

 
DAEMON_PURGE
 
表示DenyHosts在守护进程模式下运行的频率，运行清除机制清除HOSTS_DENY中的过期的记录
如果PURGE_DENY为空，这没有任何效果。
#######################################################################
#
# DAEMON_PURGE: How often should DenyHosts, when run in daemon mode,
# run the purge mechanism to expire old entries in HOSTS_DENY
# This has no effect if PURGE_DENY is blank.
#
DAEMON_PURGE = 1h
#
#######################################################################
 

``` 

### 设置DenyHost启动
启动或重启DenyHosts服务
``` 
#service denyhosts restart
```

# 总结
对于一般的个人服务器用户来说，改变SSH端口加防火墙的策略就能隔绝大部分攻击。攻击者一般不愿意从65536个端口中猜测你的新端口，基本上杜绝了收到攻击。DenyHost一般是无法改变端口后的补救措施，还会损失部分连接性能，一般只建议懒人使用。对于公司的大型正式服务器，一般需要设置仅限堡垒机和跳板机或者仅限内网访问，杜绝一切连接可能。
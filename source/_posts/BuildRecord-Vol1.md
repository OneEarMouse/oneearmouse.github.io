---
title: 在github使用hexo搭建博客记录Vol.1
date: 2019-06-17 16:43:18
categories: hexo
tags: 
description: 使用hexo搭建本网站的具体步骤记录第一部分,主要介绍hexo的配置与github部署
---

参考资料：
- Hexo搭建独立博客全纪录（三）使用Hexo搭建博客: [链接](https://baoyuzhang.github.io/2017/05/12/%E3%80%90Hexo%E6%90%AD%E5%BB%BA%E7%8B%AC%E7%AB%8B%E5%8D%9A%E5%AE%A2%E5%85%A8%E7%BA%AA%E5%BD%95%E3%80%91%EF%BC%88%E4%B8%89%EF%BC%89%E4%BD%BF%E7%94%A8Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)
- Hexo官方中文文档：<https://hexo.io/zh-cn/docs/>
- Next官方文档：<http://theme-next.iissnan.com/>

# 环境配置
要求：
- 一个github账号
- node.js
- npm
- git

本文环境：
- Windows 10 17763
- node.js-10.16.0 (node -version)
- git - 2.20.1.windows.1 (git version)
- hexo - 3.8.0 (hexo version)

# github配置
## 创建仓库
在github中新建名为“用户名.github.io”的repo。该repo作为网站代码存储，同时，http://用户名.github.io 为未来网站访问地址。github也允许绑定自定义地址，可以在repo的setting中设置。
注意点：
  - 该repo在非付费账户中必须设置为public，付费账户可以设置为private；
  - 仓库名字必须是：username.github.io，其中username是你的用户名；

## 配置SSH Key
使用Hexo配置服务器时，需要github的访问权限。同时，本地git配置的访问方式无法被Hexo直接调用，因此需要配置SSH Key使Hexo能直接调用并获得访问权限。

1. 首先在git设置身份和邮箱，关联到你的github账号。如果已经配置过请跳过
    ```
    git config --global user.name "yourname"
    git config --global user.email“your@email.com"
    ```
2. Windows环境下：/Users/your_user/.ssh删除known_hosts
3. git bash 在输入命令
   ```
   ssh-keygen -t rsa -C "your@email.com"
   ```
   git bash提示：
   ```
   Generating public/private rsa key pair.
   Enter file in which to save the key (/Users/your_user_directory.ssh/id_rsa):
   ```
   根据提示保存到该目录，获得id_rsa和id_rsa.pub。id_rsa是私钥，id_rsa.pub是公钥，请注意保管用文本方式打开id_rsa.pub，获取其中SSH Key
4. 打开https://github.com/，登陆你的账户，进入ssh设置，点击new SSH Key，将获得的Key粘贴
5. 完成

# Hexo博客框架
## Hexo简介
Hexo是一个简单、快速、强大的基于 Github Pages 的博客框架，支持Markdown格式，有众多优秀插件和主题。
github: https://github.com/hexojs/hexo
Hexo中文官网： https://hexo.io/zh-cn/

## Hexo原理
github page只能存放静态文件，而博客需要多种动态内容。如果手动更新，非常不便。Hexo将这些文件全部放在本地。每次需要更新时，先使用MarkDown语言编辑要发布的文章。写完后调用Hexo命令来批量生成页面，最后将所有改动提交到github，是一种适合github page的框架。

## Hexo安装
注意本文所有cmd命令均使用git bash输入。
安装命令：
```
npm install -g hexo
```

## Hexo初始化
在本地硬盘中找一个地方新建文件夹，例如我的D:\hexo，用于存放hexo生成的网页代码。在该目录中使用命令：
```
hexo init
```
hexo会自动在该目录中创建文件，结构如下：
```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```
## 生成静态文件
```
hexo generate (完整写法)
hexo g        (缩写)
```
根据目前目录中的文件，在public文件夹中生成html文件，用于提交到github

## 本地测试
```
hexo start  (完整写法)
hexo s      (缩写)
```
hexo会在本地调用4000端口开启本地服务器。如果端口被占用请自行调整端口分配。
启动后，就能直接在浏览器中看到Hexo的默认界面了，还有一篇hello world的初始文章

## 部署到github
### 配置文件
打开hexo根目录下配置文件_config.yml，在最后配置有关deploy的部分.注意冒号与属性直接的空格
```
deploy:
  type: git
  repository: git@github.com:yourusername/yourusername.github.io.git
  branch: master
```
### 安装npm的git插件
```
npm install hexo-deployer-git
```
### 部署到github
在git bash中使用命令来部署。注意：必须使用git bash，否则无法访问本地SSH Key文件
```
hexo deploy (完整命令)
hexo d      (缩写)
```
经过一小段时间的等待，部署完成了！
查看github上的repo，如果已经出现了内容，那么说明部署成功
打开 <http://yourusername.github.io/> ，就能看到你的网站了！
如何调整网站配置及主体，请期待Vol.2
---
title: Hexo-VPS
date: 2020-09-12 23:19:25
categories: hexo
tags:
description: 介绍如何将hexo部署到VPS上
---

最近心血来潮，决定将原本托管在github的hexo静态页面搬运到vps上，这里记录一下如何实现。

参考资料：
- https://www.xiaoweihuang.me/2018/11/05/deploy-hexo-to-vps/
- https://www.cnblogs.com/morethink/p/10867173.html

# 本地SSH配置
Hexo部署是通过git来完成的，而git又是基于ssh连接的，所以需要在本地和服务端进行ssh相关配置。

如果之前没有使用git创建过ssh。需要打开git bash, 执行以下命令：
```
$ git config --global user.email "你的邮箱"
$ git config --global user.name "你的用户名"
```
这里的邮箱和用户名都是服务器用于记录git提交记录的，然后是本地端生成ssh Key：
```
$ ssh-keygen -t rsa -C "你的邮箱" // 执行这个命令会提示输入用于保存的密钥名
和口令之类的，都不填
```
执行完后会在C:\Users\Administrator\.ssh目录下生成id_rsa和id_rsa.pub两个文件，其中id_rsa.pub文件里的内容是等会要复制到服务器那里的
可以在git bash执行以下命令获取到id_rsa.pub文件里的内容。也可以直接用记事本之类的文本编辑器打开复制
```
$ cat .ssh/id_rsa.pub // 执行后可以看到公钥内容
```

# VPS服务器配置
首先我们配置服务器上的SSH key。如果没有ssh，则自行安装。这里不再赘述
```
# 添加hexo用户（注：这里用户名应该可以随意修改，但有人说git用户更好，待调查）
adduser hexo
# 切换到hexo用户
su hexo
# 切换到hexo用户目录
cd /home/hexo
# 创建.ssh文件夹
mkdir .ssh
# 创建authorized_keys文件并编辑
echo "id_rsa.pub中的内容" > .ssh/authorized_keys
# 如果你还没有生成公钥，那么首先在本地电脑中执行 cat ~/.ssh/id_rsa.pub | pbcopy生成公钥
# 修改相应权限
chmod 600 .ssh/authorized_keys
chmod 700 .ssh
```

# 配置post-update钩子
Git的钩子脚本位于版本库.git/hooks目录下，当Git执行特定操作时会调用特定的钩子脚本。当版本库通过git init或者git clone创建时，会在.git/hooks目录下创建示例脚本，用户可以参照示例脚本的写法开发适合的钩子脚本。

钩子脚本要设置为可运行，并使用特定的名称。Git提供的示例脚本都带有.sample扩展名，是为了防止被意外运行。如果需要启用相应的钩子脚本，需要对其重命名（去掉.sample扩展名)。

```
post-update
该钩子脚本由远程版本库的git receive-pack命令调用。当从本地版本库完成一个推送之后，即当所有引用都更新完毕后，在远程服务器上该钩子脚本被触发执行。
```

因此我们需要配置post-update钩子以便可以及时更新我们在VPS上存放Hexo 静态文件的目录。

```
# 回到hexo目录
cd /home/hexo
# 变成hexo用户
su hexo
# 新建blog目录存放hexo静态文件
mkdir /home/hexo/blog
# 使用hexo用户创建git裸仓库，以blog.git为例
git init --bare blog.git
# 进入钩子文件夹hooks
cd blog.git/hooks/
# 启用post-update
mv post-update.sample post-update
# 添加执行权限
chmod +x post-update
# 配置post-update
vim post-update
```

注释如下行：
```
exec git update-server-info
```
添加如下代码：
```
git --work-tree="静态文件VPS存放目录" --git-dir="刚才新建的VPS git地址" checkout -f
```
例如：
```
git --work-tree=/home/hexo/blog --git-dir=/home/hexo/blog.git checkout -f
```

当然，这里直接放在/home目录不太妥当，也可以放在其他文件夹例如```/usr/local/static```，更符合对于不同用户的权限分离。但是注意，放在非hexo用户的目录下，需要用```chmod +777```给目录配置权限，否则git hock无法访问

# Nginx
nginx作为一个完成度非常高的负载均衡框架，和很多成熟的开源框架一样，大多数功能都可以通过修改配置文件来完成，使用者只需要简单修改一下nginx配置文件，便可以非常轻松的实现比如反向代理，负载均衡这些常用的功能。这里我们仅仅用到最简单的功能，但还是介绍一下比较好
## Nginx基本操作
启动nginx：
```
##使用systemctl
systemctl start nginx.service
#使用service
service nginx start
#或，简单粗暴的直接调用
nginx
```
停止nginx:
```
##使用systemctl
systemctl stop nginx.service
#使用service
service nginx stop
#直接强行结束
nginx -s stop
##Nginx在退出前完成已经接受的连接请求。
nginx -s quit
```
重启nginx：
当我们修改了nginx的某些配置，为了使配置生效，我们往往需要重启nginx，同样的，linux下依然有两种方式来重启我们的nginx服务:
```
##使用systemctl
systemctl restart nginx.service
#使用service
service nginx restart
#使用nginx命令重启
nginx -s reload
```
其他命令：检查配置文件，同时显示配置文件目录：
```
nginx -t
```

要注意，如果不是在本地服务器直接调试。在服务器端的各种修改很可能因为DNS解析等各种原因不能及时更新。重新nginx之后切忌多ctrl+f5刷新几次，血的教训。

## Nginx简单配置
`/etc/nginx/nginx.conf`是新的默认配置文件，但server块在最新版本中不在这里，而是放到了`/etc/nginx/sites-available`中，并在`/etc/nginx/sites-enabled`中用软连接控制配置。我们这里比较简单，直接修改`/etc/nginx/sites-available`中的default文件中的server块就行
```
server {
        # 默认监听80端口
        listen       80 default_server;
        listen       [::]:80 default_server;
        # 修改server_name为自己之前注册好的域名，没有就不用更改
        server_name  xxx.com;
        # 修改网站根目录，在这里存放你的Hexo静态文件，请自行选择或创建目录
        root         /home/hexo/blog;
        # 其他保持不变

        location / {
        }
    }
```

# 更新Hexo的deploy方式
找到本地Hexo博客的站点配置文件_config.yml，找到以下内容并修改：
```
deploy:
  type: git
  repo: git@github.com:XXXX/XXXX.github.io.git
  branch: master
```
可以看到这里只能上传到github一种。但我们想同时传到vps和github托管，可以吗？直接这样写就行：
```
deploy:
  type: git
  repo:
    github: git@github.com:XXXX/XXXX.github.io.git
    vps: hexo@xxx.com:/your/file/path
  branch: master
```
如果你修改了SSH端口，可以使用该方法来访问
```
vps: ssh://hexo@xxx.com:port/your/file/path
```

之后直接使用：
```
hexo clean
hexo generate
hexo deploy
```
就能同时上传到VPS和github托管了！
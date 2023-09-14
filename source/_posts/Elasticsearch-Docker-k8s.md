---
title: Docker与K8s中的ElasticSearch持久化
date: 2022-06-13 00:01:47
categories: ElasticSearch
tags:
description: 简单记录如何在Docker与k8s中持久化部署ElasticSearch的容器
---

容器中启动的ElasticSearch实例，如果不配置持久化，就会在重启时丢失数据。借助最近将一个实例迁移到k8s中的机会简单记录一下如何在容器中持久化ElasticSearch实例。同时记录一下ik分词器如何安装到容器中

# 容器中的es
## Docker拉取ES
1. 搜索镜像：打开 DockerHub 官网 https://hub.docker.com/然后搜索 Elasticsearch 镜像
2. 命令```docker pull elasticsearch: x.x.x```其中x.x.x为需要的版本

## 容器中的es
尝试启动一个es的docker容器，并使用```docker --it```命令
可以看到容器的默认执行目录为```/usr/share/elasticsearch```。其中有如下文件夹与文件夹
```bash
LICENSE.txt  NOTICE.txt  README.textile  bin  config  data  lib  logs  modules  plugins
```

其中：
- config文件夹为默认配置存放路径
- data为数据持久化路径
- plugins为插件安装路径

因为我们的目标是配置持久化的es，并且安装ik分词。因此这三个目录都需要进行挂载来防止容器重启后数据丢失

# k8s中的ES持久化
## 存储卷与配置
1. 先临时启动一个容器，用于配置
2. 创建一个存储卷，挂载并映射到```/usr/share/elasticsearch/data```，作为数据文件夹持久化用
3. 创建一个存储卷，挂载并映射到```/usr/share/elasticsearch/plugins```，作为插件文件夹持久化用
4. 创建一个存储卷，挂载并映射到```/usr/share/elasticsearch/myconfig```，作为配置文件夹持久化用
5. 进入容器中，执行命令```cp -r ./config、。 /myconfig```将默认的配置项复制到```/myconfig```中
6. 修改```/myconfig```文件夹中的各种配置文件，用于es的配置。参考默认的[ES文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)

## 启动容器
给容器配置环境变量```ES_PATH_CONF="/usr/share/elasticsearch/myconfig"```，容器就会在启动时自动去加载挂载的```/myconfig```文件夹中的配置文件

## 安装Ik-分词插件
1. 进入容器，来到目录```/usr/share/elasticsearch/pulgins```
2. 创建目录```mkdir /usr/share/elasticsearch/plugins/ik```
3. ```cd mkdir /usr/share/elasticsearch/plugins/analysis-ik```
4. ```wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/vx.x.x/elasticsearch-analysis-ik-x.x.x.zip```,将其中的x.x.x替换为需要的版本
5. ```unzip elasticsearch-analysis-ik-x.x.x.zip```
6. 删除容器并重启

注：这里最好不要用```/bin/elasticsearch-plugin install```命令进行安装。否则可能会将ik分词器的配置文件安装到默认的```/config```文件夹中，导致容器重启后配置失效丢失

## 验证
按照上面的挂载方式修改完成后，打开```http://ipaddress/_cluster/health?pretty```查看节点是否都启动成功。打开```http://ipaddress/_analyze```查看分词器是否配置正确


# docker中的es持久化
## 方式1
可以参考上面的方式，将config文件夹中的文件复制到/myconfig中。然后使用如下命令启动。之后参考上面的方式按照ik分词器
```bash
docker run --name elasticsearch -p 9200:9200  -p 9300:9300 \
 -e ES_PATH_CONF="/usr/share/elasticsearch/myconfig" \
 -v /home/temp/elasticsearch/myconfig:/usr/share/elasticsearch/myconfig \
 -v /home/temp/elasticsearch/data:/usr/share/elasticsearch/data \
 -v /home/temp/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
 -d elasticsearch:x.x.x
```

## 方式2
可以直接将本地的配置文件挂载到docker的es中，如下。这样可以减少一个挂载目录。之后参考上面的方式按照ik分词器
```bash
docker run --name elasticsearch -p 9200:9200  -p 9300:9300 \
 -v /home/temp/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
 -v /home/temp/elasticsearch/data:/usr/share/elasticsearch/data \
 -v /home/temp/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
 -d elasticsearch:x.x.x
```

## 验证
方式同上，略
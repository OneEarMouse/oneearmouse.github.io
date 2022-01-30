---
title: Hexo更新
date: 2021-03-07 20:40:42
categories: hexo
tags:
description: 简单记录升级Hexo以及Next主题的折腾记录
---

# Hexo以及Next主题升级
最近需要在新电脑上重用旧的hexo工程，并安装Hexo最新版本，遇到一大堆坑，特此记录

# 安装NodeJS以及Hexo
这里我选择在windows上使用nvm安装。nodejs版本过高或者过低都会导致hexo渲染错误。通过nvm不断选择调试，我选择用nodejs的12.14.0版本。

随后在git pull下的旧目录中执行这些命令进行安装。
```bash
npm install hexo-cli -g
npm install --save
```

# 重新安装Next主题
震惊的发现旧版本已经不维护了，新版本移动到了```https://github.com/theme-next/hexo-theme-next```

我因为多次尝试直接升级然后报错，最后选择了直接把```thems\next```文件夹删除，并重新从github上pull了最新版本。
```
git clone https://github.com/theme-next/hexo-theme-next themes/next
```
然后把旧的```_config.yml```文件用git compare打开，比对选项的设置。（方法虽然愚蠢、但是有效，起码比在那折腾版本文件一小时做了无用功好，是吧是的😎）

# 尝试新功能
## 页面标题自动编号功能
插件来源：参考页面(https://www.npmjs.com/package/hexo-heading-index)
1. 安装
   ```bash
   $ npm install hexo-heading-idnex --save
   ```
2. 在hexo根目录下的`_config.yml`站点配置文件中，添加如下代码来启用自动编号功能。
   ```yml
   heading_index:
   enable: true
   index_styles: "{1} {1} {1} {1} {1} {1}"
   connector: "."
   global_prefix: ""
   global_suffix: ". "
  ```
3. 注：next主题中，会在左侧侧边栏创建标号。如果与这个插件一起使用，则会重复创建。因此我们先将其关闭。打开next的配置文件```_congig.yml```，搜索```toc```将```number```后面的true改成false，这样左侧栏就不会自动计数了

## 增加访问统计
发现Next中已经增加了访问统计，直接在Next的```_config.yml```中搜索```busuanzi_count```，启用即可。但是因为Hexo是静态网站，貌似是有问题的，需要再研究一下

## 站内搜索
hexo-generator-searchdb插件
1. 安装
   ```bash
   npm install hexo-generator-searchdb --save
   ```
2. 在hexo根目录下的`_config.yml`站点配置文件中
   ```yml
    search:
    path: search.xml
    field: post
    format: html
    limit: 10000
   ```
3. 在themes目录下的`_config.yml`主题配置文件中
   ```yml
    local_search:
        enable: true
   ```

# 结尾
好了，今天的折腾又搞完了。又浪费了一个半小时搞了没用的东西呢🤔

人生不过两万天，能摸一天是一天！
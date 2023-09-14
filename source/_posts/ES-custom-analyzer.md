---
title: ElasticSearch自定义分词器
date: 2023-09-14 23:30:39
categories:
tags: ElasticSearch
description: 介绍ElasticSearch中分词器的基本组成，并给出了几个自定义分词器的简单例子及使用方法
---

# 分词器的组成
ES中的分词器的概念包括以下三个部分

## 字符过滤器 char_filter
作用：在tokenizer之前对原始文本进行处理，比如增加，删除，替换字符等

例子：
- 去除HTML标签
- 将&替换为and

## 分词器 tokenizer
作用：按照一定规律，将过滤后的原始文档切分为文本块

例子：
- 根据下划线、空格、单词大小写等分割

## 分词过滤器 token filter
作用：针对tokenizer 输出的单词进行增删改等操作。可以修改、去除、增加等

例子：
- 将单词全部小写化
- 去除无意义单词
- 增加词缀等

# ES中的内置分词器
ES中的默认分词器为：standard tokenizer, 是标准分词器。包含：
1. standard token filter: 去掉无意义的标签, 如<>, &, - 等.
2. lowercase token filter: 将所有字母转换为小写字母.
3. stop token filer(默认被禁用): 移除停用词, 比如"a"、"the"等.

# 制作自定义分词器
## 设置中的分词器格式
在ES中，设置自定义分词器需要对索引操作。具体json路径为settings-analysis。格式如下：
```json
{
  "settings": {
    "analysis": {           // 分词设置
      "char_filter": {},    // char_filter  关键字 对应字符过滤器
      "tokenizer": {},      // tokenizer 关键字 对应分词器
      "filter": {},         // filter  关键字 对应分词过滤器
      "analyzer": {}        // analyzer 关键字 定义自定义分词器
    }
  }
}
```

## 一个例子
一下是一个自定义分词器的例子
```json
PUT test_index/_settings
{
    "analysis": {
        "char_filter": {
            "&_to_and": {   // 自定义&_to_and过滤器
                "type": "mapping",
                "mappings": [
                    "& => and"
                ]
            }
        },
        "tokenizer": {
            "custom_tokenizer": { // 自定义分词器，根据下划线、横线、斜线、点分词
                "type": "pattern",
                "pattern": "[/_\\-\\.]"
            }
        },
        "filter": {
            "my_stopwords": {
                "type": "stop",
                "stopwords": [
                    "the",
                    "a"
                ]
            }
        },
        "analyzer": {
            "my_analyzer": { // 自定义的分析器名称
                "type": "custom",
                "char_filter": [
                    "html_strip",
                    "&_to_and"
                ], // 跳过HTML标签, 将&符号转换为"and"
                "tokenizer": "custom_tokenizer", // 自定义分词
                "filter": [
                    "lowercase",
                    "my_stopwords"
                ] // 转换为小写
            }
        }
    }
}
```

# 使用

分词器有两种使用情景
- 创建或更新文档时，会对文档进行分词
- 查询时，对查询语句中的关键词分词

以下简单介绍了一些用法及操作

## 创建索引时加入分词器
在创建索引时，可直接指定在索引中加入分词器。

注意：
- 加入分词器仅仅是在索引中添加分词器
- 如果某字段需要使用自定义分词器，则需要手动指定，否则使用默认分词器

```json
PUT index_name/_settings
{
    "analysis": {
        "char_filter": {
            "&_to_and": {
                "type": "mapping",
                "mappings": [
                    "& => and"
                ]
            }
        },
        "filter": {
            "my_stopwords": {
                "type": "stop",
                "stopwords": [
                    "the",
                    "a"
                ]
            }
        },
        "analyzer": {
            "my_analyzer": { // 自定义的分析器名称
                "type": "custom",
                "char_filter": [
                    "html_strip",
                    "&_to_and"
                ], // 跳过HTML标签, 将&符号转换为"and"
                "tokenizer": "standard",
                "filter": [
                    "lowercase",
                    "my_stopwords"
                ] // 转换为小写
            }
        }
    },
    "mappings": {
        "doc": {
            "properties": {
                "my_text": {
                    "type": "text",
                    "analyzer": "my_analyzer", // my_text字段使用my_analyzer分词器
                }
            }
        }
    }
}
```

## 修改索引中的分词器
在ES中如果想修改当前索引的分词器。必须先关闭索引, 添加完成后, 再及时打开索引进行搜索等操作, 否则将出现错误。

```json
// 关闭索引:
POST index_name/_close

// 启用English停用词token filter
PUT index_name/_settings
{
    "analysis": {
        "analyzer": {               // 关键字
            "my_analyzer": {        // 自定义的分词器
                "type": "standard", //分词器类型standard
                "stopwords": "_english_" //standard分词器的参数，默认的stopwords是\_none_
            }
        }
    }
}

// 重新打开索引:
POST index_name/_open
```

## 测试ES自带分词器效果
测试自带分词器时，可直接使用_analyze命令，不指定索引
```json
GET _analyze			// ES引擎中已有standard分词器, 所以可以不指定index
{
    "analyzer": "standard", 
    "text": "There-is & a DOG<br/> in house"
}
```

## 测试自定义分词器效果
在测试自定义分词器时，必须指定索引以及索引中使用了自定义分词器的字段，才能使用索引中的自定义分词器
```json
POST test/_analyze
{
  "field": "my_text",    // my_text字段使用my_analyzer分词器
  "text": ["The test message."]
}
```

## 指定查询时使用哪个分词器
- 查询时通过analyzer指定分词器
```json
GET test_index/_search
{
  "query": {
    "match": {
      "name": {
        "query": "lin",
        "analyzer": "standard"
      }
    }
  }
}
```

## 创建索引时指定search_analyzer
```json
PUT test_index
{
  "mappings": {
    "doc": {
      "properties": {
        "title":{
          "type": "text",
          "analyzer": "whitespace",
          "search_analyzer": "standard"
        }
      }
    }
  }
}
```

# 总结
本文简单介绍了ES中分词器的基本概念、构造及使用方法，并给出了一些简单的例子。

如果对分词器有进一步的学习需要，可以参考ElasticSearch官网。如果需求复杂的分词器，也可参考[IK分词器 github页面](https://github.com/medcl/elasticsearch-analysis-ik)

注：本文参考来源：
- 作者本人实践
- [Elasticsearch如何定制分词器 (自定义分词策略) ](https://www.cnblogs.com/shoufeng/p/10562746.html)
- [es的分词器analyzer ](https://www.cnblogs.com/xiaobaozi-95/p/9328948.html)
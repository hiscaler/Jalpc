---
layout: post
title:  "ElasticSearch CURD"
date:   2019-08-19
desc: "ElasticSearch CURD"
keywords: "es,elasticsearch"
categories: [Database]
tags: [es,elasticsearch]
icon: icon-db
---

## 创建索引

```json
PUT /{index}
{
    "settings": { ... any settings ... },
    "mappings": {
    "type_one": { ... any mappings ... },
    "type_two": { ... any mappings ... },
    ...
}
```

你可以通过在 config/elasticsearch.yml 中添加下面的配置来防止自动创建索引。

```ini
action.auto_create_index: false
```

## 删除索引
- DELETE /{index}
- DELETE /index_1,index_2
- DELETE /index_*
- DELETE /_all

## 索引设置

- number_of_shards

  定义一个索引的主分片个数，默认值是 `5`。这个配置在索引创建后不能修改。

- number_of_replicas

  每个主分片的复制分片个数， 默认是 `1`。 这个配置可以随时在活跃的索引上修改。 

例子：

我们可以创建只有一个主分片， 没有复制分片的小索引。 

```json
PUT /my_temp_index
{
    "settings": {
        "number_of_shards" : 1,
        "number_of_replicas" : 0
    }
}
```

然后，我们可以用 update-index-settings API 动态修改复制分片个数：

```json
PUT /my_temp_index/_settings
{
    "number_of_replicas": 1
}
```

## 配置分析器

用来配置已存在的分析器或创建自定义分析器来定制化你的索引。

standard 分析器是用于全文字段的默认分析器， 对于大部分西方语系来说是一个不错的选择。 它考虑了以下几点：

- standard 分词器， 在词层级上分割输入的文本。

- standard 表征过滤器， 被设计用来整理分词器触发的所有表征（但是目前什么都没做） 。

- lowercase 表征过滤器， 将所有表征转换为小写。

- stop 表征过滤器， 删除所有可能会造成搜索歧义的停用词， 如 a ， the ， and ， is 。

默认情况下，停用词过滤器是被禁用的。如需启用它，你可以通过创建一个基于 standard 分析器的自定义分析器，并且设置 stopwords 参数。可以提供一个停用词列表， 或者使用一个特定语言的预定停用词列表。
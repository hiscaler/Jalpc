---
layout: post
title:  "ElasticSearch 设置"
date:   2019-08-19
desc: "ElasticSearch 设置"
keywords: "es,elasticsearch"
categories: [Database]
tags: [es,elasticsearch]
icon: icon-db
---

## 增加复制分片数量

```json
PUT /index_name/_settings
{
    "number_of_replicas": 2
}
```

## 查看索引映射

```json
GET /{index}/_mapping
```

核心数据类型

- String -> string
- Whole number -> byte, short, interger,long
- Floating point -> float, double
- Boolean -> boolean
- Date -> date

动态映射猜测字段类型

- "true" or "false" -> boolean
- 123 -> long
- "123" -> string
- 123.45 -> double
- 2014-09-09 -> date
- 2014-02-30 -> string
- foo bar -> string

对于 string 字段， 两个最重要的映射参数是 index 和 analyer 。 

- index

  index 参数控制字符串以何种方式被索引。 它包含以下三个值当中的一个： 

  | analyzed     | 首先分析这个字符串， 然后索引。 换言之， 以全文形式索引此字段。 |
  | ------------ | ------------------------------------------------------------ |
  | not_analyzed | 索引这个字段， 使之可以被搜索， 但是索引内容和指定值一样。 不分析此字段。 |
  | no           | 不索引这个字段。 这个字段不能为搜索到。                      |

  其他简单类型，比如 long 、 double 、 date 等等也接受 index 参数， 但相应的值只能是 no 和 not_analyzed ， 它们
  的值不能被分析。 

- analyer

  对于 analyzed 类型的字符串字段， 使用 analyzer 参数来指定哪一种分析器将在搜索和索引的时候使用。 默认的，
  Elasticsearch使用 standard 分析器， 但是你可以通过指定一个内建的分析器来更改它， 例如 whitespace 、 simple 或 english 。

  ```json
  {
      "tweet": {
          "type": "string",
          "analyzer": "english"
      }
  } 
  ```

  ## 更新映射

  **你可以向已有映射中增加字段， 但你不能修改它。 如果一个字段在映射中已经存在， 这可能意味着那个字段的数据已经被索引。 如果你改变了字段映射， 那已经被索引的数据将错误并且不能被正确的搜索到。** 

  **我们可以更新一个映射来增加一个新字段， 但是不能把已有字段的类型那个从 analyzed 改到 not_analyzed 。**

  ```json
  // 错误
  PUT /{index}/_mapping
  {
  	"properties": {
  		"title": {
  			"type": "string",
  			"index": "not_analyzed"
  		}
  	}
  }
  ```

  
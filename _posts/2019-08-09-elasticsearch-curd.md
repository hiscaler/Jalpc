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

## 添加文档

- 使用自己的 ID

```json
PUT /{index}/{id}
{
    "field": "value",
    ...
}
```

- 自增 ID

```json
POST /{index}
{
    "field": "value",
    ...
}
```
### 创建一个新文档
```json
PUT {index}/_doc/{id}/_create
{
  "keyA": "value A",
  "keyB": "value B"
}
```
如果请求成功的创建了一个新文档， Elasticsearch将返回正常的元数据且响应状态码是 201 reated 。另一方面， 如果包含相同的 _index 、 _type 和 _id 的文档已经存在， Elasticsearch将返回 409 Conflict 响应状态码。

## 检索文档

### 检索单个文档

```json
GET {index}/_search
{
  "query": {
    "match": {
      "_id": "123456789"
    }
  },
  "size": 1,
  "_source": [
    "id",
    "key"
  ]
}
```

_source: 需要返回的元数据

### 检索文档是否存在

```json
HEAD {index}/_doc/123456789
```

### 检索多个文档

```json
GET /{index}/_mget
{
  "ids": [
    "98175",
    "98170"
  ]
}
````

## 更新文档

### 更新整个文档

```json
PUT {index}/_doc/{id}
{
  "keyA": "value A",
  "keyB": "value B"
} 
```

**文档不能被更改， 只能被替换 。**

### 文档局部更新

#### 基于请求

```json
POST {index}/_update/{id}
{
  "doc": {
    "keyA": "value AA"
  }
}
```

#### 基于脚本

```json
POST {index}/_update/{id}
{
  "script": "ctx._source.times+=1"
}

{
  "script": "ctx._source.keyA+='newValue'"
}
// Delete
{
  "script": "ctx.op = ctx._source.times == 3 ? 'delete' : 'none'"
}
```

失效的参数：
```json
"params" : {
    "new_tag" : "search"
}
```

### 更新可能不存在的文档

```json
POST /{index}/_update/{id}
{
  "script": "ctx._source.views+=1",
  "upsert": {
    "views": 1
  }
}
```

### 更新和冲突

通过 retry_on_conflict 参数设置重试次数来自动完成， 这样 update 操作将会在发生错误前重试——这个值默认为
0。

```json
POST /{index}/_update/{id}?retry_on_conflict=5
{
  "script": "ctx._source.views+=1",
  "upsert": {
    "views": 0
  }
}
```

## 删除文档

```json
DELETE {index}/_doc/{id}
```

**删除一个文档也不会立即从磁盘上移除， 它只是被标记成已删除。 Elasticsearch 将会在你之后添加更多索引的时候才会在后台进行删除内容的清理。**

## 批量操作

- 格式

  ```json
  {
      "action": {metadata}\n
      {request body}\n
      "action": {metadata}\n
      {request body}\n
  }
  ```

  `action/metadata` 这一行定义了文档行为(what action)发生在哪个文档(which document)之上。action 必须是以下几种：

  create: 当文档不存在时创建之。

  index: 创建新文档或替换已有文档。 

  update: 局部更新文档。

  delete: 删除一个文档。

- 示例

  ```json
   // 不同的 index
  POST /_bulk
  {"create":{"_index":"{index}","_id":"123"}}
  {"title":"My first blog post"}
  
  // 相同的 index（你依旧可以覆盖元数据行的 _index ， 在没有覆盖时它会使用URL中的值作为默认值。）
  POST /{index}/_bulk
  {"create":{"_id":"123"}}
  {"title":"My first blog post 1"}
  {"create":{"_id":"1234"}}
  {"title":"My first blog post 2"}
  ```

  - 注意

    bulk 请求不是原子操作——它们不能实现事务。 每个请求操作时分开的， 所以每个请求的成功与否不干扰其它操作。 

## 处理冲突

在数据库中， 有两种通用的方法确保在并发更新时修改不丢失： 

### 悲观并发控制（Pessimistic concurrency control） 

这在关系型数据库中被广泛的使用， 假设冲突的更改经常发生， 为了解决冲突我们把访问区块化。 典型的例子是在读一行数据前锁定这行， 然后确保只有加锁的那个线程可以修改这行数据。 

### 乐观并发控制（Optimistic concurrency control） 

被 Elasticsearch 使用， 假设冲突不经常发生， 也不区块化访问， 然而， 如果在读写过程中数据发生了变化， 更新操作将失败。 这时候又程序决定在失败后如何解决冲突。 实际情况中， 可以重新尝试更新， 刷新数据（重新读取） 或者直接反馈给用户 。

```json
PUT {index}/_doc/{id}?version=1
{
  "keyA": "value A",
  "keyB": "value B"
} 
```

**所有更新和删除文档的请求都接受 version 参数， 它可以允许在你的代码中增加乐观锁控制。**

### 使用外部版本控制系统

一种常见的结构是使用一些其他的数据库做为主数据库， 然后使用 Elasticsearch 搜索数据， 这意味着所有主数据库发生变化， 就要将其拷贝到 Elasticsearch 中。 如果有多个进程负责这些数据的同步， 就会遇到上面提到的并发问题。

如果主数据库有版本字段——或一些类似于 timestamp 等可以用于版本控制的字段——是你就可以在 Elasticsearch 的查询字符串后面添加 version_type=external 来使用这些版本号。 版本号必须是整数， 大于零小于 9.2e+18 ——Java中的正的 long 。

外部版本号与之前说的内部版本号在处理的时候有些不同。 它不再检查 _version 是否与请求中指定的一致， 而是检查是否小于指定的版本。 如果请求成功， 外部版本号就会被存储到 _version 中。
外部版本号不仅在索引和删除请求中指定， 也可以在创建(create)新文档中指定。 

## 查询与过滤

### 解释

我们可以使用两种结构化语句： 结构化查询（Query DSL） 和结构化过滤（Filter DSL）。查询与过滤语句非常相似， 但是它们由于使用目的不同而稍有差异。

**一条过滤语句会询问每个文档的字段值是否包含着特定值**

- 是否 created 的日期范围在 2013 到 2014 ?

- 是否 status 字段中包含单词 "published" ?

- 是否 lat_lon 字段中的地理位置与目标点相距不超过10km ?

**一条查询语句与过滤语句相似，但问法不同。查询语句会询问每个文档的字段值与特定值的匹配程度如何？查询语句的典型用法是为了找到文档**

  - 查找与 full text search 这个词语最佳匹配的文档

  - 查找包含单词 run ， 但是也包含 runs , running , jog 或 sprint 的文档

  - 同时包含着 quick , brown 和 fox --- 单词间离得越近， 该文档的相关性越高

  - 标识着 lucene , search 或 java --- 标识词越多， 该文档的相关性越高


一条查询语句会计算每个文档与查询语句的相关性，会给出一个相关性评分 _score ，并且按照相关性对匹配到的文档进行排序。 这种评分方式非常适用于一个没有完全配置结果的全文本搜。 

### 性能差异

一般情况下， 一条经过缓存的过滤查询要远胜一条查询语句的执行效率。 

### 什么情况下使用

原则上来说，使用查询语句做全文本搜索或其他需要进行相关性评分的时候，剩下的全部用过滤语句 。

### 语句示例

#### term 过滤

term 主要用于精确匹配哪些值，比如数字，日期，布尔值或 not_analyzed 的字符串（未经分析的文本数据类型）

```json
{
    {
        "term": {
            "age": 26
        }
    }, 
    {
        "term": {
            "date": "2014-09-01"
        }
    }, 
    {
        "term": {
            "public": true
        }
    }, 
    {
        "term": {
            "tag": "full_text"
        }
    }
}
```

#### terms 过滤

terms 跟 term 有点类似， 但 terms 允许指定多个匹配条件。如果某个字段指定了多个值，那么文档需要一起去做匹配 

```json
{
    "terms": {
        "tag": [
            "search", 
            "full_text", 
            "nosql"
        ]
    }
}
```

#### range 过滤

range 过滤允许我们按照指定范围查找一批数据

```json
{
    "range": {
        "age": {
            "gte": 20, 
            "lt": 30
        }
    }
}
```

范围操作符包含：

- gt : 大于
- gte : 大于等于
- lt : 小于
- lte : 小于等于

#### exists 和 missing 过滤

exists 和 missing 过滤可以用于查找文档中是否包含指定字段或没有某个字段， 类似于SQL语句中的 IS_NULL 条件

```json
{
    "exists": {
        "field": "title"
    }
}
```

#### bool 过滤

bool 过滤可以用来合并多个过滤条件查询结果的布尔逻辑，它包含以下操作符

- must : 多个查询条件的完全匹配，相当于 AND

- must_not : 多个查询条件的相反匹配，相当于 NOT

- should : 至少有一个查询条件匹配，相当于 OR

这些参数可以分别继承一个过滤条件或者一个过滤条件的数组

#### match_all 查询

使用 match_all 可以查询到所有文档，是没有查询条件下的默认语句。

```json
{
    "match_all": {}
}
```

#### match 查询

match 查询是一个标准查询，不管你需要全文本查询还是精确查询基本上都要用到它。

```json
{
    "match": {
        "age": 26
    }
}, 
{
    "match": {
        "date": "2014-09-01"
    }
}, 
{
    "match": {
        "public": true
    }
}, 
{
    "match": {
        "tag": "full_text"
    }
}
```



它只能就指定某个确切字段某个确切的值进行搜索，而你要做的就是为它指定正确的字段名以避免语法错误。

#### multi_match 查询

multi_match 查询允许你做 match 查询的基础上同时搜索多个字段

```json
{
    "multi_match": {
        "query": "full text search", 
        "fields": [
            "title", 
            "body"
        ]
    }
}
```

#### bool 查询

bool 查询与 bool 过滤相似，用于合并多个查询子句。不同的是，bool 过滤可以直接给出是否匹配成功，而 bool 查询要计算每一个查询子句的 _score（相关性分值）。

- must : 查询指定文档一定要被包含。
- must_not : 查询指定文档一定不要被包含。
- should : 查询指定文档，有则可以为文档相关性加分。 

以下查询将会找到 title 字段中包含 "how to make millions"， 并且 "tag" 字段没有被标为 spam 。如果有标识为 "starred" 或者发布日期为 2014 年之前，那么这些匹配的文档将比同类网站等级高：

```json
{
    "bool": {
        "must": {
            "match": {
                "title": "how to make millions"
            }
        }, 
        "must_not": {
            "match": {
                "tag": "spam"
            }
        }, 
        "should": [
            {
                "match": {
                    "tag": "starred"
                }
            }, 
            {
                "range": {
                    "date": {
                        "gte": "2014-01-01"
                    }
                }
            }
        ]
    }
}
```

#### 查询与过滤条件的合并

查询语句和过滤语句可以放在各自的上下文中。 在 ElasticSearch API 中我们会看到许多带有 query 或 filter 的语句。
这些语句既可以包含单条 query 语句，也可以包含一条 filter 子句。换句话说，这些语句需要首先创建一个 query 或 filter 的上下文关系。

复合查询语句可以加入其他查询子句，复合过滤语句也可以加入其他过滤子句。通常情况下，一条查询语句需要过滤语句的辅助，全文本搜索除外。

所以说，查询语句可以包含过滤子句，反之亦然。以便于我们切换 query 或 filter 的上下文。这就要求我们在读懂需求的同时构造正确有效的语句。

##### 带过滤的查询语句

在收件箱中查询 email 内容包括“business opportunity” 的邮件

```json
GET /_search
{
    "query": {
        "filtered": {
            "query": {
                "match": {
                    "email": "business opportunity"
                }
            }, 
            "filter": {
                "term": {
                    "folder": "inbox"
                }
            }
        }
    }
}
```

##### 单条过滤语句

过滤出收件箱总的所有记录

```json
GET /_search
{
    "query": {
        "filtered": {
            "filter": {
                "term": {
                    "folder": "inbox"
                }
            }
        }
    }
}
```
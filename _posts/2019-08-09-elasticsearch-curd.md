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
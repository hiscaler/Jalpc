---
layout: post
title:  "ElasticSearch 设置"
date:   2019-08-19
desc: "ElasticSearch 设置"
keywords: "es,elasticsearch"
categories: [es]
tags: [es,elasticsearch]
icon: icon-database
---

## 增加复制分片数量

```json
PUT /index_name/_settings
{
    "number_of_replicas": 2
}
```


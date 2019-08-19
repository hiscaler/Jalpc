---
layout: post
title:  "ElasticSearch 知识点"
date:   2019-08-19
desc: "ElasticSearch 知识点"
keywords: "es,elasticsearch"
categories: [es]
tags: [es,elasticsearch]
icon: icon-db
---

## 名词解释

- Node: 一个 Elasticsearch 实例
- Cluster: 由一个或者多个 Node 组成，他们具有相同的 cluster.name，他们协同工作，分享数据和负载，当有新的节点加入或者移除，集群会感知并平衡数据。
- Index: 索引（一个存储关联数据的地方，实际上， 索引只是一个用来指向一个或多个分片shards)的“逻辑命名空间(logical namespace)” ）
- Shard: 分片。一个最小级别“工作单元(worker unit)”，它只是保存索引中所有数据的一小片。
  1. Primary Shard: 主分片（索引中的每个文档属于一个单独的主分片 ）
  2. Replica Shard: 复制分片（复制分片只是主分片的一个副本， 它用于提供数据的冗余副本， 在硬件故障之后提供数据保护， 同时服务于像搜索和检索等只读请求。 ）

集群中一个节点会被选举为主节点（master），它用来管理集群中的一些变更， 例如新建或删除索引、 增加或移除节点等。 主节点不需要参与文档级别的更改或搜索， 这意味着只有一个主节点不会随着流量的增长而成为集群的瓶颈。

做为用户， 我们能够与集群中的任何节点(any node in the cluster)通信， 包括主节点。 任何一个节点互相知道文档存在于哪个节点上， 它们可以转发请求到我们需要数据所在的节点上。 我们通信的节点负责收集各节点返回的数据， 最后一起返回给客户端。 这一切都由 Elasticsearch 透明的管理。

## 文档元数据

- _index: 文档存储的地方_
- type: 文档代表对象的类
- _id: 文档的唯一标识

## 集群状态

查询：GET /_cluster/health

- green: 所有主要和复制的分片都可用
- yellow: 所有主分片可用，但不是所有复制分片都可用
- red: 不是所有的主分片都可用
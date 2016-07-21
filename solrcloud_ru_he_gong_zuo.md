# SolrCloud 如何工作

## SolrCloud 关键概念

一个 SolrCloud 集群由一些物理概念之上的逻辑概念组成

### 逻辑概念

* 一个集群(cluster)可以承载多个文档(document)的集合(collection)
* 一个集合(collection)可以被划分为多个分片(shard)，每一个分片(shard)都包含了集合(collection)的全部文档(document)的子集
* 决定集合(collection)划分为几个分片(shard)的是
  * 集合(collection)包含多少文档(document)，这里说的是理论上限
  * 并发的查询请求数量

### 物理概念

* 一个集群(cluster)由一个或多个节点(node)组成，每个节点(node)都运行着 Solr 服务进程实例
* 每个节点(node)可以承载多个核(core)
* 集群(cluster)中的每个核(core)是一个逻辑分片(shard)的物理副本(replica)
* 集合(cluster)里的每个副本(replica)使用相同的配置
* 决定每个分片(shard)里有几个副本(replica)的是
  * 构建集合(collection)的冗余度，及集群(cluster)里的某几个节点(node)不可用时的容错能力
  * 在重负载时，可以同时处理的查询请求，这里说的是理论上限
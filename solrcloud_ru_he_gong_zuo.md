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

A Cluster is made up of one or more Solr Nodes, which are running instances of the Solr server process.
Each Node can host multiple Cores.
Each Core in a Cluster is a physical Replica for a logical Shard.
Every Replica uses the same configuration specified for the Collection that it is a part of.
The number of Replicas that each Shard has determines:
Apache Solr Reference Guide 6.0 548
1.
2.
3.
The level of redundancy built into the Collection and how fault tolerant the Cluster can be in the
event that some Nodes become unavailable.
The theoretical limit in the number concurrent search requests that can be processed under heavy
load.
# SolrCloud

Apache Solr 包含了创建集群来结合容错和高可用的能力，即 **SolrCloud**，提供了分布式索引和查询的能力，支持如下的特性

* 集中配置整个集群
* 查询的自动负载均衡及故障切换
* 集成 Zookeeper 来协调和配置集群

SolrCloud 是一个灵活的、分布的查询和索引，无需中心节点来分配节点、分片和复制。Solr 使用 ZooKeeper ，基于配置文件和 schemas 来管理这些场景。文档可以发送到任何服务器上，ZooKeeper 总能找到它。
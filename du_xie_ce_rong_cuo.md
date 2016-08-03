# 读写侧容错

SolrCloud 支持伸缩性，高可用和读写容错。这意味着，如果你有个大的集群，总是可以发送请求到集群：只要有可能，读请求总是能返回结果，即使某些节点宕机；写请求将收到回执。你不会丢失数据

## 读容错

SolrCloud 集群里，读请求是集合里的节点来均衡负载的。你仍然需要一个外部的负载均衡器来，或者一个智能客户端，能明白如何读 ZooKeeper 里的元数据来发现应向哪个节点发送请求。

Solr 提供了一个 Java 的客户端 [CloudSolrClient](http://lucene.apache.org/solr/6_0_0/solr-solrj/org/apache/solr/client/solrj/impl/CloudSolrClient.html)

### *zkConnected*



### *shards.tolerant*

## 写容错

### *Recovery*

### *Achieved Replication Factor*



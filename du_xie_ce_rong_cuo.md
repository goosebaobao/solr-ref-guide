# 读写侧容错

SolrCloud 支持伸缩性，高可用和读写容错。这意味着，如果你有个大的集群，总是可以发送请求到集群：只要有可能，读请求总是能返回结果，即使某些节点宕机；写请求将收到回执。你不会丢失数据

## 读容错

SolrCloud 集群里，读请求是集合里的节点来均衡负载的。你仍然需要一个外部的负载均衡器来，或者一个智能客户端，能明白如何读 ZooKeeper 里的元数据来发现应向哪个节点发送请求。

Solr 提供了一个 Java 的客户端 [CloudSolrClient](http://lucene.apache.org/solr/6_0_0/solr-solrj/org/apache/solr/client/solrj/impl/CloudSolrClient.html)

### *zkConnected*

一个 Solr 节点会在它能联系到每个分片的至少一个副本时返回查询结果，甚至是在它收到请求时无法联系到 ZooKeeper。为了容错这通常是优先的做法，但是可能会得到错误的结果或者脏数据，如果整个集合的结构发生了大的变化而该节点还没有能从 ZooKeeper 得到通知。例如，新增或移除了分片，或者某个分片切分成子分片

每个查询结果都包含一个 `zkConnected` 头来指示节点是否在处理请求时连接到 ZooKeeper

```json
{
  "responseHeader": {
    "status": 0,
    "zkConnected": true,
    "QTime": 20,
    "params": {
      "q": "*:*"
    }
  },
  "response": {
    "numFound": 107,
    "start": 0,
    "docs": [ ... ]
  }
}
```

### *shards.tolerant*

如果一个或多个分片不可用，Solr 默认的行为是请求失败。但是，很多场景部分结果是可接受的，所以 Solr 提供一个逻辑参数 `shards.tolerant` (默认`false`)。如果 `shards.tolerant=true`，会返回部分结果。如果返回的结果不包含所有分片，返回头里有一个特殊的标识 `partialResults`。客户端可以同时指定 `shards.info` 参数来获取更多的细节

```json
{
  "responseHeader": {
    "status": 0,
    "zkConnected": true,
    "partialResults": true,
    "QTime": 20,
    "params": {
      "q": "*:*"
    }
  },
  "response": {
    "numFound": 77,
    "start": 0,
    "docs": [ ... ]
  }
}
```

## 写容错

### *Recovery*

### *Achieved Replication Factor*



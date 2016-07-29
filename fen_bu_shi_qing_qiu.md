# 分布式请求

当一个 Solr 节点接收到搜索请求，这个请求在后台被路由到被搜索集合的某个分片的一个副本上。这个被选中的副本担当一个聚合器：创建内部请求到集合的所有分片的某个随机的副本，协调所有响应，发出任何必要的后续内部请求（例如，改善 facet 值，或请求额外的存储字段），构建返回给客户端的最终响应

## 限制哪些分片(shards)被查询

使用 SolrCloud 的一个好处是，多个分片组成的大集合，但在某些案例里，你只对分片的子集返回的结果感兴趣。你可以选择查询所有还是部分分片。

在集合的所有分片上查询，看上去眼熟，就好像没有运行 SolrCloud（意思是在所有分片上查询的语法和单核）

```
http://localhost:8983/solr/gettingstarted/select?q=*:*
```

另一方面，如果你想只在一个分片上搜索，你可以指定分片的逻辑 id，如

```
http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=shard1
```

如果想在一组分片上搜索，可以一起指定

```
http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=shard1,shard2
```

在上面的例子里，分片 id 用来选择该分片的一个随机的副本

你也可以明确指定想要的副本取代分片 id

```
http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=localhost:7574/solr/ge
ttingstarted,localhost:8983/solr/gettingstarted
```

或者你也可以用管道符(|)指定一个分片的副本列表来选择

```
http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=localhost:7574/solr/ge
ttingstarted|localhost:7500/solr/gettingstarted
```

当然，你可以指定一个逗号分隔的分片列表，每一个都由管道符(|)分隔多个副本，下面例子里，2 个分片被查询，第一个分片是随机选择的副本，第二个分片是在管道符(|)限定的副本里随机一个

```
http://localhost:8983/solr/gettingstarted/select?q=*:*&shards=shard1,localhost:7574/
solr/gettingstarted|localhost:7500/solr/gettingstarted
```

## 配置 ShardHandlerFactory

你可以在 Solr 分布式查询里配置并发和线程池，更好的细粒度控制和调整，默认的配置有利于吞吐量。

配置标准的处理器(handler)，在 solrconfig.xml 里按如下例子配置

```xml
<requestHandler name="standard" class="solr.SearchHandler" default="true">
  <!-- other params go here -->
  <shardHandler class="HttpShardHandlerFactory">
    <int name="socketTimeOut">1000</int>
    <int name="connTimeOut">5000</int>
  </shardHandler>
</requestHandler>
```


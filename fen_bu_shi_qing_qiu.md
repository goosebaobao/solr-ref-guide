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

可用的参数列表

| 参数 | 默认值 | 含义 |
| -- | -- | -- |
| `socketTimeout` | 0 (使用操作系统默认值) | socket 等待的时间，毫秒 |
| `connTimeout` | 0 (使用操作系统默认值) | 接受绑定或连接 socket 的时间，毫秒 |
| `maxConnectionsPerHost` | 20 | 分布式查询时每个分片最大的并发连接数 |
| `maxConnections` | 10000 | 分布式查询时总的并发连接上限数 |
| `corePoolSize` | 0 | 分布式查询时线程数下限 |
| `maximumPoolSize` | Integer.MAX_VALUE | 分布式查询时线程数最大值 |
| `maxThreadIdleTime` | 5 | 线程在回收前等待的时间，秒 |
| `sizeOfQueue` | -1 | 如果指定，线程池将用一个队列来替代直接切换缓冲区。高吞吐量的系统设为 -1 来使用直接切换缓冲区，希望更好的延迟的系统配置一个合理的队列大小来处理变化的请求 |
| `fairnessPolicy` | false | 为 true，则分布式查询使用先进先出方式来消费，false 则吞吐量是延迟处理 |

## 配置 statsCache(分布式 IDF)

为计算关联度（匹配度？）需要统计文档和词条。Solr 提供 4 种开箱即用的实现

* `LocalStatsCache`：只用局部词条和文档来统计关联度。对于分片上的统一术语，这个很不错
* `ExactStatsCache`：使用全局值来统计文档频率
* `ExactSharedStatsCache`：和 ExactStatsCache 一样，但是相同词条的后续请求不被统计
* `LRUStatsCache`：用一个 LRU(Least Recently Used，最近最少使用) 算法的 cache 来保存全局统计

在 `solrconfig.xml` 文件里，使用 `<statsCache>` 标签来配置。示例：

```xml
<statsCache class="org.apache.solr.search.stats.ExactStatsCache"/>
```


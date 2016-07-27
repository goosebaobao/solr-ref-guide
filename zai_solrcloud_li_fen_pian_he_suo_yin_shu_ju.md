# 在 SolrCloud 里分片和索引数据

当你的数据对一个节点来说太大时，你可以创建多个分片，将数据分散存储于其中。

在 SolrCloud 之前，Solr 支持分布式查询，即一个查询跨越多个分片，整个索引上执行而不会遗漏任何文档。所以将核划分成分片并非 SolrCloud 独有的概念。但是，有几个需要 SolrCloud 改善的分布式的问题：

1. 将 core 分片是手工滴
2. 不支持分布式索引，这意味着你要把文档发送到一个特定的分片，而 Solr 自己不知道要把文档发送到哪个分片
3. 没有负载均衡或故障切换，如果你有大量的查询，你需要知道发送给谁

SolrCloud 解决了这些问题。它支持分布式索引和查询，ZooKeeper 提供了故障切换和负载均衡。此外，每个分片可以有多个副本来提升健壮性

在 SolrCloud 里没有主(master)或从(slave)，取而代之的是，领导(leader)和副本(replica)。领导是自动选举的，初始是基于先来先服务原则，然后就是基于 ZooKeeper 的选举

如果一个领导挂了，它的某个副本会被自动选为新的领导。如果每个节点都启动了，会分配给最少副本的分片，如果有个限制，会分配给 shard id 最小的分片。（这一段貌似是说哪个分片做 leader）

当一个文档发送给机器来索引，系统首先判断该机器是否领导或副本

* 如果该机器是个副本，文档转发给领导来处理
* 如果该机器是个领导，SolrCloud 判断文档应去往哪个分片，转发文档到分片的领导，为这个分片做索引，并且转发索引笔记到自己和所有副本

## 文档路由

创建一个 collection 时，用 `router.name` 参数来指定路由器实现。如果你使用 `"compositeId"` 路由器，可以在文档 id 中添加前缀来计算 hash 值，solr 使用这个值来决定文档被发送到哪个分片去索引。前缀可以是任何你喜欢的东东（不一定要用分片的名字），但应该是一致的。例如，如果你想要把一个客户的文档放在一起，可以把客户的 id 或名字作为前缀。如果客户名字是 "IBM"，有个文档 id 为 "12345"，可以把前缀插入 id 字段，即 "IBM!12345"，感叹号 ("!") 很重要，它表明了用于决定文档所处分片的前缀。

然后，在查询时，使用 `_route_` 参数将前缀包含在查询中（例如，`q=solr&_route_=IBM!`）来定位分片。在某些场景下，这会改善查询性能，因为在查询所有分片时不用考虑网络因素

> `_route_` 参数取代了 `shard.keys`，后者已废弃并将在 Solr 的后续版本被移除

`compositeId` 路由器支持 2 层前缀。例如，一个前缀首先路由到区域，然后是客户："USA!IBM!12345"

另一个用例，客户 "IBM" 有很多文档，你想要把他们分割到多个分片。语法是这样的：  "shard_key/num!document_id"，/num 是用于 hash 的shard_key 的比特位的数量

所以，"IBM/3!12345" 会为 shard_key 占用 3 位，文档 id 占用 29 位。这样就将分片划分为 8 份。如果 num 为 2，那么就是将文档划分为 4 份。

查询时，用 `_route_` 参数来指定分片，如 `q=solr&_route_=IBM/3!`

如果不想影响文档如何存储，就不要在文档 id 里指定前缀

如果创建 collection 时，使用 "implicit" 路由器，`router.field` 参数可以指定一个字段来识别文档属于哪个分片，木有这个字段的文档会被拒绝，同样可以用 `_route_` 参数来命名指定的分片

## 切分分片

在 SolrCloud 里创建 collection 时，你要决定初始的分片数目。很难确定需要多少个分片，特别是需求随时会变化，并且事后发现数目不对的代价很高，牵涉到创建新的核和重新索引所有数据

分片要使用 Collection API，当前允许将 1 个分片切分为 2 份。已存在的分片依然保留，所以这个切分动作实际上是创建了 2 个副本作为分片。当你准备好时，可以删除老的分片。

## 忽略客户端提交

大多数场景下，在运行 SolrCloud 时，索引客户端不应发送明确的提交请求。相反的，应该用 `openSearcher=false` 配置自动提交，自动软提交，以使最新的更新能在查询中可见。这能确保在及群里自动提交是按预定计划发生的。为了强调客户端不应该明确提交的策略，应该更新所有索引数据到 SolrCloud 的客户端。不过这并非总是可行的，所以 Solr 提供了 IgnoreCommitOptimizeUpdateProcessorFactory，允许你忽略明确的提交，并且/或者优化客户端请求而不需要重构客户端代码。要激活这个请求处理器，你需要将下面内容添加到 solrconfig.xml

```xml
<updateRequestProcessorChain name="ignore-commit-from-client" default="true">
  <processor class="solr.IgnoreCommitOptimizeUpdateProcessorFactory">
    <int name="statusCode">200</int>
  </processor>
  <processor class="solr.LogUpdateProcessorFactory" />
  <processor class="solr.DistributedUpdateProcessorFactory" />
  <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>
```

如上面的例子，处理器返回 200 给客户端，但是会忽略提交/优化请求。注意你需要把 SolrCloud 必需的隐含处理器串联起来，毕竟这个自定义的处理器链会取代默认的处理器链

下面的例子会抛出一个代码为 403 的异常，及定制的错误消息

```xml
<updateRequestProcessorChain name="ignore-commit-from-client" default="true">
  <processor class="solr.IgnoreCommitOptimizeUpdateProcessorFactory">
    <int name="statusCode">403</int>
    <str name="responseMessage">Thou shall not issue a commit!</str>
  </processor>
  <processor class="solr.LogUpdateProcessorFactory" />
  <processor class="solr.DistributedUpdateProcessorFactory" />
  <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>
```

最后，你也可以配置为忽略优化，让提交通过

```xml
<updateRequestProcessorChain name="ignore-optimize-only-from-client-403">
  <processor class="solr.IgnoreCommitOptimizeUpdateProcessorFactory">
    <str name="responseMessage">Thou shall not issue an optimize, but commits are OK!</str>
    <bool name="ignoreOptimizeOnly">true</bool>
  </processor>
  <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>
```
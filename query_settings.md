# Query Settings

本章节的设置影响了 solr 处理和响应请求的方式。这些设置全都在 `solrconfig.xml` 的 `<query>` 元素下

```xml
<query>
  ...
</query>
```

## cache

solr cache 和确定的 index searcher 实例关联，一个 searcher 生命期里不会改变的索引的明确视图。只要那个 index searcher 被使用，它的 cache 里的任何项目都是有效和可重用的。solr caching 和其他许多应用的 caching 不同，solr 里缓存的对象不会在一段时间间隔后过期；它们会在 index searcher 生命期里保持有效

当一个新的 searcher 被开启，当前的 searcher 继续为请求服务，同时新的 searcher 自动预热其缓存。新的 searcher 用当前 searcher 的缓存来预填充自己的。当新的 searcher 就绪，它注册成为当前的 searcher 并开始处理所有新的搜索请求。旧的 searcher 完成所有的请求后被关闭

solr 有 3 个 cache 实现：`solr.search.LRUCache`, `solr.search.FastLRUCache`, `solr.search.LFUCache`

缩写 LRU 代表 Least Recently Used(最近最少使用)，当一个 LRU cache 充满，最后访问的最老的对象会被清理，给新的对象提供空间。结果就是频繁访问的对象会留在 cache，同时那些不是频繁访问的对象被清理，如果需要的话会重新从索引里取到 cache

`FastLRUCache`，solr 1.4 已引入，无锁设计(<font color='red'>lock-free，相对于 `LRUCache` 可以做到读写不加锁</font>)，所以很适合缓存那些在一个请求里被命中多次的对象

`LRUCache` 和 `FastLRUCache` 在预热时，用一个整数或百分比来评估相对于当前 cache 大小的的自动预热数(<font color='red'>意思是初始化时预加载多少数据吗?</font>)

`LFUCache` 引用 Least Frequently Used(最近最不常用) cache。工作方式类似 LRU cache，除了当 cache 充满时最少使用的对象会被淘汰。

> **LRU 和 LFU 区别**
>
> * LRU 根据最后一次命中时间来淘汰数据
> * LFU 淘汰在一个时间段内命中次数最少的
> 
> 例如，一个空闲很久的对象上一秒刚刚被命中，LRU 很大概率保留该对象，LFU 很大概率淘汰该对象
> 
> by goosebaobao

solr 管理界面的统计页会显示所有活动 cache 的执行信息，可以帮助你调整适合应用的 cache 尺寸。当一个 searcher 结束，其 cache 的使用概要会写入日志。

每个 cache 都有定义初始尺寸(`initialSize`)，最大尺寸(`size`)，预热时的对象数量(`autowarmCount`)的设置。 LRU 和 FastLRUCache 实现可以用百分比来代替 `autowarmCount` 的绝对数量值

以下是各个 cache 的细节描述

### filterCache

`SolrIndexSearcher` 用这个 cache 过滤匹配一个查询的所有未排序的文档集。数值属性用来控制 cache 里的对象数量。

solr 用 `filterCache` 来缓存使用 `fq` 参数的查询结果。后续使用相同参数的查询命中并从 cache 返回结果。参考 [搜索](sou_suo.md) 了解 `fq` 参数详情。


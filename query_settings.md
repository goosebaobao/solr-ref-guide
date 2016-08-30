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

solr 用 `filterCache` 来缓存使用 `fq` 参数的查询结果。后续使用相同参数的查询命中并从 cache 返回结果。参考 [搜索](pu_tong_cha_xun_can_shu.md) 了解 `fq` 参数详情。

当配置参数 `facet.method` 设置为 `fc`，solr 也把这个用于缓存 faceting。

```xml
<filterCache class="solr.LRUCache" 
    size="512" 
    initialSize="512" 
    autowarmCount="128"/>
```

### documentCache

介个 cache 保存 Lucene 文档对象(每个文档的存储字段)。由于 Lucene 内部的文档 id 是临时的，这个 cache 不会自动预热。`documentCache` 的大小应该总是大于 `max_results` 乘以 `max_concurrent_queries`，以确保不需要在一个请求里重新获取文档。你的文档存储的字段越多，这个 cache 占用的内存越多。

```xml
<documentCache class="solr.LRUCache" 
    size="512" 
    initialSize="512" 
    autowarmCount="0"/>
```

### 使用自定义 cache

你也可以定义一个用你自己的代码的 cache。可以调用 `SolrIndexSearch` 的方法 `getCache()`， `cacheLookup()`， `cacheInsert()` 拿到你的 cache 对象并使用它。

```xml
<cache name="myUserCache" class="solr.LRUCache"
    size="4096"
    initialSize="1024"
    autowarmCount="1024"
    regenerator="org.mycompany.mypackage.MyRegenerator" />
```

如果想要自动预热你的 cache，包含一个 `regenerator` 属性，值为 `solr.search.CacheRegenerator` 实现类的全路径名。也可以使用 `NoOpRegenerator`，用旧的项目简单填充 cache。在 `regenerator` 参数里定义： `regenerator="solr.NoOpRegenerator"`

## Query Sizing and Warming

### maxBooleanClauses

一个 boolean 查询最多有几个从句。可影响范围或前缀查询扩展的大量 boolean 查询。如果超过限制会抛出异常

```xml
<maxBooleanClauses>1024</maxBooleanClauses>
```

> 该选项对所有 core 生效。如果多个 `solrconfig.xml` 文件里该属性不一致，最后一个初始化的 core 的值被采用

### enableLazyFieldLoading

若设为 true 则非直接请求的字段会延迟加载。如果大多数一般查询只需要一个小的字段子集，尤其是很少访问的字段有大的尺寸时可提升性能

```xml
<enableLazyFieldLoading>true</enableLazyFieldLoading>
```

### useFilterForStortedQuery

配置 solr 使用过滤器来满足一个搜索。如果请求的排序不包含 "score"， `filterCache` 将用过滤器来匹配该查询。多数场景下，如果相同的搜索请求经常用不同的排序选项且都不用 "score" 才有用。

```xml
<useFilterForSortedQuery>true</useFilterForSortedQuery>
```

### QueryResultWindowSize

和 `queryResultCache` 一起使用，将 cache 超出请求的文档的 id。例如，如果一个搜索请求 文档 10 到 19，且 `QueryResultWindowSize` 为 50，文档 0 到 49 将被 cache。

```xml
<queryResultWindowSize>20</queryResultWindowSize>
```

### queryResultMaxDocsCached

这个参数设置 `queryResultCache` 里任意 key 缓存的文档最大数量

```xml
<queryResultMaxDocsCached>200</queryResultMaxDocsCached>
```

### useColdSearcher

若当前没有注册的 searcher，搜索请求应该等待一个新的 searcher 预热(false)还是立即执行(true)。当设为 false，请求会阻塞，直到 searcher 完成其 cache 的预热

```xml
<useColdSearcher>false</useColdSearcher>
```

### maxWarmingSearchers

任何时刻，后台正在预热的 searcher 最多可以有几个。超过该限制会抛出错误(error)。对只读的从机(slave)，2 是合理的。主机(master)应高一点点

```xml
<maxWarmingSearchers>2</maxWarmingSearchers>
```

## Query-Related Listeners 查询相关监听器

在 [cache](#cache) 章节已描述过，新的 Index Searcher 是缓存的。使用监听器触发执行查询相关的任务是可能的。最通用的是在定义查询推动 Index Searcher 启动时预热。这个做法其中一个好处是字段 cache 会预填充以加速排序。

良好的查询选择器(?)是这类监听器的关键。最好选择你最常用和/或最重的查询，并且不仅包括用到的关键字，还有任何其他参数诸如排序或过滤请求。

有 2 类事件可以触发监听器。一个 `firstSearch` 事件

```xml
<listener event="newSearcher" class="solr.QuerySenderListener">
  <arr name="queries">
  <!--
    <lst><str name="q">solr</str><str name="sort">price asc</str></lst>
    <lst><str name="q">rocks</str><str name="sort">weight asc</str></lst>
  -->
  </arr>
</listener>

<listener event="firstSearcher" class="solr.QuerySenderListener">
  <arr name="queries">
    <lst><str name="q">static firstSearcher warming in solrconfig.xml</str></lst>
  </arr>
</listener>
```

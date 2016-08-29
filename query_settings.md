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

`FastLRUCache` 

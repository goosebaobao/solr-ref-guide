# 建议

Solr 建议组件(SuggestComponet)为用户在查询词条时提供自动的建议。在你自己的应用程序使用它来实现强大的自动建议特性。

虽然使用语法检查(Spell checking)功能提供自动建议是可行的，但是 Solr 专门设计了 SuggestComponet 来实现该功能。这个方法利用了 Lucene 的建议实现且支持 Lucene 提供的所有查找实现

建议的主要特性

* 可插拔的查找实现
* 可插拔的词典，带来选择词典实现的灵活性
* 分布式支持

在 Solr 的 `techproducts` 示例里已在 `solrconfig.xml` 里配置了新的建议实现。了解更多的搜索组件，参考 `RequestHandlers and SearchComponents in SolrConfig`

覆盖了如下小节
* 在 `solrconfig.xml` 配置建议
 * 添加建议搜索组件
 * 添加建议请求处理器
* 使用例子
 * Get Suggestions with Weights
 * Multiple Dictionaries
 * Context Filtering
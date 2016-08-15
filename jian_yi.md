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

## 在 solrconfig.xml 里配置建议

`techproducts` 示例的 `solrconfig.xml` 已配置了 `suggest` 搜索组件和一个 `/suggest` 请求处理器。以此为你的配置基础，或按下面描述的从头创建配置

### 添加建议搜索组件

首先是在 `solrconfig.xml` 里添加一个搜索组件，并让它使用建议组件，如下

```xml
<searchComponent name="suggest" class="solr.SuggestComponent">
  <lst name="suggester">
  <str name="name">mySuggester</str>
  <str name="lookupImpl">FuzzyLookupFactory</str>
  <str name="dictionaryImpl">DocumentDictionaryFactory</str>
  <str name="field">cat</str>
  <str name="weightField">price</str>
  <str name="suggestAnalyzerFieldType">string</str>
  <str name="buildOnStartup">false</str>
  </lst>
</searchComponent>
```

#### 建议搜索组件参数

建议搜索组件有几个配置参数。查找实现(`lookupImpl`，如何在建议词典里找到词条)和词典实现(`dictionaryImpl`，如何在建议词典里保存词条)的选择基于几个必须的参数。不管是哪种查找或词典的实现，下面列出的是主要的参数

| 参数 | 说明 |
| -- | -- |
| searchComponentName | 搜索组件名称 |
| name | 建议的名称，可以在 URL 参数和 SearchHandler 配置里引用该名称。可以有多个建议(?) |
| lookupImpl | 查找实现。有几个可用的实现，参考 `Lookup Implementations`。如果未设置，默认为 JaspellLookupFactory |
| dictionaryImpl | 词典实现。有几个可用的实现，参考 `Dictionary Implementations`。如果未设置，默认的词典实现为 HighFrequencyDictionaryFactory，如果 `sourceLocation` 使用的话，则为 FileDictionaryFactory|
| field | 索引里的一个字段，用来作为建议词条的基础，如果 `sourceLocation` 为空(意味着 FileDictionaryFactory 之外的词典实现)那么这个字段里索引的词条被使用。<br>要作为建议的基础，这个字段必须是存储的。你也许想要用 copyField 来创建一个包含文档里其他字段的建议字段。无论如何，你希望该字段的分析最小化，所以，一个额外的选项是在你的 schema 创建一个只使用基本的分词器或过滤器的字段，如下面的一个字段<br><br>`<fieldType class="solr.TextField" name="textSuggest" positionIncrementGap="100">`<br>`<analyzer>`<br>`<tokenizer class="solr.StandardTokenizerFactory"/>`<br>`<filter class="solr.StandardFilterFactory"/>`<br>`<filter class="solr.LowerCaseFilterFactory"/>`<br>`</analyzer>`<br>`</fieldType>`<br><br>但是，如果你想要在词条上作更多的分析，如果使用 AnalyzingLookupFactory 作为查找实现，你就有了在索引和查询时分析的字段选项 |
| sourceLocation | 词典文件路径，用于 FileDictionaryFactory。如果该值为空，那么主索引将被用于词条和权重的来源 |
| storeDir | 保存词典文件的路径 |
| buildOnCommit 或 buildOnOptimize | 如果为 true，查找数据结构在软提交后会被重建。默认为 false，查找数据仅在 URL 参数 `suggest.build=true` 时重建。`buildOnCommit` 会在每次软提交时重建词典，`buildOnOptimize` 仅在索引优化时重建词典。某些查找实现重建非常耗时，尤其是对大索引，这种场景下，不推荐在有高频的软提交时使用 `buildOnCommit` 或 `buildOnOptimize`，替代的是，推荐低频的手工创建建议，使用 `suggest.build=true` 发送请求|
| buildOnStartup | |


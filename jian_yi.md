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
| buildOnCommit 或 buildOnOptimize | 如果为 true，查找数据结构在软提交后会被重建。默认为 false，查找数据仅在 URL 参数 `suggest.build=true` 时重建。`buildOnCommit` 会在每次软提交时重建词典，`buildOnOptimize` 仅在索引优化时重建词典。某些查找实现创建(建议)非常耗时，尤其是对大索引，这种场景下，不推荐在有高频的软提交时使用 `buildOnCommit` 或 `buildOnOptimize`，替代的是，推荐低频的手工创建建议，使用 `suggest.build=true` 发送请求 |
| buildOnStartup | true 表示在 Solr 启动时或 core 重载时创建查找数据结构。如果该参数未指定，会检查查找数据结构是否存在于磁盘，如果不存在则创建。设为 true 会导致 core 在加载(或重载)时更耗时，因为需要创建建议数据结构，而这是个耗时的操作。通常把这个设置设为 false，并手工创建建议，使用 `suggest.build=true` 发送请求|

#### 查找实现

##### AnalyzingLookupFactory

##### FuzzyLookupFactory

##### AnalyzingInfixLookupFactory

##### BlendedInfixLookupFactory

##### FreeTextLookupFactory

##### FSTLookupFactory

##### TSTLookupFactory

A simple compact ternary trie based lookup.

##### WFSTLookupFactory

##### JaspellLookupFactory

#### 词典实现

##### DocumentDictionaryFactory

##### DocumentExpressionDictionaryFactory

##### HighFrequencyDictionaryFactory

##### FileDictionaryFactory

#### 多重词典

### 添加建议请求处理器(Suggest Request Handler)

添加搜索组件以后，必须在 `solrconfig.xml` 里添加请求处理器(request handler)。这个请求处理器和其他的请求处理器一样，允许你配置默认参数来为建议请求服务。这个请求处理器的定义里必须包含之前定义的建议搜索组件。

```xml
<requestHandler name="/suggest" class="solr.SearchHandler" startup="lazy">
  <lst name="defaults">
    <str name="suggest">true</str>
    <str name="suggest.count">10</str>
  </lst>
  <arr name="components">
    <str>suggest</str>
  </arr>
</requestHandler>
```

#### 建议请求处理器参数

| 参数 | 说明 |
| -- | -- |
| suggest=true | 该参数必须为 true |
| suggest.dictionary | 在搜索组件里配置的词典组件的名称。该参数是必须滴，可以在请求处理器里设置，也可以在查询时通过参数传递 |
| suggest.q | 查找建议时用的查询 |
| suggest.count | 指定 Solr 返回的建议数量 |
| suggest.cfq | cfq = Context Filter Query，如果支持的话，用于过滤建议的上下文字段 |
| suggest.build | true 表示构建建议索引。仅在初次请求时有用；你应该不会想在每次请求时构建词典，尤其是在生产环境。如果想要保持词典最新，应该在搜索组件上使用 `buildOnCommit` 或 `buildOnOptimize` 参数 |
| suggest.reload | true 表示重载建议索引 |
| suggest.buildAll | true 表示构建所有的建议索引|
| suggest.reloadAll | true 表示重载所有的建议索引 |

> **上下文过滤**
>
> 上下文过滤(suggest.cfg)当前仅 AnalyzingInfixLookupFactory 和 BlendedInfixLookupFactory 支持，且仅当 Document*Dictionary。其他实现将忽视过滤请求，返回未过滤的匹配。




## 用法示例

### 根据权重获取建议

### 多重词典

### 内容过滤




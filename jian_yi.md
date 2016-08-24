# 建议

Solr 建议组件(SuggestComponet)为用户在查询词条时提供自动的建议。在你自己的应用程序使用它来实现强大的自动建议特性。

虽然使用语法检查(Spell checking)功能提供自动建议是可行的，但是 Solr 专门设计了 SuggestComponet 来实现该功能。这个方法利用了 Lucene 的建议实现且支持 Lucene 提供的所有查找实现

建议的主要特性

* 可插拔的查找实现
* 可插拔的词典，带来选择词典实现的灵活性
* 分布式支持

在 Solr 的 `techproducts` 示例里已在 `solrconfig.xml` 里配置了新的建议实现。了解更多的搜索组件，参考 `RequestHandlers and SearchComponents in SolrConfig`

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

`lookupImpl` 参数定义了在建议索引里查找词条的算法。有几种实现可供选择，有些还需要额外的配置参数

##### AnalyzingLookupFactory

该实现首先分析输入文本，然后将分析过的结果添加到一个加权的 FST(?)，在查找时同样这么做。

这个实现使用下面列出的额外的属性

* suggestAnalyzerFieldType: 在构建和查询时分析建议所用的字段类型
* exactMatchFirst: 默认=true，首先返回准确的建议，即便其前缀或其他字符串在 FST 里有更高的权重。(这个应该是表示优先返回完全匹配的建议)
* preserveSep: 默认=true，保留词元之间的分隔符。这表示对分词敏感(例如，baseball 和 base ball 是不同的)
* preservePositionIncrements: 如果为 true，建议会保留位置增量。这表示构建建议时保留位置间隔的分词过滤器更受尊重(?)。默认=false

##### FuzzyLookupFactory

AnalyzingSuggester 的扩展，但是模糊性。Levenshtein 算法用于衡量相似性。

该实现使用下面的额外属性

* exactMatchFirst: 默认=true，首先返回准确的建议，即便其前缀或其他字符串在 FST 里有更高的权重。(这个应该是表示优先返回完全匹配的建议)
* preserveSep: 默认=true，保留词元之间的分隔符。这表示对分词敏感(例如，baseball 和 base ball 是不同的)
* maxSurfaceFormsPerAnalyzedForm: Maximum number of surface forms to keep for a single analyzed form. When there are too many surface forms we discard the lowest weighted ones.
* maxGraphExpansions: When building the FST ("index-time"), we add each path through the tokenstream graph as an individual entry. This places an upper-bound on how many expansions will be added for a single suggestion. The default is -1 which means there is no limit.
* preservePositionIncrements: 如果为 true，建议会保留位置增量。这表示构建建议时保留位置间隔的分词过滤器更受尊重(?)。默认=false
* maxEdits: 可编辑的字符串最大数值。系统最大限制为 2，默认为 1
* transpositions: If true, the default, transpositions should be treated as a primitive edit operation.
* nonFuzzyPrefix: The length of the common non fuzzy prefix match which must match a suggestion. The default is 1.
* minFuzzyLength: The minimum length of query before which any string edits will be allowed. The default is 3.
* unicodeAware: 默认=false，如果为 true，则 maxEdits, minFuzzyLength, transpositions 和 nonFuzzyPrefix 参数以 unicode code 字符来计算而不是字节

##### AnalyzingInfixLookupFactory

##### BlendedInfixLookupFactory

##### FreeTextLookupFactory

##### FSTLookupFactory

##### TSTLookupFactory

简单紧凑的三叉树(Ternary Trie)的查找

##### WFSTLookupFactory

##### JaspellLookupFactory

来自 JsSpell 项目，基于三叉树但更加复杂的查找。如果想要更加复杂的匹配结果使用这个实现

#### 词典实现

词典实现定义了词条如何存储。有多个选项，如果有必要的话，单个请求可以使用多个词典。

##### DocumentDictionaryFactory

词典包含：词条，权重和可选的来自索引的负荷(payload)

参数

* weightField: 一个存储的字段，或DocValue 数值字段。该字段可选
* payloadField: 一个存储的字段，可选的
* contextField: 上下文过滤字段，注意只有某些查找实现支持过滤

##### DocumentExpressionDictionaryFactory

与 DocumentDictionaryFactory 一样，但是允许指定一个任意的表达式。

参数

* weightExpression: 用于对建议评分的任意表达式，必须使用数值字段。该字段是必须滴
* payloadField: 一个存储的字段，可选的
* contextField: 上下文过滤字段，注意只有某些查找实现支持过滤

##### HighFrequencyDictionaryFactory

##### FileDictionaryFactory

#### 多重词典

单个建议组件里可以包含多个词典实现。只需简单的分别定义建议，示例如下

```xml
<searchComponent name="suggest" class="solr.SuggestComponent">
  <lst name="suggester">
    <str name="name">mySuggester</str>
    <str name="lookupImpl">FuzzyLookupFactory</str>
    <str name="dictionaryImpl">DocumentDictionaryFactory</str>
    <str name="field">cat</str>
    <str name="weightField">price</str>
    <str name="suggestAnalyzerFieldType">string</str>
  </lst>
  <lst name="suggester">
    <str name="name">altSuggester</str>
    <str name="dictionaryImpl">DocumentExpressionDictionaryFactory</str>
    <str name="lookupImpl">FuzzyLookupFactory</str>
    <str name="field">product_name</str>
    <str name="weightExpression">((price * 2) + ln(popularity))</str>
    <str name="sortField">weight</str>
    <str name="sortField">price</str>
    <str name="storeDir">suggest_fuzzy_doc_expr_dict</str>
    <str name="suggestAnalyzerFieldType">text_en</str>
  </lst>
</searchComponent>
```

在查询里使用建议时，可以在请求里定义多个 'suggest.dictionary' 参数，引用搜索组件里定义的每个建议的名称即可。响应里会包含每个建议的小节。

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
> 上下文过滤(suggest.cfg)当前仅 AnalyzingInfixLookupFactory 和 BlendedInfixLookupFactory 支持，且仅当词典实现为 Document*Dictionary。其他实现将忽视过滤请求，返回未过滤的匹配。

## 用法示例

### 根据权重获取建议

这是最基本的建议，使用单个词典和单个 Slor core，示例

```
http://localhost:8983/solr/techproducts/suggest?suggest=true&suggest.build=true&suggest.dictionary=mySuggester&wt=json&suggest.q=elec
```

在这个例子里，有一个简单的请求：,suggest.q 参数指定字符串 'elec'，suggest.build 参数指定构建建议词典(注意，你不会想要每次请求都构建建议词典，取而代之的是使用 buildOnCommit 或 buildOnOptimize，如果你定期改变文档)

示例的响应

```json
{
  "responseHeader": {
    "status": 0,
    "QTime": 35
  },
  "command": "build",
  "suggest": {
    "mySuggester": {
      "elec": {
        "numFound": 3,
        "suggestions": [
          {
            "term": "electronics and computer1",
            "weight": 2199,
            "payload": ""
          },
          {
            "term": "electronics",
            "weight": 649,
            "payload": ""
          },
          {
            "term": "electronics and stuff2",
            "weight": 279,
            "payload": ""
          }
        ]
      }
    }
  }
}
```

### 多重词典

如果定义了多个词典，可以在查询里使用它们。示例查询

```
http://localhost:8983/solr/techproducts/suggest?suggest=true&suggest.dictionary=mySuggester&suggest.dictionary=altSuggester&wt=json&suggest.q=elec
```

在这个例子里，发送了 suggest.q 参数为字符串 'elec' 和 2 个 suggest.dictionary 

示例的响应

```json
{
  "responseHeader": {
    "status": 0,
    "QTime": 3
  },
  "suggest": {
    "mySuggester": {
      "elec": {
        "numFound": 1,
        "suggestions": [
          {
            "term": "electronics and computer1",
            "weight": 100,
            "payload": ""
          }
        ]
      }
    },
    "altSuggester": {
      "elec": {
        "numFound": 1,
        "suggestions": [
          {
            "term": "electronics and computer1",
            "weight": 10,
            "payload": ""
          }
        ]
      }
    }
  }
}
```

### 内容过滤

上下文过滤让你使用分离的上下文字段来过滤建议，例如分类，部门或其他。AnalyzingInfixLookupFactory 和 BlendedInfixLookupFactory 当前支持这个特性，当后台是用的 DocumentDictionaryFactory。

在你的建议配置里添加 `contextField`。这个示例将建议名称且按分类过滤

```xml
<!--solrconfig.xml-->

<searchComponent name="suggest" class="solr.SuggestComponent">
  <lst name="suggester">
    <str name="name">mySuggester</str>
    <str name="lookupImpl">AnalyzingInfixLookupFactory</str>
    <str name="dictionaryImpl">DocumentDictionaryFactory</str>
    <str name="field">name</str>
    <str name="weightField">price</str>
    <str name="contextField">cat</str>
    <str name="suggestAnalyzerFieldType">string</str>
    <str name="buildOnStartup">false</str>
  </lst>
</searchComponent>
```

示例的内容过滤建议查询

```cmd
http://localhost:8983/solr/techproducts/suggest?suggest=true&suggest.build=true& \ 
suggest.dictionary=mySuggester&wt=json&suggest.q=c&suggest.cfq=memory
```

该建议将只返回产品标记为 cat=memory 的建议




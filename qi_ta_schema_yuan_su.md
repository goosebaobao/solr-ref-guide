# 其他 schema 元素

## Unique Key 唯一键

`uniqueKey` 元素指示文档的唯一标识是哪个字段。虽然 `uniqueKey` 不是必须的，但几乎是你的应用程序设计中总是要保证的。例如，如果想要更新索引里的文档应该用到 `uniqueKey` 

定义

```xml
<uniqueKey>id</uniqueKey>
```

`copyField` 不能用于 `uniqueKey` 字段。也不能用 `UUIDUpdateProcessorFactory` 自动生成 `uniqueKey` 值

如果字段是多值的或继承自多值字段类型，用作 `uniqueKey` 字段会导致操作失败。但是，`uniqueKey` 会继续工作，只要该字段被正确的使用。

## 默认的搜索字段和查询操作

虽然已不建议使用并会在某个时候弃用，solr 仍然支持在 schema 配置 `<defaultSearchField/>`(被 `df` 参数代替) 和 `solrQueryParserDefaultOperator="OR"/>`(被  `q.op` 参数代替)

如果你在 schema 设定了这些选项，强烈推荐用请求参数(或默认请求参数)替换它们，因为未来的 solr 发行版本可能会移除。

## Similarity 相似性

Similarity 是搜索时给文档评分的 Lucene 类。

每个 collection 都有一个全局的 similarity，默认情况下，solr 使用自带的 `SchemaSimilarityFactory`，允许特定字段类型配置其特定的 Similarity，并为所有未明确指定 Similarity 的字段类型使用 `BM25Similarity`。

在 `schema.xml` 里，任何字段类型的定义之外，声明一个顶级的 `<similarity/>` 元素，可覆盖默认行为。这个 similarity 声明要么直接引用一个有无参构造函数的类，例如下面的例子

```xml
<similarity class="solr.BM25Similarity"/>
```

要么引用一个 `SimilarityFactory` 实现，接受可选的初始化参数

```xml
<similarity class="solr.DFRSimilarityFactory">
  <str name="basicModel">P</str>
  <str name="afterEffect">L</str>
  <str name="normalization">H2</str>
  <float name="c">7</float>
</similarity>
```

大多数场景下，如果你的 `schema.xml` 包含了字段类型里的 `<similarity/>` 声明，如上面那样指定全局的 similarity 会导致错误。一个关键的异常就是你可能明确声明了 `SchemaSimilarityFactory`，并设定所有字段类型的默认行为，但是这个默认行为没有声明为一个明确的 similarity，即已配置了 similarity 的字段类型的名字(用 `defaultSimFromFieldType` 设定)(<font color='red'>这段话很绕，意思就是，如果你已经在字段类型里配置了 similarity，要想配置一个全局的 similarity，很可能会出错。这是因为你的全局 similarity 是用的 `SchemaSimilarityFactory`，但是你没有配置这个 factory 的 `defaultSimFromFieldType` 为那些个已配置了 similarity 字段的其中之一的名字</font>)

```xml
<similarity class="solr.SchemaSimilarityFactory">
  <str name="defaultSimFromFieldType">text_dfr</str>
<similarity>
<fieldType name="text_dfr" class="solr.TextField">
  <analyzer ... />
  <similarity class="solr.DFRSimilarityFactory">
    <str name="basicModel">I(F)</str>
    <str name="afterEffect">B</str>
    <str name="normalization">H3</str>
    <float name="mu">900</float>
  </similarity>
</fieldType>
<fieldType name="text_ib">
  <analyzer ... />
  <similarity class="solr.IBSimilarityFactory">
    <str name="distribution">SPL</str>
    <str name="lambda">DF</str>
    <str name="normalization">H2</str>
  </similarity>
</fieldType>
<fieldType name="text_other">
  <analyzer ... />
</fieldType>
```

上面例子里，`IBSimilarityFactory`(使用基于信息的模式)将被用于类型为 `text_ib` 的所有字段，同时 `DFRSimilarityFactory`(随机偏离)将既用于类型为 `ext_dfr` 的所有字段，又用于任何未在字段类型里设定 `<similarity/>` 的字段(<font color='red'>即，字段类型为 `text_other` 的字段，和字段类型为 `text_+dfr` 的字段一样使用 `IBSimilarityFactory`</font>)

如果 `SchemaSimilarityFactory` 明确声明，且没有配置 `defaultSimFromFieldType`，那么 `BM25Similarity` 会用作默认值。

除了这里提到的几种 factory 以外，还有一些其他 Similarity 实现，诸如 `SweetSpotSimilarityFactory`， `ClassicSimilarityFactory` 等等，可以使用，详细信息，参阅 solr 的 [similarity factory](lucene.apache.org/solr/6_0_0/solr-core/org/apache/solr/schema/SimilarityFactory.html) java 文档 
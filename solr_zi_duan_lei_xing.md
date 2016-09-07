# solr 字段类型

字段类型定义了 solr 如何解释字段的数据及字段如何被查询。solr 默认包含了很多字段类型，而且也可以自定义。

## 定义和属性

字段类型的定义可包含 4 类信息

* 字段名(必须) 
* 实现类名(必须)
* 如果字段类型是 `TextField`，字段解析的说明
* 字段类型属性 - 依赖于实现类，有些属性可能是必须的

### schema.xml 里的字段定义

字段类型在 `schema.xml` 里定义。每个字段类型定义在 `fieldType` 元素之间。它们可以用 `types` 元素集中。下面是一个字段类型 `text_general` 定义的例子

```xml
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <!-- in this example, we will only use synonyms at query time
    <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
    -->
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" 
      ignoreCase="true" expand="true"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

上面例子里的第一行包含了字段类型的名字，`text_general`，以及实现类的名字，`solr.TextField`。其余的定义是关于字段解析，参考 ` Understanding Analyzers, Tokenizers, and Filters`

实现类确保字段被正确处理。在 `schema.xml` 里的类名，字符串 `solr` 是 ` org.apache.solr.schema` 或 `org.apache.solr.analysis` 的简写。因此，`solr.TextField` 实际是 `org.apache.solr.schema.TextField`。

### 字段类型属性

字段类型的 `class` 决定了字段类型大多数行为，但是仍然可以定义可选的属性。例如，下面的日期字段类型定义了 2 个属性，`sortMissingLast` 和 `omitNorms`

```xml
<fieldType name="date" class="solr.TrieDateField" sortMissingLast="true" omitNorms="true"/>
```

字段类型分为 3 个大类

* 字段类型的类
* 通用属性 所有字段类型都支持
* 字段默认属性 在字段类型上指定后，可以被字段继承替换字段默认行为的属性(<font color='red'>存疑：如果字段上定义的属性和字段类型不同，应该是以字段上定义的为准</font>)

#### 通用属性

| 属性 | 描述 | 值 |
| -- | -- | -- |
| name | 字段类型名字。这个值在字段定义时用于 "type" 属性。强烈建议名字以字母或下划线而非数字开头。目前并非强制要求 | |
| class | 存储和索引该类型数据的类。注意，类名可以用 "solr" 前缀，solr 会自动识别该类在哪个包 - 所以 "solr.TextField" 有效。如果使用第三方的类，使用完整的类名。"solr.TextField" 等效于 "org.apache.solr.schema.TextField"" | |
| positionIncrementGap | 对于多值字段，指定多值之间的距离，这能避免"虚假"的短语匹配 | integer |
| autoGeneratePhraseQueries | 用于文本字段。如果为 true，solr 自动对相邻的词条生成短语查询。如为 false，只有用双引号包围的词条才会作为短语处理 | true 或 false |
| docValuesFormat | 定义一个自定义 `DocValuesFormat` 的类型字段 ，需要一个 schema 感知的编解码器，诸如 `SchemaCodecFactory` 已在 `solrconfig.xml` 里配置好 | n/a |
| postingsFormat | 定义一个自定义 `PostingsFormat` 的类型字段 ，需要一个 schema 感知的编解码器，诸如 `SchemaCodecFactory` 已在 `solrconfig.xml` 里配置好 | n/a |

> Lucene 索引向后兼容只支持默认的编解码器。如果在 schema.xml 里自定义 `postingsFormat` 或 `docValuesFormat`，升级到新版本的 solr 之前可能要求你切换到默认的编解码器并优化索引以重新写入默认编解码器，或者在升级之后从零开始重建索引。

#### 字段默认属性

这些属性要么在字段类型指定，要么在特定字段上覆盖字段类型的指定。每个属性的默认值依赖于对应的 `FieldType` 类，取决于 `<schema/>` 的 `version` 属性值。下面的表格包含了 solr 提供的大多数 `FieldType` 实现的默认值，如果 `schema.xml` 的 `version="1.6"`

| 属性 | 描述 | 值 | 隐含默认值 |
| -- | -- | -- | -- |
| indexed | 如为 true，字段值能用于查询检索匹配的文档 | true/false | true |
| stored | 如为 true，字段值可以被查询获取 | true/false | true |
| docValues | 如为 true，字段值将放入面向列的 `DocValues` 结构 | true/false | false |
| sortMissingFirst<br>sortMissingLast | 当一个索引字段未提供时，控制文档(在索引)的位置(是在最前/最后) | true/false | false | 
| multiValued | 如为 true，单个文档在这个字段可能包含多个值 | true/false | false |
| omitNorms | 如为 true，省略与此字段关联的加权基准(这会关闭长度标准化，及索引时对该字段的加权，从而节省内存占用)。对所有原始(无需解析)的字段类型，诸如 int，float，date，bool，和 string，默认为 true。只有全文本字段或需要索引时加权的字段才需要加权基准(norms) | true/false | \* |
| omitTermFreqAndPositions | 如为 true，在提交字段时省略词条的频率/次数，位置，载荷。对于不需要这些信息的字段能提升性能，同时还减少了索引需要的存储空间。依赖位置信息的查询会无声的失败。对于所有非文本字段这个属性默认为 true | true/false | \* |
| omitPositions | 和 `omitTermFreqAndPositions` 类似，但是会保留词条频率/次数信息 | true/false | \* |
| termVectors<br>termPositions<br>termOffsets<br>termPayloads | 这些选项命令 solr 保持每个文档完整的词条向量，随意的包括在那些向量里每个词条的位置，偏移，及载荷信息。这些可用于加快高亮和其他辅助功能，但会极大增加索引的尺寸。在 solr 的典型应用里是没有必要的 | true/false | false |
| required | 如为 true，solr 拒绝添加一个该字段没有值的文档 | true/false | false |
| useDocValuesAsStored | 如果该字段开启了 `docValues`，设为 true 将允许该字段在 `fl=*` 时像一个存储的字段一样返回(即使该字段的 `stored=false`) | true/false | true |

> term 的 `Vector` 实际上就是由 term 的 `positions`， `offset`， `payloads` 组成的

### 字段类型相似性

字段类型可以指定 `<similarity/>` ，可用于当给一个有该字段类型字段的文档评分时，只要 collection 的 "global" 相似性允许。默认情况，所有没有定义 similarity 的字段类型使用 `BM25Similarity`。要了解更多细节，及配置 global 和 每个类型的 similarity，参考 `Other Schema Elements`

## solr 自带字段类型

下面表格列出了 solr 可用的字段类型。`org.apache.solr.schema` 包包含列出的所有类

| 类 | 描述 |
| -- | -- |
| BinaryField | 二进制数据 |
| BoolField | 包含 true 或 false。"1"， "t"， "T" 为首字母视同 true，其他首字母的视同 false |
| CollationField | 索引和范围查询支持 Unicode 排序规则。如果使用 ICU4J，ICUCollationField 是个更好的选择。参考 `Unicode Collation` |
| CurrencyField | 支持金额和汇率 |
| DateRangeField | 支持日期范围索引，也包括时间点 |
| ExternalFileField | 从磁盘文件里拉取数值 |
| EnumField | 对于那些不容易以字母或数字顺序来保存的(例如一个严重程度的列表)，可定义一个值的枚举集。这个字段类型需要一个配置文件，用来有序的记录字段值 |
| ICUCollationField |  索引和范围查询支持 Unicode 排序规则。参考 `Unicode Collation` |
| LatLonType | 相联系的纬度，经度对，纬度在前 |
| PointType | 任意维度的点，在蓝图或CAD绘图等资源里搜索时有用 |
| PreAnalyzedField | 提供了一种向 solr 发送连续的词元流的方法，词元流可附带独立的存储字段，并将此信息存储和索引而不做任何额外的处理 |
| RandomSortField | 该字段类型不包含值(?)查询如果在这个字段上做排序将随机排列。使用动态字段来使用这个特性 |
| SpatialRecursivePrefixTreeFieldType | (简写为 RPT) 维度+逗号+经度字符串或其他的 WKT 格式 |
| StrField | 字符串(UTF8 编码或 Unicode) |
| TextField | 文本，通常是多个单词或词元 |
| TrieDateField | 日期<br>`precisionStep="0"` 可按日期排序，索引最小<br>`precisionStep="8"` 默认值，可范围查询 |
| TrieDoubleField | double，8字节<br>precisionStep="0" 可按数字排序，索引最小<br>precisionStep="8" 默认值，可范围查询 |
| TrieField | 这个类型必须指定 type 属性，可选的值为 [integer, long, float, double, date]。等效于对应的 Trie 字段<br>precisionStep="0" 可按数字排序，索引最小<br>precisionStep="8" 默认值，可范围查询 |
| TrieFloatField | float，4字节<br>precisionStep="0" 可按数字排序，索引最小<br>precisionStep="8" 默认值，可范围查询 |
| TrieIntField | int，4字节<br>precisionStep="0" 可按数字排序，索引最小<br>precisionStep="8" 默认值，可范围查询 |
| TrieLongField | long，8字节<br>precisionStep="0" 可按数字排序，索引最小<br>precisionStep="8" 默认值，可范围查询 |
| UUIDField | 通用唯一标识符，传入“NEW”，solr会创建一个 UUID，cloud 模式不建议这样使用 |

## 金额和汇率

## 日期

## 枚举字段

## 外部文件及处理

## 字段属性用例
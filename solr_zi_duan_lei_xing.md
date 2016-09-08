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
| CurrencyField | 支持货币和汇率 |
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
| TrieDoubleField | double，8字节<br>`precisionStep="0"` 可按数字排序，索引最小<br>`precisionStep="8"` 默认值，可范围查询 |
| TrieField | 这个类型必须指定 type 属性，可选的值为 `integer`, `long`, `float`, `double`, `date`。等效于对应的 Trie 字段<br>`precisionStep="0"` 可按数字排序，索引最小<br>`precisionStep="8"` 默认值，可范围查询 |
| TrieFloatField | float，4字节<br>`precisionStep="0"` 可按数字排序，索引最小<br>`precisionStep="8"` 默认值，可范围查询 |
| TrieIntField | int，4字节<br>`precisionStep="0"` 可按数字排序，索引最小<br>`precisionStep="8"` 默认值，可范围查询 |
| TrieLongField | long，8字节<br>`precisionStep="0"` 可按数字排序，索引最小<br>`precisionStep="8"` 默认值，可范围查询 |
| UUIDField | 通用唯一标识符，传入 "NEW"，solr 会创建一个 UUID。注意：在 SolrCloud 模式，对于大多数用户配置一个 UUIDField 默认值为 "NEW" 是不明智的(而且，如果 UUID 值配置为唯一键，也不可能)，因为这会导致每个 replica 的每个 document 都有一个唯一的 UUID 值。建议用 UUIDUpdateProcessorFactory 来生成 UUID 值|

## 货币和汇率

`Currency` 字段类型支持以下特性

* 点查询
* 范围查询
* 函数范围查询
* 排序
* 使用货币代码或者符号解析货币
* 对称和非对称汇率

### 货币配置

`Currency` 字段类型在 `schema.xml` 定义，默认配置如下

```xml
<fieldType name="currency" class="solr.CurrencyField" precisionStep="8"
    defaultCurrency="USD" currencyConfig="currency.xml" />
```

这个例子里，我们定义了字段类型的名字和类，定义了 `defaultCurrency` 为 "USD"，即美元。还定义了 `currencyConfig`，使用名为 `currency.xml` 的文件。这个是默认货币到其他货币的汇率文件。还有一个代替的实现允许下载货币数据

索引期间，金额字段能被索引为本地货币。例如，一个欧洲电子商务网站上的商品，索引其价格字段为 "100,EUR" 是很合理的。价格和货币之间用逗号分隔，价格应该被编码为一个浮点值。

查询时，范围和点查询都可以支持。

### 汇率

通过指定提供者来配置汇率。Solr 自带 2 类提供者：`FileExchangeRateProvider` 或 `OpenExchangeRatesOrgProvider`

#### FileExchangeRateProvider

这个提供者需要你提供一个汇率文件。这是默认的，意思是要用这个提供者你只需要指定文件的路径和名字到 `currencyConfig` 

这里有一个 solr 自带的示例 `currency.xml` 文件，和 `shema.xml` 文件在同一个目录，下面是该文件的一小段

```xml
<currencyConfig version="1.0">
  <rates>
    <!-- Updated from http://www.exchangerate.com/ at 2011-09-27 -->
    <rate from="USD" to="ARS" rate="4.333871" comment="ARGENTINA Peso" />
    <rate from="USD" to="AUD" rate="1.025768" comment="AUSTRALIA Dollar" />
    <rate from="USD" to="EUR" rate="0.743676" comment="European Euro" />
    <rate from="USD" to="CAD" rate="1.030815" comment="CANADA Dollar" />

    <!-- Cross-rates for some common currencies -->
    <rate from="EUR" to="GBP" rate="0.869914" />
    <rate from="EUR" to="NOK" rate="7.800095" />
    <rate from="GBP" to="NOK" rate="8.966508" />
    
    <!-- Asymmetrical rates -->
    <rate from="EUR" to="USD" rate="0.5" />
  </rates>
</currencyConfig>
```

#### OpenExchangeRatesOrgProvider

你可以配置 solr 从 [OpenExchangeRates.org](https://openexchangerates.org/) 下载汇率，可以每小时更新美元和 170 种货币的汇率。

这个案例里，你需要指定 `providerClass`，并注册以获得一个 API key，如下所示

```xml
<fieldType 
  name="currency" 
  class="solr.CurrencyField" 
  precisionStep="8" 
  providerClass="solr.OpenExchangeRatesOrgProvider" 
  refreshInterval="60" 
  ratesFileLocation="http://www.openexchangerates.org/api/latest.json?app_id=yourPersonalAppIdKey"/>
```

`refreshInterval` 以分钟为单位，所以上面例子会每隔 60 分钟下载最新的汇率。这个刷新间隔可以加大，但不能减少。

## 日期

### 日期格式化

solr 的 日期字段 (`TrieDateField` 和 `DateRangeField`) 表现为一个毫秒精度的时间点。其所用格式是 [XML Schema specification](https://www.w3.org/TR/xmlschema-2/#dateTime)([ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 的一个有限子集)的 `dateTime` 的权威表述的有限格式(<font color='red'>这一段大概意思就是并不支持各种日期格式，只支持有限的一种或几种</font>)。如果熟悉 Java 8，solr 使用 [DateTimeFormatter.ISO_INSTANT](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html#ISO_INSTANT) 来格式化和解析。

```
YYYY-MM-DDThh:mm:ssZ
```

* `YYYY` 年
* `MM` 月
* `DD` 月份里的日期
* `hh` 24小时制的小时
* `mm` 分钟
* `ss` 秒
* `Z` 就是字符 'Z'，表示这个字符串用 UTC 表示日期

注意，不能指定时区，表示时间的字符串总是以 UTC 来表示，示例如下

```
1972-05-20T17:33:18Z
```

如果你愿意，还可以包含秒的小数，但是任何超出毫秒的精度都会被忽略。示例如下

* `1972-05-20T17:33:18.772Z`
* `1972-05-20T17:33:18.77Z`
* `1972-05-20T17:33:18.7Z`

在 0000 年之前的日期，必须以 `'-'` 开头，`'+'` 开头表示 9999 年以后。0000 年 视为 公元前 1 年(year 1 BC)，没有 公元前/后 0 年这种事。

> **查询时可能要转义**

> 如你所见，日期格式包含了冒号来分隔时，分和秒。由于对于 solr 大多数的查询解析器来说，冒号是个特殊字符，所以有时候需要转义

> 这是个无效查询
 
> `datefield:1972-05-20T17:33:18.772Z`

> 这些是有效查询
 
> `datefield:1972-05-20T17\:33\:18.772Z`
 
> `datefield:"1972-05-20T17:33:18.772Z"`
 
> `datefield:[1972-05-20T17:33:18.772 TO *]`

#### 日期范围格式化

solr 的 `DateRangeField` 支持前面所述的时刻语法(和下面要讲的数学日期一起)和日期范围表示。一个例子是截断日期，表示整个日期跨度。其他例子是范围语法(`[ TO ]`)，这里有些例子

* `2000-11` – 2000 年 整个 11 月
* `2000-11T13` – 同上，不过是当日 13 时(下午 1-2 时)
* `-0009` – 公元前 10 年。0 年，是公元 0 年，也是公元前 1 年
* `[2000-11-01 TO 2014-12-01]` – 不解释(<font color='red'>霸气~</font>)
* `[2014 TO 2014-12-01]` – 从 2014 到 2014-12-01
* `[* TO 2014-12-01]` – 从可表示的最早时间到 2014-12-01

局限：范围语法不支持内嵌数学日期。如果你指定一个用数学日期截断的 `TrieDateField` 日期实例，如 `NOW/DAY`，你得到的依然是当天的第一毫秒，而不是整天的范围。开区间范围(使用 `{` 和 `}`)在查询时可用，但不能用来做范围索引。

### 数学日期

solr 日期字段类型也支持*数学日期*表达式，这使得创建一个相对于某确定时刻的时间更容易了，包括用特殊的值 "`NOW`" 表示当前时间。

#### 数学日期语法

数学日期表达式不是以特定单位增加时间，就是以特定单位对当前时间取整。表达式可以是链式的，从左至右执行

例如：从现在到 2 个月后的时刻

```
NOW+2MONTHS
```

一天以前

```
NOW-1DAY
```

斜杠表示取整，下面表示当前小时的开始

```
NOW/HOUR
```

下面例子计算(毫秒精度) 6 个月 3 天后并按天取整，即那天开始的时刻

```
NOW+6MONTHS+3DAYS/DAY
```

注意，数学日期最常和 NOW 一起用，与此同时，也可以和任何确定的时刻一起

```
1972-05-20T17:33:18.772Z+6MONTHS+3DAYS/DAY
```

#### 影响数学日期所需要的参数

##### NOW

在一个分布式请求里，solr 内部使用 `NOW` 参数来确保数学日期表达式跨越多个节点的一致性。但是，它也可以指定为命令 solr 使用一个任意的时刻(过去或未来的)来覆盖，为了所有场景，那里特殊值 "`NOW`" 会加入数学日期表达式。

##### TZ

### 更多 DataRangeField 细节

## 枚举字段

## 外部文件及处理

## 字段属性用例
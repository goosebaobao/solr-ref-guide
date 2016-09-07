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
| -- | -- |
| name | 字段类型名字。这个值在字段定义时用于 "type" 属性。强烈建议名字以字母或下划线而非数字开头。目前并非强制要求 | |
| class | 存储和索引该类型数据的类。注意，类名可以用 "solr" 前缀，solr 会自动识别该类在哪个包 - 所以 "solr.TextField" 有效。如果使用第三方的类，使用完整的类名。"solr.TextField" 等效于 "org.apache.solr.schema.TextField"" | |
| positionIncrementGap | 对于多值字段，指定多值之间的距离，这能避免"虚假"的短语匹配 | integer |
| autoGeneratePhraseQueries | 用于文本字段。如果为 true，solr 自动对相邻的词条生成短语查询。如为 false，只有用双引号包围的词条才会作为短语处理 | true 或 false |
| docValuesFormat | 定义一个自定义 `DocValuesFormat` 的类型字段 ，需要一个 schema 感知的编解码器，诸如 `SchemaCodecFactory` 已在 `solrconfig.xml` 里配置好 | n/a |
| postingsFormat | 定义一个自定义 `PostingsFormat` 的类型字段 ，需要一个 schema 感知的编解码器，诸如 `SchemaCodecFactory` 已在 `solrconfig.xml` 里配置好 | n/a |

> Lucene 索引向后兼容只支持默认的编解码器。如果在 schema.xml 里自定义 `postingsFormat` 或 `docValuesFormat`，升级到新版本的 solr 之前可能要求你切换到默认的编解码器并优化索引以重新写入默认编解码器，或者在升级之后从零开始重建索引。

#### 字段默认属性



## solr 自带字段类型

## 金额和汇率

## 日期

## 枚举字段

## 外部文件及处理

## 字段属性用例
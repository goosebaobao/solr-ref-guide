# 定义字段

字段在 `schema.xml` 的 `field` 元素里定义。定义好字段类型后，定义字段很简单。

## 例子

下面例子定义了一个名为 `price`，类型名为 `float`，默认值为 `0.0` 的字段，`indexed` 和 `stored` 属性明确设定为 `true`，同时，其他的属性都继承自 `float` 字段类型

```xml
<field name="price" type="float" default="0.0" indexed="true" stored="true"/>
```

## 字段属性

| 属性 | 描述 |
| name | 字段名。应该是字母数字下划线组成，不能以数字开头。名字前后都是下划线的(如 `_version_`)为保留字。每个字段都要有名字 |
| type | 这个字段的类型名。是在 `fieldType` 里定义的 `name`。每个字段都要有类型名 |
| default | 索引时，如果该字段没有值，一个默认值会自动添加到文档里的该字段。如果未指定则没有默认值 |

## 可选的字段类型覆盖属性

字段可以有和很多字段类型一样的属性。下表所列属性如在字段里设定，将覆盖其在字段类型 `<fieldType/>` 里明确设定的值或默认值。

| 属性 | 描述 | 值 | 默认值 |
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

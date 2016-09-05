# Codec Factory 编解码器工厂

`solrconfig.xml` 的 `CodecFactory` 用于确定在写入索引到磁盘时使用哪个 Lucene 的 [Codec](http://lucene.apache.org/core/6_0_0/core/org/apache/lucene/codecs/Codec.html)

如果不指定，使用 Lucene 默认的 Codec，但是 [solr.SchemaCodecFactory](http://lucene.apache.org/solr/6_0_0/solr-core/org/apache/solr/core/SchemaCodecFactory.html) 也是可用的，它支持 2 个特性

1. 基于 Schema 定义每个字段类型，`docValuesFormat` 和 `postingsFormat`，参考 [FieldType Definitions and Properties](https://cwiki.apache.org/confluence/display/solr/Field+Type+Definitions+and+Properties#FieldTypeDefinitionsandProperties-GeneralProperties)
2. `compressionMode` 选项
  * `BEST_SPEED` 默认，为搜索速度优化
  * `BEST_COMPRESSION` 为磁盘空间占用优化

示例

```xml
<codecFactory class="solr.SchemaCodecFactory">
  <str name="compressionMode">BEST_COMPRESSION</str>
</codecFactory>
```


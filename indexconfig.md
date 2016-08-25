# IndexConfig

`solrconfig.xml` 里的 `indexConfig` 定义了 Lucene 索引写入器的行为。在 solr 包含的示例 `solrconfig.xml` 里其设置默认是注释的，表示采用默认设置。多数场景下，默认设置很好

```xml
<indexConfig>
  ...
</indexConfig>
```

## writing new segment 写入新段

### ramBufferSizeMB

一旦累计的文档更新超过了这个存储空间(以 MB 为单位)，那么挂起的更新会立即刷入(<font color='red'>这意思是对文档的更新先在内存里执行，等到内存里的数据达到阈值，再统一从内存写入磁盘文件...吧...</font>)。这也可能创建一个新段(segment)或触发一次合并。使用这个设置要比 `maxBufferedDocs` 好。如果 `maxBufferedDocs` 和 `ramBufferSizeMB` 都在 `solrconfig.xml` 设置，那么任意一个限制达到都会触发一次刷入(flush)。默认值是 `100MB`

```xml
<ramBufferSizeMB>100</ramBufferSizeMB>
```

### maxBufferedDocs

设置在刷入到新段前在内存里缓冲多少个文档。这也可能会触发一次合并。solr 默认使用 `ramBufferSizeMB` 设置

```xml
<maxBufferedDocs>1000</maxBufferedDocs>
```

### useCompoundFile

最近的写入(尚未 merge)索引是否使用 `Compound File Segment` 格式，默认是 `false`

```xml
<useCompoundFile>false</useCompoundFile>
```

## merging index segment 合并索引段

### mergePolicyFactory

定义合并段怎么做。solr 默认使用 `TieredMergePolicy`，合并段时使用相同的大小。其他可用的策略有 `LogByteSizeMergePolicy` 和 `LogDocMergePolicy`

```xml
<mergePolicyFactory class="org.apache.solr.index.TieredMergePolicyFactory">
  <int name="maxMergeAtOnce">10</int>
  <int name="segmentsPerTier">10</int>
</mergePolicyFactory>
```
### Controlling Segment Sizes: Merge Factors



## compound file segment 复合文件段

## index locks 索引锁

## other indexing settings 其他索引设置


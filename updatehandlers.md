# UpdateHandlers

`solrconfig.xml` 的 `<updateHandler>` 元素的配置对索引更新有影响，其设置影响的是内部的更新是如何完成的，对 request hander 处理客户端的更新请求的的高级配置没有影响

```xml
<updateHandler class="solr.DirectUpdateHandler2">
  ...
</updateHandler>
```

## commit 提交

发送到 solr 的数据直到被提交到索引才可搜索。原因是在某些场景下提交较慢，且应与其他的提交请求隔离以避免覆盖数据。所以，在数据提交时最好进行控制。控制提交的时机有几个选项

### commit and softCommit 提交和软提交

在 solr 里，一个 `提交(commit)` 是要求 solr `提交` 修改到 Lucene 索引文件的行动。默认情况下，提交行动的结果是硬提交——所有 Lucene 索引文件到稳定的存储(磁盘)。当客户端更新请求包含了 `commit=true` 参数，这会确保被更新里的新增、删除影响的所有索引段会在索引更新结束时立即写入到磁盘。

如果指定了附加的标记 `softCommit=true`，solr 执行软提交，意思是 solr 会立即提交修改到 Lucene 数据结构但不保证 Lucene 文件写入到磁盘。这是近实时(Near Real Time)存储的一个实现，促进文档可见的一个特性，你不需要等待后台的合并和存储(如果使用 SolrCloud，则是对于 zk)结束。完整的提交意味着，如果服务器崩溃，solr 会精确的知道你的数据保存在哪；软提交意味着数据已存储，但是其位置信息尚未存储。由于不需要等待后台的合并结束，软提交给你更快的可见性。

关于 Near Real Time 操作的更多信息，参考 `Near Real Time Searching`

### autoCommit 自动提交

这些设置控制挂起的更新自动推送到索引的频率。可以用 `commitWithin` 取代 `autoCommit`，该参数可以在发送更新请求到 solr 时定义，或在一个 updateRequestRequest 里定义

| 设置 | 说明 |
| -- | -- |
| maxDocs | 从上次提交后至今发生的更新的数量 |
| maxTime | 最早的未提交更新开始至今的毫秒数 |
| openSearcher | 当执行一次提交时是否开启一个新的 searcher。<br>默认为false，本次提交会把最近的索引变更刷入磁盘，但不会开启一个新的 searcher 来使这些变更可见 |

如果 `maxDocs` 或 `maxTime` 任一个满足，solr 自动执行提交。如果没有 `autoCommit` 标签，那么只有明确的提交会更新索引。是否使用自动提交取决于你的应用。

决定最好的自动提交参数，是在性能和准确性(<font color='red'>这应该是搜索结果的准确性</font>)之间做平衡。频繁的更新会提升准确性，因为新的内容会更快的被搜索到，但会降低性能。降低频率能提升性能但需要更久才能在查询里显示更新

```xml
<autoCommit>
  <maxDocs>10000</maxDocs>
  <maxTime>1000</maxTime>
  <openSearcher>false</openSearcher>
</autoCommit>
```

可以像设定自动提交一样来设定软自动提交，只需要把 `autoCommit` 替换成 `autoSoftCommit`

```xml
<autoSoftCommit>
  <maxTime>1000</maxTime>
</autoSoftCommit>
```

### cmomitWithin

`commitWithin` 设置可强制文档在一个定义的时间段提交。多用于 `Near Real Time Searching`，默认是执行软提交。在主从架构下，不会复制新文档到从服务器。可以添加一个参数来强制硬提交，如下

```xml
<commitWithin>
  <softCommit>false</softCommit>
</commitWithin>
```

这样配置后，每次当你的更新信息的一部分是 `commitWithin`，就会自动执行硬提交

## Event Listeners 事件监听器

UpdateHandler 可配置更新相关的时间监听器。可以在任何提交(`event="postCommit"`)以后或者仅在优化指令(`event="postOptimize"`)之后被触发

可以编写自己的更新事件监听器，但一般是通过 `RunExecutableListerer` 来运行外部程序

| 设置 | 说明 |
| -- | -- |
| exe | 要运行的程序名称，应该是相对于 solr home 的路径名 |
| dir | 工作目录，默认是 "." |
| wait | 强制调用线程等待，直到程序返回响应。默认=true |
| args | 传送给程序的任意参数，默认为空 |
| env | 任意要设置的环境变量，默认为空 |

## Transaction Log 事务日志

在 `RealTime Get` 里描述过，事务日志是那个特性必须的。在 `solrconfig.xml` 的 `updateHandler` 里配置

实时获取当前依赖更新日志，默认是开启滴。配置如下

```xml
<updateLog>
  <str name="dir">${solr.ulog.dir:}</str>
</updateLog>
```

有 3 个专家级的配置项可影响索引性能及一个 replica 在必须进入全量恢复前可以落后于更新多远——参考 [写入侧容错](https://cwiki.apache.org/confluence/display/solr/Read+and+Write+Side+Fault+Tolerance#ReadandWriteSideFaultTolerance-WriteSideFaultTolerance) 获取更新信息

| 设置名称 | 类型 | 默认值 | 说明 |
| -- | -- | -- | -- |
| numRecordsToKeep | int | 100 | 每个日志多少条更新记录 |
| maxNumLogsToKeep | int | 10 | 最多几个日志 |
| numVersionBuckets | int | 65536 | 翻译不能... |

一个例子，`solrconfig.xml` 要包含在 `<config><updateHandler>`，使用如下高级设置

```xml
<updateLog>
  <str name="dir">${solr.ulog.dir:}</str>
  <int name="numRecordsToKeep">500</int>
  <int name="maxNumLogsToKeep">20</int>
  <int name="numVersionBuckets">65536</int>
</updateLog>
```
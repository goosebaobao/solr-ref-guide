# DataDir 和 DirectoryFactory

## 用 DataDir 参数设定索引数据的位置

Solr 默认将索引数据存储在 solr home 下的 `/data` 目录。如果你想用一个不同的目录来存储索引数据，在 `solrconfig.xml` 里用 `<dataDir>` 参数。可以用绝对路径或 SolrCore 实例目录的相对路径。

示例

```xml
<dataDir>/var/data/solr/</dataDir>
```

如果使用 solr 的主从复制功能来复制索引数据，那么从节点的 `<dataDir>` 要和主节点一致

## 为索引指定 DirectoryFactory

默认的 `solr.StandardDirectoryFactory` 基于文件系统，并尝试挑选当前 JVM 和平台最优的实现。可以强行指定一种：`solr.MMapDirectoryFactory`, `solr.NIOFSDirectoryFactory`, 或 `solr.SimpleFSDirectoryFactory`

```xml
<directoryFactory name="DirectoryFactory"
    class="${solr.directoryFactory:solr.StandardDirectoryFactory}"/>
```

`solr.RAMDirectoryFactory` 是基于内存的实现，不做持久化，在复制上不能工作。使用 DirectoryFactory 保存索引到 RAM.

```xml
<directoryFactory class="org.apache.solr.core.RAMDirectoryFactory"/>
```

> 如果使用 Hadoop 且把索引存储到 HDFS，可以使用 `solr.HdfsDirectoryFactory` 替换以上列举的实现


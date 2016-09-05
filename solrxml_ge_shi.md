# solr.xml 格式

本章节将描述包含在 solr 里的默认的 `solr.xml` 文件，及如何修改。

## 定义 Solr.xml

你可以在 solr 主目录或 zk 找到 `solr.xml`。默认的 `solr.xml` 看上去是这样滴

```xml
<solr>
  <solrcloud>
    <str name="host">${host:}</str>
    <int name="hostPort">${jetty.port:8983}</int>
    <str name="hostContext">${hostContext:solr}</str>
    <int name="zkClientTimeout">${zkClientTimeout:15000}</int>
    <bool name="genericCoreNodeNames">${genericCoreNodeNames:true}</bool>
  </solrcloud>
  <shardHandlerFactory name="shardHandlerFactory" class="HttpShardHandlerFactory">
    <int name="socketTimeout">${socketTimeout:0}</int>
    <int name="connTimeout">${connTimeout:0}</int>
  </shardHandlerFactory>
</solr>
```

如你所见，solr 配置是 solrcloud 友好的。但是，`<solrcloud>` 元素的出现*不*代表 solr 实例正在运行 solrcloud 模式。除非启动时指定了 `-DzkHost` 或 `-DzkRun`，否则该元素会被忽略

### solr.xml 参数

#### `<solr>`

`<solr>` 标签没有属性，这是 `solr.xml` 的根元素。下表是每个 xml 元素的子节点

| 节点 | 描述 |
| -- | -- |
| adminHandler | 如果使用的话，这个属性应类(继承自 CoreAdminHandler)的全路径名 FQN(Fully qualified name).例如， adminHandler="com.myorg.MyAdminHandler" 将配置自定义 admin handler(MyAdminHandler) 来处理管理请求。如果这个属性未设置，solr 使用默认的 admin handler，org.apache.solr.handler.admin.CoreAdminHandler。该参数的更多信息，参考 [wiki页面](http://wiki.apache.org/solr/CoreAdmin#cores) |
| collectionsHandler | 如上，可配置自定义的 CollectionHandler 实现 |
| infoHandler | 如上，配置自定义 InfoHandler 实现 |
| coreLoadThreads | 加载 core 同时分配的线程数 |
| coreRootDirectory | 发现 core 的树的根目录，默认是 SOLR_HOME |
| managementPath | 当前未用 |
| sharedLib | 所有 core 都共享的公共库目录的路径。这个目录的任何 JAR 文件都会被添加到搜索路径。该路径是相对于 solr 主目录 |
| shareSchema | 若设为 true，要确保多个 core 指向同一个 schema 资源文件，指向同一个 IndexSchema 对象。共享 IndexSchema 对象会加速 core 的加载。如果使用这个特性，确保在 schema 文件没有使用 core 的属性|
| transientCacheSize | 有多少个设置为 transient=true 的 core 在最近最少使用的 core 被新的 core 交换前被加载 |
| configSetBaseDir | solr core 的 configsets 目录，默认是 SOLR_HOME/configsets |

#### `<solrcloud>`

除非启动时指定了 `-DzkHost` 或 `-DzkRun`，否则该元素会被忽略

| 节点 | 描述 |
| -- | -- |
| distribUpdateConnTimeout | 为集群内部的更新设置底层的 "connTimeout" |
| distribUpdateSoTimeout | 为集群内部的更新设置底层的 "SocketTimeout" |
| host | solr 用来访问 core 的主机名 |
| hostContext | url 上下文路径 |
| hostPort | solr 访问 core 使用的端口。在默认的 `solr.xml` 文件，设置为 `${jetty.port:8983}` |
| leaderVoteWait | 当 solrcloud 启动，在假定任何没有汇报的 node 已经挂掉之前，等待 node 多久去发现 shard 的所有已知的 replica |
| leaderConflictResolveWait | 当尝试选举一个 shard 的 leader，这个属性设置一个 replica 将要等着看冲突状态的解决的最大时间；临时的冲突状态可能发生在重启时，尤其是 Overseer 所在的主机重启。典型的，默认值 180000 ms 足够冲突得以解决；如果你的 solrcloud 有成百上千的小 collection，你也许需要增加这个值 |
| zkClientTimeout | 用于 solrcloud，到 zk 服务器的连接的超时时间 |
| zkHost | 在 solrcloud 模式，solr 用来连接到 zk 主机的 url |
| genericCoreNames | 若为 `TRUE`，node 名字不是基于 node 的地址，而是基于通用名来标识 core。当一个不同的机器接管了 core 将更易理解 |
| zkCredentialsProvider & zkACLProvider | 可选参数，如果使用了 ZooKeeper Access Control |

#### `<logging>`

| 节点 | 描述 |
| -- | -- |
| class | 记录日志的类。相应的文件必须可用，可在 `solrconfig.xml` 用 `<lib>` 指定 |
| enabled | true/false - 是否启用日志 |

##### `<logging><watcher>`

| 节点 | 描述 |
| -- | -- |
| size | 缓冲的 log 事件数量 |
| threshold | 日志级别，以上的将被记录。例如，使用 log4j 可以被设定为 DEBUG， WARN， INFO 等等 |

#### `<shardHandlerFactory>`

自定义 Shard handler 

```xml
<shardHandlerFactory name="ShardHandlerFactory" class="qualified.class.name">
```

由于这是自定义的 shard handler，子元素取决于其实现

## 在 solr.xml 里的 jvm 系统属性

`solr.xml` 支持 JVM 系统属性值，运行时设定配置选项。语法是 `${属性名:[可选的默认值]}`。可以定义一个在运行时能被覆盖的默认值。如果默认值未设定，那么运行时必须指定属性值，否则 `solr.xml` 会在解析时报错。

启动 JVM 时使用 `-D` 设定的任意属性都可以用在 `solr.xml` 文件。

例如，在下面所示的 `solr.xml` 文件， `socketTimeout` 和 `connTimeout` 值都被设为 '0'。但是，如果你启动 solr 时使用 `"bin/solr -DsocketTimeout=1000"`，`HttpShardHandlerFactory` 的 `socketTimeout` 选项会用 1000ms 覆盖，同时 `connTimeout` 继续使用默认值 '0'

```xml
<solr>
  <shardHandlerFactory name="shardHandlerFactory" class="HttpShardHandlerFactory">
    <int name="socketTimeout">${socketTimeout:0}</int>
    <int name="connTimeout">${connTimeout:0}</int>
  </shardHandlerFactory>
</solr>
```
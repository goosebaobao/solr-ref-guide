# 定义 core.properties

core discovery(core 发现/感知)表示创建一个 core 就和磁盘上的 `core.properties` 一样简单(这 tmd 什么比喻...)。`core.properties` 文件是个简单的 JAVA 属性文件，每行是一个 k-v 对，例如 `name=core1`。注意不需要引号

一个最小化的 `core.properties` 文件看上去是这样的(但是，也可以是空的)

```ini
name=my_core_name
```

## core.properties 位置

在 `solr.home` 的子目录下放置名为 `core.properties` 的文件来配置 solr core。树的深度没有限制，可定义的 core 的数量也没有限制。core 可以在树的任何地方，但不能定义在一个已存在的 core 下。即，如下是不允许的

```bash
./cores/core1/core.properties
./cores/core1/coremore/core5/core.properties
```

这个例子里，将在 core1 停止枚举(<font color='red'>应该是说，solr 不会在 core1 的子目录里查看是否有 core.properties 文件存在</font>)

如下是合法滴

```bash
./cores/somecores/core1/core.properties
./cores/somecores/core2/core.properties
./cores/othercores/core3/core.properties
./cores/extracores/deepertree/core4/core.properties
```

solr 可以被分割为多个 core，每个 core 都有自己的配置和索引。core 可以专注于单个应用或另一个完全不同的，但是所有的管理操作都是通过一个公共的管理界面。你可以创建新的 core，停止 core，甚至用一个 core 替换另一个正在运行的 core，都不需要停止或重启 solr。

如有必要，你的 `core.properties` 文件可以是空的。假设一个空的 `core.properties` 文件，位于 `./cores/core1`(相对于 `solr_home`)，这个案例里，core 的名字假设为 "core1"，instanceDir 将是包含 `core.properties` 文件的目录(`./cores/core1`)，dataDir 将是 `../cores/core1/data`，等等

> 可不配置 core 来运行 solr

## 定义 core.properties 文件

最小化的 `core.properties` 文件是空文件，所有的属性都使用默认值。

java 属性文件可以用 "#" 或 "!" 注释一行

下面的表格定义了有效的属性

| 属性 | 描述 |
| -- | -- |
| name | solr core 的名字。当用 CoreAdminHandler 运行命令时用这个名字来指代 core |
| config | 给定的 core 的配置文件的名字，默认是 `solrconfig.xml` |
| shema | 给定的 core 的 schema 文件名，默认是 `schema.xml`，但是注意，如果使用 "managed schema"(默认行为)，那么任何不匹配 `managedSchemaRespurceName` 的属性值都只会读取一次，便转换成 managed shema。|
| dataDir | core 的数据目录(存储索引的目录)，一个绝对路径或者相对于 `instanceDir` 的目录，默认为 `data` |
| configSet | 配置集的名字，用来配置 core |
| properties | core 的属性文件的名字。绝对路径或者是相对(`instanceDir`)路径 |
| transient | 如为 **true**，core 会在 solr 达到 `transientCacheSize` 时被卸载。如果未指定，默认为 **fasle**。最近最少使用的 core 首先被卸载。*不建议在 solrcloud 模式设为 **true*** |
| loadOnStartup | 如为 **true**，未指定时的默认值，core 在 solr 启动时加载。*不建议在 solrcloud 模式设为 **false*** |
| coreNodeName | 只用于 solrcloud，这是承载 replica 的 node 的唯一标识。默认情况下，coreNodeName 自动生成，但是明确设定该属性允许你手工分配一个新 core 来代替一个已存在的 replica。例如：在新机器上用新的主机名或端口，用备份还原来取代一个硬件故障的机器 |
| ulogDir | core(solrcloud) 的更新日志(update log)的绝对或相对路径 |
| shard | 分配给这个 core 的 shard (solrcloud) |
| collection | 这个 core 所属的 collection (solrcloud) |
| roles | solrcloud 将来的参数，或用户用来标记 node 为自用的 |
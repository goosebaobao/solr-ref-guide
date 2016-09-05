# solr cores 及 solr.xml

在 solr 里，术语 *core* 指的是单个的索引及关联的事务日志和配置文件(其中包括 `solrconfig.xml`，schema 文件)。如果有必要，你的 solr 可以有多个core，让你可以在同一个服务器索引不同结构的数据，并控制你的数据如何展示给不同的用户。在 solrcloud 模式你可能对术语 *collection* 更熟悉。一个 collection 在后台是由一个或多个 core 组成的。

可用 `bin/solr` 脚本创建 core，或作为 solrcloud collection 的一部分使用 API 创建。core 的属性(诸如索引数据目录，配置文件目录，名称，及其他)在 `core.properties` 文件里定义。你的 solr 安装目录下的任何目录(或是在 `solr_home` 定义的目录)下的任何 `core.properties` 文件会被 solr 发现，其中定义的属性将会被用于文件里命名的 core。

在单机模式，`solr_home` 目录必须有 `solr.xml`。在 solrcloud 模式，如果存在，`solr.xml` 会从 zk 加载

> 在 solr 旧版本，必须在 `solr.xml` 的 `<core>` 里预先定义 core 让 solr 知道他们。现在，solr 支持自动发现 core，它们再不用被明确定义。推荐用 API 动态的创建 cores/collections


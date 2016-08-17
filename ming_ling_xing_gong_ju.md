# 命令行工具

Solr 管理页面(默认在 ` http://hostname:8983/solr/`)，提供了索引监控，执行统计，索引分布信息，副本信息，及 jvm 的线程信息。

此外，SolrCloud 也提供了管理页面(` http://localhost:8983/solr/#/~cloud`)，及 zk 命令行工具(CLI)。CLI 脚本在 `server/scripts/cloud-scripts`，让你上传配置信息到 zk。还有一些其他命令来集合集到集合，创建或清除 zk 路径，从 zk 下载配置到本地。

> Solr 的zkcli.sh 对比 Zookeeper 的 zkCli.sh
> 
> `zkcli.sh` 由 solr 提供，和 ZooKeeper 发行版所含的 `zkCli.sh` 不一样。
> 
> zk 的 `zkCli.sh` 提供了完整和通用的 zk 数据操作。 solr 的 `zkcli.sh` 为 solr 特有，用来处理 zk 里的 solr 数据




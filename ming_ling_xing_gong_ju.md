# 命令行工具

Solr 管理页面(默认在 ` http://hostname:8983/solr/`)，提供了索引监控，执行统计，索引分布信息，副本信息，及 jvm 的线程信息。

此外，SolrCloud 也提供了管理页面(` http://localhost:8983/solr/#/~cloud`)，及 zk 命令行工具(CLI)。CLI 脚本在 `server/scripts/cloud-scripts`，让你上传配置信息到 zk。还有一些其他命令来集合集到集合，创建或清除 zk 路径，从 zk 下载配置到本地。

> Solr 的zkcli.sh 对比 Zookeeper 的 zkCli.sh
> 
> `zkcli.sh` 由 solr 提供，和 ZooKeeper 发行版所含的 `zkCli.sh` 不一样。
> 
> zk 的 `zkCli.sh` 提供了完整和通用的 zk 数据操作。 solr 的 `zkcli.sh` 为 solr 特有，用来处理 zk 里的 solr 数据

## 使用 Solr 的 zk CLI

支持的命令行选项

| 简写 | 参数用法 | 含义 |
| -- | -- |
| | `-cmd <arg>` | 要执行的指令：`bootstrap`, `upconfig`, `downconfig`, `linkconfig`, `makepath`, `get`, `getfile`, `put`, `putfile`, `list`, `clear` 或 `clusterprop`。该参数是必须滴 |
| `-z` | `-zhhost <locations>` | zk 地址，该参数对所有指令都是必须滴 |
| `-c` | `-collection <name>` | 用于 `linkconfig`：collection 的名称 |
| `-d` | `-confdir <path>` | 用于 `upconfig`：配置文件的目录。<br>用于 `downconfig`：从 zk 拉取的文件保存的目标目录 |
| `-h` | `help` | 显示帮助信息 |
| `-n` | `-confname <arg>` | 用于 `upconfig`, `linkconfig`, `downconfig`：配置集的名称 |
| `-r` | `-runzk <port>` | 仅用于集群运行在单个机器上，传递 solr 运行的端口来运行内置的 zk |
| `-s` | `-solrhome <path>` | 当运行 `-runzk` 时，或用于 `bootstrap`：solrhome 的位置，必须滴 |
|  | -name <value> | 用于 `clusterprop`：集群的属性名，必须滴 |
|  | -val <value> | 用于 `clusterprop`：集群的属性值，如果未指定则为 null |

简写时用一个破折号(例如，`-c mycollection`)，完整格式时可以用 1 个破折号或 2 个破折号(例如，`-collection mycollection` 或 `--collection mycollection`)

## zk CLI 示例



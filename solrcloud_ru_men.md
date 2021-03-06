# SolrCloud 入门

SolrCloud 设计目标是提供一个高可用，容错的环境，将索引过的内容和查询请求分发到多台服务器上。

这是一个数据被组织为多份或多片，存储在多个服务器上，复制副本来以提供扩展性和容错的系统；使用 ZooKeeper 服务器管理所有属性，以使索引和查询请求可被正确路由。

本小节解释了 SolrCloud 及其内部工作细节，但是在你深入之前，最好是搞清楚你要达成什么目标。本页提供了一个简单的启动 Solr cloud 模式的操作说明，使你能对solr 分片在索引和查询时如何互相协作有个感性认识。为此，我们将用一个简单的例子，在单个服务器上配置 SolrCloud，显然这不是一个真正的，有多台服务器或虚拟机的生产环境。

## SolrCloud 示例

### 交互式启动

`bin/solr` 脚本使 SolrCloud 入门很简单，它引导你 cloud 模式启动 solr 并添加 collection，输入下述命令开始

```shell
$ bin/solr -e cloud
```

这会启动一个交互式的会话，引导你创建一个使用内置的 ZooKeeper 的、简单的 SolrCloud 集群。脚本询问你想要在本地集群创建几个节点（node），默认是 2 个

```shell
Welcome to the SolrCloud example!

This interactive session will help you launch a SolrCloud cluster on your local workstation.
To begin, how many Solr nodes would you like to run in your local cluster? (specify 1-4 nodes) [2]
```

脚本支持启动 4 个节点，但我们推荐 2 个。这些节点在一台机器上，使用不同的端口来模拟在不同机器上。接下来，脚本提示你每个节点绑定的端口

```shell
Please enter the port for node1 [8983]
```

为每个节点选择可用的端口，默认第一个节点是 8983，第二个节点是 7574。脚本将顺序启动各节点，并显示启动服务器的命令，如下

```shell
solr start -cloud -s example/cloud/node1/solr -p 8983
```

第一个节点将同时启动一个内置的 ZooKeeper，绑定 9983 端口，第一个节点的 solr home 目录为 `example/cloud/node1/solr`，由 `-s` 选项指定

在集群的所有节点都启动后，脚本提示你输入要创建的 collection 的名字

```shell
Please provide a name for your new collection: [gettingstarted]
```

默认的名字是 "gettingstarted"，你也可以选择一个更恰当的名字

然后，脚本会提示你这个 collection 跨越的分片的数量。

然后，脚本会提示你每个分片的副本数量。

最后，脚本会提示你的 collection 的配置目录的名字，你可以选择 **basic_configs**，  **data_driven_schema_configs**， 或 **sample_techproducts_configs**。配置目录是从 `server/solr/configsets/` 拉取的，所以你可以事先审核这些配置。如果你要为你的文档定义 schema，且需要一些灵活性，默认配置 **data_driven_schema_configs** 是很有用的。

此刻，你在本地的 Solr 集群里新建了一个 collection。为了验证这一点，可以执行 status 命令

```shell
$ bin/solr status
```

如果在这个过程中，碰到了任何的错误，在 `example/cloud/node1/logs` 和 `example/cloud/node2/logs` 查看 solr 的日志文件。

可以在 Solr 管理界面的 cloud 面板查看你的 collection 是如何发布到集群的：http://localhost:8983/solr/#/~cloud，Solr 也提供一个 healthcheck 命令来为 collection 执行简单的诊断

```shell
$ bin/solr healthcheck -c gettingstarted
```

healthcheck 命令收集 collection 里每个副本的基本信息，例如文档数量，当前状态（active，down，等），地址（副本在集群的位置）

现在可以用 post 工具将文档添加到 SolrCloud

要停止 solr，使用 stop 命令，如下

```shell
$ bin/solr stop -all
 ```

### -noprompt 启动选项

以默认配置启动 SolrCloud，而不是交互式会话模式

```shell
$ bin/solr -e cloud -noprompt
```

### 重启节点

使用 `bin/solr` 脚本重启 SolrCloud 节点，例如，重启端口为 8989 的节点 1 

```shell
$ bin/solr restart -c -p 8983 -s example/cloud/node1/solr
```

重启端口为 7574 的节点 2

```shell
$ bin/solr restart -c -p 7574 -z localhost:9983 -s example/cloud/node2/solr
```

注意，重启节点 2 时，你需要指定 ZooKeeper 的地址（-z localhost:9983），这样节点 2 才能加入到集群

### 向集群添加节点

向一个已存在的集群添加节点，有一点高级。。。和更多一点对 solr 的理解。当你使用 startup 脚本启动了一个 SolrCloud 集群后，可以用下面的命令添加节点

```
$ mkdir <solr.home for new solr node>
$ cp <existing solr.xml path> <new solr.home>
$ bin/solr start -cloud -s solr.home/solr -p <port num> -z <zk hosts string>
```

注意，上面的命令需要你创建 solr home 目录，你需要复制 `solr.xml` 到 `solr_home` 目录，或在 ZooKeeper `/solr.xml`

示例如下

```
$ mkdir -p example/cloud/node3/solr
$ cp server/solr/solr.xml example/cloud/node3/solr
$ bin/solr start -cloud -s example/cloud/node3/solr -p 8987 -z localhost:9983
```

上面的命令将在 8987 端口启动一个 solr 节点，其 home 目录为 `example/cloud/node3/solr`。新的节点将日志文件写入到 `example/cloud/node3/logs`
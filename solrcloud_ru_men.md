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


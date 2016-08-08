# 创建外部 ZooKeeper

虽然 Solr 附带了一个 ZooKeeper，你应该考虑使用内置的 ZooKeeper 的不利之处：关闭一个多余的 Solr 实例将同时关闭其附带的 ZooKeeper，而这个 ZooKeeper 也许并非多余。由于 ZooKeeper 集群在任何时候都需要至少其服务器数量的一半以上的节点在运行，这可能会是一个麻烦。

解决方案是一个外部的 ZooKeeper。

> **需要多少 ZooKeeper?**
> 
> 要保证一个 ZooKeeper 服务可用，需要多数的机器能彼此通信。如果可以容忍有 F 台机器处于故障状态，那么你至少要有 2\*F+1 台机器。因此，3 台机器可以有 1 台故障，5 台机器可以有 2 台故障。要注意 6 台机器只能有 2 台故障，因为 3 台机器并非多数。
>
> 基于这个理由，ZooKeeper 部署一般是奇数台机器。

当计划需要配置多少 zk 节点时，记住主要的原理是保证有多数的服务器在提供服务。通常建议有奇数台 zk 服务器组成你的 zk 服务。例如，如果你只有 2 个 zk 节点且其中一个宕机，50% 的可用服务器并非多数，所以 zk 将不能提供服务。但是，如果你有 3 个 zk 节点且其中一个宕机，你有 66% 的可用服务器，zk 将保持正常直到你修复宕机的节点。如果你有 5 节点，你可以在 2 个节点宕机的情况下继续运作。

## 下载 Apache ZooKeeper

第一步是下载 Apache ZooKeeper，[下载地址](http://zookeeper.apache.org/releases.html)

> 使用一个独立的 zk 时，你应注意保持 zk 的版本和 Solr 的要求一致。因为你的 zk 不会随着 Solr 的升级而自动升级版本。(相对于使用 Solr 内置的 zk 而言)
>
>  Solr 当前使用的 zk 版本为 v3.4.6

## 安装配置单个 ZooKeeper

### 创建实例

解压文件到指定的目录

### 配置实例

接下来是配置你的 zk 实例：创建如下的文件 `<ZOOKEEPER_HOME>/conf/zoo.cfg`，并添加如下的内容

```ini
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

参数如下

**tickTime**：以毫秒为单位，一个 tick 的时长。tick 是 zk 的一个时间单位，在 zk 检测服务器是否正常工作时(心跳)，最小的会话超时时间是 2 tick。

**dataDir**：zk 用这个目录来保存数据。该目录初始应该为空

**clientPort**：Solr 用来访问 zk 的端口

一旦这个文件就绪，你就已准备好启动 zk 实例了。

### 运行实例

使用 `ZOOKEEPER_HOME/bin/zkServer.sh` 脚本来运行 zk 实例，命令为 `zkServer.sh start`

zk 提供了其他很多强大的配置，但是深入研究这些内容已超出了这个说明的范围。对于本文的示例，默认的配置已足够好了。

### 在 Solr 中使用 zk 实例

在 Solr 里使用 zk 实例很简单：在 bin/solr 脚本使用 `-z` 参数。例如，要在 solr 里指向 你启动在 2181 端口的 zk，只需如此

随同已运行在 2181 端口的 zk 启动 `cloud` 例子(其他均为默认选项)

```shell
bin/solr start -e cloud -z localhost:2181 -noprompt
```

添加一个指向 2181 端口的 zk 的节点

```
bin/solr start -cloud -s <path to solr home for new node> -p 8987 -z localhost:2181
```





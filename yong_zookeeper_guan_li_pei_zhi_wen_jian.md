# 用 ZooKeeper 管理配置文件

使用 SolrCloud，你的配置文件保存在 zk。这些文件用以下方式之一上传到 zk

* 用 bin/solr 脚本启动 SolrCloud 示例应用
* 用 bin/solr 脚本创建一个集合(collection)
* 直接将配置上传到 zk

## 启动引导

当你用 bin/solr -e cloud 第一次启动 SolrCloud时，相关的配置自动上传到 zk 并连接到最近创建的集合(collection)

下面的命令启动 SolrCloud，连接到默认的集合(gettingstarted)，并上传默认的配置(data_driven_schema_configs)

```
$ bin/solr -e cloud -noprompt
```

使用 bin/solr 创建集合(collection)时，用 `-d` 选项直接上传一个配置目录

```
$ bin/solr create -c mycollection -d data_driven_schema_configs
```

create 命令会上传 `data_driven_schema_configs` 配置目录的一个副本到 zk 的 `/configs/mycollection`

一旦一个配置目录被上传到 zk，你就可以用 zkCLI( ZooKeeper Command Line Interface ) 来更新它们。

> 最好是将这些文件置于版本控制之下

## 用 zkcli 或 SolrJ 上传配置

在生产环境，也可以用 solr 的 zkcli.sh 脚本，或者 java 方法 CloudSlorClient.updataConfig() 上传配置

下面的命令使用 zkcli 脚本上传一个新的配置

```
$ sh zkcli.sh -cmd upconfig -zkhost <host:port> -confname <name for configset> -solrhome <solrhome> -confdir <path to directory with configset>
```

## 管理 SolrCloud 配置文件

要更新或更换 SolrCloud 配置文件

1. 从 zk 下载最新的配置文件，使用源码控制的检出
2. 进行变更
3. 提交更新后的文件到源码控制
4. 将变更推送到 zk
5. 重载集合(collection)以使变更生效




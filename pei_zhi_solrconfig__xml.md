# 配置 solrconfig.xml

`solrconfig.xml` 是拥有最多参数的 solr 配置文件。当配置 solr 时，通常是通过 `solrconfig.xml`，无论是直接编辑或是使用 Config API 创建配置覆盖(`configoverlay.json`)来覆盖 `solrconfig.xml` 里的值。

在 `solrconfig.xml` 里，配置如下的重要特性

* 请求处理器(request handler)，处理发送到 solr 的请求，诸如添加文档到索引的请求，或请求查询结果的请求
* 监听器(listener)，监听查询相关的特定事件，可用来触发特定代码的执行，诸如调用一些公共的查询来预热缓存
* http 请求转发
* 管理页面
* 复制、重复相关的参数

`solrconfig.xml` 文件在每个集合的 `conf/` 目录。在 `server/solr/configsets` 目录下可以发现几个注释完善的示例文件，示范了几种不同安装的最佳实践


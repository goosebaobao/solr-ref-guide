# 配置 solrconfig.xml

`solrconfig.xml` 是拥有最多参数的 solr 配置文件。当配置 solr 时，通常是通过 `solrconfig.xml`，无论是直接编辑或是使用 Config API 创建配置覆盖(`configoverlay.json`)来覆盖 `solrconfig.xml` 里的值。

在 `solrconfig.xml` 里，配置如下的重要特性

* 请求处理器(request handler)，处理发送到 solr 的请求，诸如添加文档到索引的请求，或请求查询结果的请求
* 监听器(listener)，监听查询相关的特定事件，可用来触发特定代码的执行，诸如调用一些公共的查询来预热缓存
* http 请求转发
* 管理页面
* 复制、重复相关的参数

`solrconfig.xml` 文件在每个集合的 `conf/` 目录。在 `server/solr/configsets` 目录下可以发现几个注释完善的示例文件，示范了几种不同安装的最佳实践

## 在 solr 配置文件里使用属性变量

在配置文件 `solrconfig.xml` 可以使用属性变量作为占位符，语法是 `${属性名:可选的默认值}`。以下是设定属性值的方法

### JVM 系统属性

启动 JVM 时，通过 `-D` 标识设定 JVM 系统属性，可以用在配置文件里。

例如，在 `solrconfig.xml` 文件里，可以这样定义锁定类型

```xml
<lockType>${solr.lock.type:native}</lockType>
```

这表示锁定类型(`lockType`)默认为`native`，但是在启动 JVM 时，可以用 JVM 系统属性来覆盖该默认值

```bash
bin/solr start -Dsolr.lock.type=none
```

通常，你可以传递任意 java 系统属性给 `bin/solr` 脚本，使用标准的 `-Dproperty=value` 语法，或者，添加公共的系统属性到 solr 包含文件(`bin/solr.in.sh`)里定义的 `SOLR_OPTS` 环境变量。

### solrcore.properties

如果在 solr core 的配置目录包含 `solrcore.properties` 文件，该文件可以定义任意的属性名及其值，这些属性都可以在配置文件里作为变量使用



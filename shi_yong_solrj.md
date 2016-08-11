# 使用 SolrJ

SolrJ 使 Java 程序更容易地和 Solr 通信。SolrJ 隐藏了大量连接到 Solr 的细节，允许你的程序和 Solr 之间用简单的高级方法交互。

SolrJ 的中心是 `org.apache.solr.client.solrj` 包，仅包含 5 个主要的类。创建一个 `SolrClient`，代表你想要使用的 Solr 实例，然后发送 `SolrRequests` 或 `SolrQuerys` 并获取 `SolrResponses`。

`SolrClient` 是个抽象类，所以要连接到远程的 Solr 实例，你实际创建的是 `HttpSolrClient` 或  `CloudSolrClient` 之一。都是通过 HTTP 与 Solr 通信，不同的是 `HttpSolrClient` 配置为使用一个确定的 Solr URL，而 `CloudSolrClient` 配置为使用 SolrCloud 集群的 zkHost 串。

```java
// 单节点 Solr 客户端

String urlString = "http://localhost:8983/solr/techproducts";
SolrClient solr = new HttpSolrClient(urlString);
```

```java
// SolrCloud 客户端

String zkHostString = "zkServerA:2181,zkServerB:2181,zkServerC:2181/solr";
SolrClient solr = new CloudSolrClient(zkHostString);
```

有了 `SolrClient` 后，就可以调用 `query()`，`add()`，和 `commint()` 方法。

## 创建和运行 SolrJ 应用程序

SolrJ api 包含在 Solr 里，所以你无需再下载或安装。但是，为了创建和运行 SolrJ 应用程序，需要添加一些库到类路径。

在创建阶段，本小节展示的例子需要 `solr-solrj-x.y.z.jar` 位于类路径。

在运行阶段，本小节展示的例子需要这些库在 'dist/solrj-lib' 目录。

使用 maven，将下面代码放入项目的 `pom.xml`

```xml
<dependency>
  <groupId>org.apache.solr</groupId>
  <artifactId>solr-solrj</artifactId>
  <version>x.y.z</version>
</dependency>
```

如果你担心 SolrJ 库会增加你的应用的大小，你可用代码混淆器(例如 [ProGuard](proguard.sourceforge.net))移除你没有用到的 api。

## 设置 XMLResponseParser

SolrJ 使用二进制格式而不是 XML 作为默认的响应格式。如果你使用 Solr 1.x 版本 和 SolrJ 3.x+ 版本，那么你**必须**使用 XML 响应。SolrJ 响应二进制格式在 3.x 版本变更过，导致了与 Solr 1.x 完全不兼容。

下面代码将返回格式设为 XML

```java
server.setParser(new XMLResponseParser());
```

## 执行查询

使用 `query()` 让 Solr 搜索结果。你必须传递一个描述查询的 `SolrQuery` 对象，并获得一个 QueryResponse (来自于 `org.apache.solr.client.solrj.response` 包)。

`SolrQuery` 有一些方法，可以让选择请求处理器和发送参数更简单。这里有一个很简单的例子，使用默认的请求处理器，设置查询串

```java
SolrQuery query = new SolrQuery();
query.setQuery(mQueryString);
```

要选择一个不同的请求处理器，在 SolrJ 4.0 和以后版本有个特定的方法

```java
query.setRequestHandler("/spellCheckCompRH");
```

你也可以在查询里设置任意参数。下面代码的前 2 行是互相等价的，第 3 行演示了如果在查询串里用一个 `q` 参数

```java
query.set("fl", "category,title,price");
query.setFields("category", "title", "price");
query.set("q", "category:books");
```

准备好你的 `SolrQuery` 后，用 `query()` 提交它

```java
QueryResponse response = solr.query(query);
```

客户端创建一个网络连接并发送该查询。Solr 处理查询，发送响应，由一个 `QueryResponse` 解析。

`QueryResponse` 是满足查询的文档的集合。你可以用 `getResults()` 来获取文档，还可以用其他方法来获取高亮和 facet 信息

```java
SolrDocumentList list = response.getResults();
```

## 索引文档

其他操作也是一样滴简单。要索引(添加)文档，你要做的是创建一个 `SolrInputDocument` 传递给 `SolrClient` 的 `add()` 方法。下面例子假设 SolrClient 对象 'solr' 已经创建

```java
SolrInputDocument document = new SolrInputDocument();
document.addField("id", "552199");
document.addField("name", "Gouda cheese wheel");
document.addField("price", "49.99");
UpdateResponse response = solr.add(document);

// Remember to commit your changes!

solr.commit();
```

## 以 XML 或 二进制格式更新内容

## 使用 ConcurrentUpdateSolrClient

## 内嵌的 SolrServer


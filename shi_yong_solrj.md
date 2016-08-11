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

## 设置 XMLResponseParser

## 执行查询

## 索引文档

## 以 XML 或 二进制格式更新内容

## 使用 ConcurrentUpdateSolrClient

## 内嵌的 SolrServer


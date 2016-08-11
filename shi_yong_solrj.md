# 使用 SolrJ

SolrJ 使 Java 程序更容易的和 Solr 通信。SolrJ 隐藏了大量连接到 Solr 的细节，允许你的程序和 Solr 之间用简单、高级方法交互。

SolrJ 的中心是 `org.apache.solr.client.solrj` 包，仅包含 5 个主要的类。创建一个 `SolrClient`，代表你想要使用的 Solr 实例，然后发送 `SolrRequests` 或 `SolrQuerys` 并获取 `SolrResponses`。


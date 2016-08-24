# Lib Directives

可以在 `solrconfig.xml` 里定义 `<lib/>` 来加载插件

插件按出现在 `solrconfig.xml` 的顺序加载，如果有依赖的库，先列出最底层的依赖库的 jar 包

可以使用正则表达式来加载同一个目录下的其他依赖的 jar 包。所有目录都是相对 Solr `instanceDir` 目录的

> `instanceDir` 即 core 的目录，一般是 `solr/server/solr/${core_name}`，`solrconfig.xml` 文件就在其下的 `conf/` 目录

```xml
<lib dir="../../../contrib/extraction/lib" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-cell-\d.*\.jar" />
<lib dir="../../../contrib/clustering/lib/" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-clustering-\d.*\.jar" />
<lib dir="../../../contrib/langid/lib/" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-langid-\d.*\.jar" />
<lib dir="../../../contrib/velocity/lib" regex=".*\.jar" />
<lib dir="../../../dist/" regex="solr-velocity-\d.*\.jar" />
```

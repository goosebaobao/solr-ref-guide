# RequestHandlers 和 SearchComponents

在 `solrconfig.xml` 的 `<query>` 之后配置 request handler(请求处理器) 和 search component(搜索组件)

一个 search component 是搜索的一个特性，诸如高亮或 Faceting。search component 在 `solrconfig.xml` 里定义，与  request handler 分开，并一起注册

## request handler

每个 request handler 都有名字和\(Java\)类。request handler 的名字被 solr 请求所引用，通常是个路径。举个例子，如果 solr 安装在  `http://localhost:8983/solr/`  且有个 collection  名为 `gettingstared`，可以发送如下的请求

```bash
http://localhost:8983/solr/gettingstarted/select?q=solr
```

这个查询将被名为 `/select` 的 request handler 处理。这里只使用了 `q` 参数，包含了一个简单的关键字 `solr`作为查询的词条。如果 request handler 定义了更多的参数，它们将被用于任何我们发送到这个 request handler 的查询，除非它们被客户端发送的查询覆盖(<font color='red'>)意思是可以在 `solrconfig.xml` 里定义 request handler 时指定参数的值，但客户端发送查询时可以用自己的参数值来覆盖定义在配置文件里的值</font>)

如果定义了另一个 request handler，应使用其名字来发送请求。举个例子，`/update` 是个处理索引更新(例如，发送新的文档来创建索引)的 request handler。`/select` 是默认用来处理查询请求的 request handler。

request handler 也可以处理嵌入其路径名的请求，例如，一个请求使用 `/myhandler/extrapath` 可能被名为 `/myhandler` 的 request handler 处理。如果有个 request handler 明确的定义了名字为 `/myhandler/extrapath` 那就优先处理这个嵌入的路径。这些都是假定你在使用 solr 自带的 request handler 类；如果使用自定义的 request handler，要确保其能处理嵌入路径。

可以用 `initParams` 配置 request handler 的默认值。这些默认值可以在每个独立的请求里作为通用的属性。例如，如果你想要创建返回相同 field 列表的多个 request handler，可以用 `initParams` 配置 field 列表。

### SearchHandlers

solr 默认定义的主要的 request handler 是 SearchHandler，用于处理搜索查询。这个 request handler 已定义，且有一系列默认值定义在 `defaults` 列

例如，在默认的 `solrconfig.xml`，第一个 request handler 是这样定义的

```xml
<requestHandler name="/select" class="solr.SearchHandler">
  <lst name="defaults">
    <str name="echoParams">explicit</str>
    <int name="rows">10</int>
  </lst>
</requestHandler>
```

这个例子定义了 `rows` 参数为 10，表示返回多少查询结果。`echoParams` 参数，表示如果有 debug 信息返回的话，是否在查询结果里返回。同时，注意在 list 里定义默认值的方式，如果参数是 string，integer，或其他类型。

所有这些在 `searching` 章节里描述过的参数，可以被作为默认值定义在任一个 SearchHandler 里

除了 `defaults`，SearchHandler 还有其他选项，如下

* appends：定义要添加到用户查询里的参数。可以是过滤查询，或其他应该添加到每个查询的查询规则。Solr 没有一个允许客户端覆盖这些额外的参数的机制，所以，你要绝对确信你总是想要在查询里应用这些参数

```xml
<lst name="appends">
    <str name="fq">inStock:true</str>
</lst>
```

在这个例子里，过滤查询 fq=inStock:true 将总是被加到每一个查询

* invariants：定义不能被客户端覆盖的参数，在 `invariants` 里定义的值将总是被使用，而不论用户，客户端传递或通过 `defaults`  或 `appends` 指定的值

```xml
<lst name="invariants">
  <str name="facet.field">cat</str>
  <str name="facet.field">manu_exact</str>
  <str name="facet.query">price:[* TO 500]</str>
  <str name="facet.query">price:[500 TO *]</str>
</lst>
``` 

在这个例子里，用于限制 facet 返回结果的字段已经被定义。如果客户端请求 facet，那么在这个配置里定义的 facet 是他/她唯一能看到的 facet。

reques hander 最后是 `componets`，定义一系列可以被 request hander 使用的 search component，只能和 request handler 一起注册。后面会讨论如何定义一个 search component。`components` 元素只能在 SearchHandler 这个 request handler 里使用。

`solrconfig.xml` 里包含了许多其他的 SearchHandler 例子，如有必要可以修改

### UpdateRequestHandlers

UpdateReqeustHandler 是更新索引的 request handler。

在本指南里，这些 handler 的细节在章节 `Uploading Data with Index Handlers`

### ShardHandlers

可以配置一个 request handler 跨越集群的多个 shard 做搜索，使用分布式搜索。更多有关分布式查询和配置 ShardHandler 的信息在章节 `Distributed Search with Index Sharding`

### Other Request Handlers

`solrconfig.xml` 定义了其他 request handler，在以下章节

* RealTime Get
* Index Replication
* Ping

## Search Components

search components 定义了 SearchHandlers 执行查询的逻辑

### Default Components

有几个和所有 SearchHandler 一起工作的默认的 search component 不需要任何附加的配置。如果没有用 `first-components` 和 `last-components` 定义的 component，默认会按下面所列出的顺序执行

| Component 名称 | 类名 | 更多信息 |
| -- | -- | -- |
| query | solr.QueryComponet | |
| facet | solr.FacetComponet | |
| mlt | solr.MoreLikeThisComponet | |
| highlight | solr.HighlightComponet | |
| stats | solr.StatsComponet | |
| debug | solr.DebugComponet | |
| expand | solr.ExpandComponet | - |

如果你用这些默认名称之一注册了一个新的 search componen，新定义的 component 将取代默认的

### First-Components and Last-Components

可以定义一些 component 在上面列出的默认 components 之前(`first-components`)或之后(`last-components`)被使用

> `first-components` 和/或 `last-components` 只能和默认 components 联合使用。如果自定义了 `components`，默认 components 将不会执行，且 `first-components` 和 `last-components` 也无效

```xml
<arr name="first-components">
  <str>mycomponent</str>
</arr>
<arr name="last-components">
  <str>spellcheck</str>
</arr>
```

### Components

如果自定义 `components`，前述的默认 components 将不会执行，且 `first-components` 和 `last-components` 也无效

```xml
<arr name="components">
  <str>mycomponent</str>
  <str>query</str>
  <str>debug</str>
</arr>
``` 

### Other Userful Components

本指南的其他章节描述了很多其他有用的 components，它们是

* `SpellCheckComponent`
* `TermVectorComponent`
* `QueryElevationComponent`
* `TermsComponent`


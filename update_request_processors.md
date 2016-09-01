# Update Request Processors

solr 接收的每个更新请求都运行在插件 Update Request Processor 的链上。这很有用，例如，要添加一个字段到索引的文档或修改某个字段的值或因提交的文档不符合规则而放弃更新。事实上，solr 大量的特性是作为 Update Processor 来实现的，因此理解该插件如何工作和哪里配置很有必要。

## Anatomy and life cycle 结构和生命周期

一个 Update Request Processor 作为一个或多个 update processor 链的一部分创建。solr 创建了一个默认的 update request processor 链实现一些基本的特性。这个默认的链用来处理每一个 update request 除非用户配置和指定一个不同的自定义的链

最简单的了解 update request processor 的方法是查看抽象类 [UpdateRequestProcessor](http://lucene.apache.org/solr/6_0_0//solr-core/org/apache/solr/update/processor/UpdateRequestProcessor.html) 的 javadoc 文档。每个 UpdateRequestProcessor 应该有一个相应的继承自 [UpdateRequestProcessorFactory](http://lucene.apache.org/solr/6_0_0/solr-core/org/apache/solr/update/processor/UpdateRequestProcessorFactory.html) 的工厂类。solr 用这个工厂类创建该插件的一个新实例。这种设计有 2 个好处

1. 一个 update request processor 不需要是线程安全的，因为它只会被一个请求线程使用并在请求结束后销毁
2. 工厂类可以接受配置参数并可能需要在请求间维持任意的状态。工厂类必须是线程安全的。

每个 update request processor 链在 solr core 加载期间构建并缓存到该 core 卸载。链里的每个 UpdateRequestProcessorFactory 也用 `solrconfig.xml` 里的配置实例化和初始化。

当一个更新请求被 solr 接收，它查找更新链用于该请求。链里的每个 UpdateRequestProcessor 的新实例被相应的工厂创建。这个更新请求被相应的 [UpdateCommand](http://lucene.apache.org/solr/6_0_0/solr-core/org/apache/solr/update/UpdateCommand.html) 对象解析。每个 UpdateRequestProcessor 实例负责调起链上的下一个插件。不调起下一个处理器会导致链式处理短路(<font color='red'>提前结束链式处理</font>)，甚至可以抛出异常来放弃后续处理

> 单个更新请求可能包含一批多个文档要保存或删除，因此 UpdateRequestProcessor 相应的 processXXX 方法会被调用多次。但是，可以保证是单个线程连续调用这些方法(<font color='red')这是说单个请求的一系列处理是串行的</font>)

## Configuration 配置

update request processor 链的创建，既可以在 solrconfig.xml 里直接创建完整的链，也可以在 solrconfig.xml 里个别的创建 update processor，然后在运行时用请求参数指定所有处理器动态的创建

但是，在我们理解如何配置 update processor 链之前，我们必须先学习默认的 update processor 链，因为它提供了在自定义的请求处理器链也需要的基本的特性

### The default update request processor chain 默认更新请求处理器链

如果 solrconfig.xml 里没有配置 update processor 链，solr 自动创建默认的 update processor 链作用于所有更新请求。这个默认的 update processor 链由下面的处理器按顺序组成

1.LogUpdateProcessorFactory - 在请求期间跟踪命令的处理并记下日志
2.DistributedUpdateProcessorFactory - 负责将更新请求分发到正确的节点。例如，路由请求到正确的 shard 的 leader，且将请求从 leader 分发到每个 replica。这个 processor 仅在 SolrCloud 模式才会激活
3.RunUpdateProcessorFactory - 使用 solr 内部的 api 来执行更新

每个 processor 执行一个基本功能，且自定义链通常会包含全部这些 processor。在自定义链里 RunUpdateProcessorFactory 通常是最后一个 processor

### Custom update request processor chain 自定义更新请求处理器链

下面的例子演示了如何在 solrconfig.xml 配置自定义链

```xml
<!-- updateRequestProcessorChain -->

<updateRequestProcessorChain name="dedupe">
  <processor class="solr.processor.SignatureUpdateProcessorFactory">
    <bool name="enabled">true</bool>
    <str name="signatureField">id</str>
    <bool name="overwriteDupes">false</bool>
    <str name="fields">name,features,cat</str>
    <str name="signatureClass">solr.processor.Lookup3Signature</str>
  </processor>
  <processor class="solr.LogUpdateProcessorFactory" />
  <processor class="solr.RunUpdateProcessorFactory" />
</updateRequestProcessorChain>
```

上面的例子，用 SignatureUpdateProcessorFactory， LogUpdateProcessorFactory， RunUpdateProcessorFactory 新建了一个名为 "dedupe" 的更新处理器链。 SignatureUpdateProcessorFactory 有几个参数，诸如 "signatureField"， "overwriteDupes" 等等。这个链示例了 solr 如何配置对文档去重，通过计算用 name， features， cat 字段的值的签名用作 id 字段。你可能注意到，这个例子没有指定 DistributedUpdateProcessorFactory - 因为这个处理器对 solr 正确运作及其重要，solr 将自动插入 DistributedUpdateProcessorFactory 到不包含它的链里，刚好在 RunUpdateProcessorFactory 之前。

> **RunUpdateProcessorFactory**
> 不要忘记将 RunUpdateProcessFactory 添加到你在 solrconfig.xml 里定义的任何链的末尾，否则那个链的更新请求将不会真正地更新索引数据

### 配置顶级处理器

在 solrconfig.xml 也可以配置独立于链的 update request processor 

```xml
<!-- updateProcessor -->

<updateProcessor class="solr.processor.SignatureUpdateProcessorFactory" name="signature">
  <bool name="enabled">true</bool>
  <str name="signatureField">id</str>
  <bool name="overwriteDupes">false</bool>
  <str name="fields">name,features,cat</str>
  <str name="signatureClass">solr.processor.Lookup3Signature</str>
</updateProcessor>

<updateProcessor class="solr.RemoveBlankFieldUpdateProcessorFactory" name="remove_blanks"/>
```

这个例子里，配置了名为 "signature" 的 SignatureUpdateProcessorFactory 和名为 "remove_blanks" 的 RemoveBlankFieldUpdateProcessorFactory。在 solrconfig.xml 如上定义后，就可以在 update request processor 链里引用他们，如下

```xml
<!-- updateRequestProcessorChains and updateProcessors -->

<updateProcessorChain name="custom" processor="remove_blanks,signature">
  <processor class="solr.RunUpdateProcessorFactory" />
</updateProcessorChain>
```

## Update processors in SolrCloud

solr 单机模式，每个更新只会被链里的更新处理器执行一次。在 SolrCloud 模式下更新处理器的行为值得特别考虑。

SolrCloud 的一项关键功能是路由和分发请求 - 对于更新请求这是由 DistributedUpdateRequestProcessor 实现的，且这个处理器因其重要功能被 solr 给与了特殊地位。

在分布式 SolrCloud 场景，链里的所有 DistributedUpdateProcessor *之前* 的处理器在接收客户端请求的第一个节点(node)上运行，不论这个节点是 leader 或 replica。然后 DistributedUpdateProcessor 转发更新到恰当的 shard 的 leader(或多个 leader，如果该更新影响多个文档，例如删除或提交)。Shard Leader 通过事务日志来运用原子更新，然后转发更新到 shard 的所有 replica。Leader 和每个 replica 执行链上的所有在 DistributedUpdateProcessor *之后* 的处理器。





## Using custom chains

## Update Request processor Factories
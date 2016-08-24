# Schema Factory

Solr 的 Schema API 使远程客户端通过 REST 接口访问 schema 信息和修改 schema

Schema API 支持读取所有类型的 schema，修改 schema 则取决于 `<schemaFactory/>`

## Managed Schema Default

若 `<schemaFactory/>` 未在 `solrconfig.xml` 明确指定，solr 隐含的会使用 `ManagedIndexSchemaFactory`，默认为 `mutable`，且将 schema 信息保存在 `managed-chema` 文件

```xml
<!-- An example of Solr's implicit default behavior if no
  no schemaFactory is explicitly defined.
-->
<schemaFactory class="ManagedIndexSchemaFactory">
  <bool name="mutable">true</bool>
  <str name="managedSchemaResourceName">managed-schema</str>
</schemaFactory>
```

选项

* `mutable`：能否对 schema 数据进行改变。必须设为 **true** 来允许 Schema API 进行修改
* `managedSchemaResourceName`：可选，默认为 `managed-schema`，可以定义为除了 `schema.xml` 以外的任何名称

使用上面的默认设置，就可以用 Schema API 来修改 schema。如果想要锁定 schema，阻止以后的修改，可以将 `mutable` 设为 **false**

## classic schema.xml

另一种使用可管理的 schema 的方式是明确的配置 `ClassicIndexSchemaFactory`，`ClassicIndexSchemaFactory` 需要 `schema.xml`，并且不允许运行时对 schema 的修改。`schema.xml` 必须手工编辑，且仅随 collection 加载时加载

```xml
<schemaFactory class="ClassicIndexSchemaFactory"/>
```

### 从 schema.xml 切换到 managed schema

如果一个已存在的 collection 使用 `ClassicIndexSchemaFactory`，想要转换成使用 managed schema，可以简单的修改 `solrconfig.xml`，指定使用 `ManagedIndexSchemaFactory`。

solr 重启时，检测到 `schema.xml` 文件存在，且 `managedSchemaResourceName` 指定的文件(如 `managed-schema`)不存在，则存在的 `schema.xml` 文件会被改名为 `schema.xml.bak`，且内容会重写到 `managed-schema`，文件顶部如下

```xml
<!-- Solr managed schema - automatically generated - DO NOT EDIT -->
```

现在可以自由地用 Schema API 来修改 schema，并可以删除掉 `schema.xml.bak`

### 改为手工编辑的 schema.xml

如果 solr 启动为使用 managed schema，想要切换到手工编辑 `schema.xml`，按如下步骤

1. 将 `managed-schema.xml` 更名为 `schema.xml`
2. 修改 `solrconfig.xml` 替换 `schemaFactory` 类
  3. 清除 `ManagedIndexSchemaFactory` 的定义
  4. 添加 `ClassicIndexSchemaFactory` 的定义
5. 重载 core

如果使用的是 SolrCloud，需要修改 zk 上的文件
# RequestDispatcher

`solrconfig.xml` 的 `requestDispatcher` 元素控制了 solr 的 HTTP `RequestDispatcher` 实现如何响应请求。其参数定义了是否要处理 `/select` url，是否应支持远程流，上传文件的最大尺寸，及它如何响应 HTTP cache 头

## handleSelect

> `handlerSelect` 是为了向后兼容，新的 solr 爱好者没有必要修改默认的配置

`<requestDispatcher>` 元素的首个配置项是 `handleSelect`。取值为 "true" 或 "false"。它控制着 solr 怎样响应 `/select?qt=XXX` 这样的请求。默认值为 "false" 表示如果一个 requestHandler 没有明确的注册到 `/select` 则忽略 `/select`。"true" 将路由查询请求到 `qt` 定义的解析器

在 solr 最新版本里，`/select` requestHandler 是默认定义的，所以 `false` 设置会很好的生效。参考 [RequestHandler 和 SearchComponent](requesthandlers_he_searchcomponents.md)获取更多信息

```xml
<requestDispatcher handleSelect="true" >
  ...
</requestDispatcher>
```

## requestParsers

`requestParsers>` 控制解析请求。这是个空 xml 元素，没有内容只有属性。

`enableRemoteStreaming` 属性控制是否允许远程内容流。`false` 不允许流，默认为 `true` 让你指定内容的位置，使用 `stream.file` 或 `stream.url` 参数

如果允许远程流，确保开启验证。否则，其他人可能通过任意 url 访问你的内容。把 solr 放在防火墙后面来防止不受信任的客户端访问也是个好主意。

`multipartUploadLimitInKB` 设定了通过 HTTP POST 请求提交的文档的最大尺寸，单位为 KB。

`formDataUploadLimitInKB` 设定了 HTTP POST 提交的表单数据(application/x-www-form-urlencoded)的最大尺寸，KB

`addHttpRequestToContext` 用于表明原始的 `HttpServletRequest` 对象应该被包含在 `SolrQueryRequest` 的上下文里，使用 `httpRequest` 为 key。这个 `HttpServletRequest` 不会被任何 Solr component 使用，但开发自定义的插件时可能会有用。

```xml
<requestParsers enableRemoteStreaming="true"
    multipartUploadLimitInKB="2048000"
    formdataUploadLimitInKB="2048"
    addHttpRequestToContext="false" />
```

## httpCaching

`httpCaching` 控制 HTTP cache 控制头。不要和 solr 内部的 cache 配置混淆，这个元素控制 HTTP 响应的 caching 是在 W3C HTTP 规范里定义的

这个元素有 3 个属性和 1 个子元素。`<httpCaching>` 控制 GET 请求是否返回一个 304，如果是，响应应该是啥。当一个 HTTP 客户端应用发布一个 GET，如果资源在它上次获取以后没有被修改，会指定一个 304 响应

| 参数 | 描述 |
| -- | -- |
| never304 | `true` 表示 GET 请求永远不会返回 304，即便请求的资源没有被修改。同时其他 2 个属性被忽略。设为 true 是为了方便开发 |
| lastModFrom | 取值为 `openTime`(默认) 或 `dirLastMod`，`openTime` 表示最后修改时间，即用来与客户端发送的 If-Modified-Since 头信息比较的时间，是 searcher 启动的时间。如果你想要索引最后在磁盘更新的精确时间，使用 `dirLastMod` |
| etagSeed | `ETag` 头信息的值。修改这个值有助于强制客户端重新获取内容，即使索引没有改变——例如，如果对配置做了修改。|

```xml
<httpCaching never304="false"
      lastModFrom="openTime"
      etagSeed="Solr">
  <cacheControl>max-age=30, public</cacheControl>
</httpCaching>
```

## cacheControl

上面的属性之外，`<httpCaching>` 有个子元素： `<cacheControl>`。这个元素的内容将作为 HTTP 响应头 Cache-Control 的值。这个头用来修改请求客户端的默认的 caching 行为。Cache-Control 头可能的取值在 HTTP 1.1 规范的 [14.9](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9) 章节定义

设置 max-age 字段控制客户端在重新请求之前可以重用 cache 响应多久。设置这个时间间隔应根据你多久更新你的索引，且无论你的应用程序使用的内容有点点过期。

`must-revalidate` 设置要求客户端在重用前向服务器确认其 cache 副本依然有效。这会确保使用更及时的结果，避免不必要的第二次获取
# RequestDispatcher

`solrconfig.xml` 的 `requestDispatcher` 元素控制了 solr 的 HTTP `RequestDispatcher` 实现如何响应请求。其参数定义了是否要处理 `/select` url，是否应支持远程流，上传文件的最大尺寸，及它如何响应 HTTP cache 头

## handleSelect

> `handlerSelect` 是为了向后兼容，新的 solr 爱好者没有必要修改默认的配置

`<requestDispatcher>` 元素的首个配置项是 `handleSelect`。取值为 "true" 或 "false"。它控制着 solr 怎样响应 `/select?qt=XXX` 这样的请求。默认值为 "false" 表示如果一个 requestHandler 没有明确的注册到 `/select` 则忽略 `/select`。"true" 将路由查询请求到 `qt` 定义的解析器

在 solr 最新版本里，`/select` requestHandler 是默认定义的，所以 `false` 设置会很好的生效。参考 [RequestHandler 和 SearchComponent](requesthandlers_he_searchcomponents.md)获取更多信息

## requestParsers

## httpCaching


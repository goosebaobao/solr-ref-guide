# InitParams

`solrconfig.xml` 里的 `<initParams>` 使你可在 handler 配置之外定义 request handler 参数。

用例是

* 一些 handler 是隐含定义的(<font color='red'>应该是说某些属性在源码里硬编码了默认值</font>)，应该有一个新增/添加/覆盖这些隐含属性的方法
* 少数属性用于多个 handler。这样可以仅定义一次这些属性并应用于多个 handler

例如，如果想要几个 handler 返回相同的字段列表，可以创建一个 `<initParams>` 元素而不用在每一个 handler 里定义相同的参数。如果有单个 handler 要返回不同的字段，可以在 `<requestHandler>` 里单独定义参数来覆盖

`<initParams>` 里的属性和配置镜像了 request handler 的属性和配置(这应该是说其 xml 节点和 `<requestHandler>` 一样)。它能包含 defaults，appends，invariants，和任何 request handler 一样
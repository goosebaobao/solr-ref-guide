# InitParams

`solrconfig.xml` 里的 `<initParams>` 使你可在 handler 配置之外定义 request handler 参数。

用例是

* 一些 handler 是隐含定义的(<font color='red'>应该是说某些属性在源码里硬编码了默认值</font>)，应该有一个新增/添加/覆盖这些隐含属性的方法
* 少数属性用于多个 handler。这样可以仅定义一次这些属性并应用于多个 handler

例如，如果想要几个 handler 返回相同的字段列表，可以创建一个 `<initParams>` 元素而不用在每一个 handler 里定义相同的参数。如果有单个 handler 要返回不同的字段，可以在 `<requestHandler>` 里单独定义参数来覆盖

`<initParams>` 里的属性和配置镜像了 request handler 的属性和配置(这应该是说其 xml 节点和 `<requestHandler>` 一样)。它能包含 defaults，appends，invariants，和任何 request handler 一样。

例如，在 `data_driven_config` 例子里的一个 `<initParams>` 定义

```xml
<initParams path="/update/**,/query,/select,/tvrh,/elevate,/spell,/browse">
  <lst name="defaults">
    <str name="df">_text_</str>
  </lst>
</initParams>
```

这为 path 里的全部 request handler 设置了默认的搜索字段("df")为 "\_text\_"。如果以后想要 `/query` request handler 默认搜索一个不同的字段，可以在 `<requestHandler>` 里定义这个参数覆盖 `<initParams>`

语法和 `<requestHandler>` 类似，如下

| 属性 | 描述 |
| -- | -- |
| path | 逗号分隔的路径列表，可以用通配符来定义嵌入路径，如下文所述 |
| name | 这一批参数的名称。如果没有一个确定的路径，这个名字可以直接用在 request handler 里。如果给 `<initParams>` 一个名称，就可以在 `<requestHandler>` 里引用这个名称<br> 例如，一个 `<initParams>` 的名称为 "myParams"，可以在定义 request handler 时使用 <br><br> `<requestHandler name="/dump1" class="DumpRequestHandler" initParams="myParams"/>` |

## 通配符

`<initParams>` 支持用通配符定义嵌套路径。单个星号(\*)指示嵌套路径深一层，2 个星号(\*\*)指示所有深度的嵌套路径。

例如，有如下的 `<initParams>`

```xml
<initParams name="myParams" path="/myhandler,/root/*,/root1/**">
  <lst name="defaults">
    <str name="fl">_text_</str>
  </lst>
  <lst name="invariants">
    <str name="rows">10</str>
  </lst>
  <lst name="appends">
    <str name="df">title</str>
  </lst>
</initParams>
```

定义了 3 个路径

* `/myhandler` 声明了一个明确的路径
* `/root/*` 表示 `/root` 下的路径，不含子目录
* `/root1/**` `/root`` 开头的所有路径

当定义 request handler 时，通配符将如下工作

```xml
<requestHandler name="/myhandler" class="SearchHandler"/>
```

`/myhandler` 类在 `<initParams>` 里定义了 `path`，所以会使用那些参数

```xml
<requestHandler name="/root/search5" class="SearchHandler"/>
```

`/root` 下定义了一层深度，所以这个 request handler 将使用这些参数，但是，下面这个不止一层深度就不会使用

```xml
<requestHandler name="/root/search5/test" class="SearchHandler"/>
```

想要定义所有深度的嵌套路径，应该使用双星号，如同 `/root1/**`

```xml
<requestHandler name="/root1/search/tests" class="SearchHandler"/>
```

`/root1` 下的任意路径，不论是否在 request handler 里明确定义，都会使用 `initParams` 里定义的参数





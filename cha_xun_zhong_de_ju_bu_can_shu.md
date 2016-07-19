# 查询中的局部参数

局部参数是 solr 请求参数的参数（？），提供了一种添加元数据到某个参数的方法

局部参数被指定为参数前缀（？），如下例的查询

`q= solr recks`

可以在查询前置局部参数来为标准查询解析器提供更多的信息，例如：修改默认操作类型为 AND，默认字段为 title

`q={!q.op=AND df=title}solr rocks`

## 局部参数基本语法

要指定一个局部参数，在要修改的参数前插入

* 以 `{!` 开头
* 任意个数的键值对，即 key=value，以空格分隔
* 以 `}` 结尾，且后面紧跟原查询参数

每个参数仅可指定一个局部参数，键值对里的值可以用单引号或双引号包围，反斜线作为引号的转义符

## 查询类型简写

如果一个局部参数看上去只有值而没有名字，表明使用了隐含的名字 `type`。表明使用哪种查询解析器来解析查询语句，即

`q={!dismax qf=myfield}solr rocks` 等价于 `q={!type=dismax qf=myfield}solr rocks`

如果未指定 `type` 那么 lucene parser 是默认的解析器，即

`fq={!df=summary}solr rocks` 等价于 `fq={!type=lucene df=summary}solr rocks`

## 用'v'键指定参数值

局部参数里的特殊键 `v`，可以用来代替原查询参数，即

`q={!dismax qf=myfield}solr rocks` 等价于 `q={!type=dismax qf=myfield v='solr rocks'}`

## 参数无关

参数无关或间接让你使用另一个参数的值而不是直接引用它（？）

`q={!dismax qf=myfield}solr rocks` 等价于 `q={!type=dismax qf=myfield v=$qq}&qq=solr rocks`

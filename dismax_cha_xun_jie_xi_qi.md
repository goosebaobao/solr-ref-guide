# DisMax 查询解析器

DisMax 查询解析器被设计来处理简单短语和多个不同权重的字段。

通常，DisMax 查询解析器更像 google 而不是标准查询解析器。

DisMax 支持 Lucene 查询解析器语法的精简子集

好奇 DisMax 这个名字的来源？DisMax 代表 Maximum Disjunction。不论是否能记住这个名字的含义，只要记住 DisMax 是易于使用，能接受任何输入且不会出错的。

## 参数

| 参数 | 说明 |
| :--- | :--- |
| q | 查询的原始输入 |
| q.alt | 如果 q 参数未使用，这个参数用来呼叫标准查询解析器解析输入 |
| qf | query fields，指定执行查询时所需要的索引字段，如果未指定则为 df 定义的字段 |
| mm | 最小匹配数，指定查询最少要匹配几个条件，如果没有指定该参数，则依赖于 q.op 参数（不论该参数是在查询条件里指定，还是在 solrconfig.xml 里指定的默认值，或 schema.xml 里通过 defaultOperator 选项指定）：q.op = AND 则 mm = 100%；q.op = OR 则 mm=1。用户也可以在 solrconfig.xml 里指定 mm 的默认值。该参数还允许在表达式里混入空格，例如 " 3 &lt; -25% 10 &lt; -3\n", " \n-25%\n", " \n3\n " |
| pf | phrase fields：增加文档的分数以防 q 参数的所有词条出现在附近 （神马意思?） |
| ps | phrase slop：指定 2 个词条分开的位置数，以匹配一个特定的短语 |
| qs | query phrase slop：含义同 ps，一般和 qf 参数一起适应 |
| tie | Tie Breaker：一个远小于 1 的浮点值，在 DisMax 查询里作为决定性因素 （这 tmd 又是神马意思？） |
| bq | boost query：指定一个因子，对于匹配时需要加强重要性的那些词条或短语 |
| bf | Boost Functions：指定一个函数来加权 |

###qf

字段列表，每个字段都有一个权重因子来增强或减弱该字段在查询中的重要性，示例

`qf="fieldOne^2.3 fieldTwo fieldThree^0.4"`

上面例子表示 fieldOne 的权重因子为2.3，fieldTwo 为默认的权重因子，fieldThree 的权重因子为0.4mm
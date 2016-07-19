# 函数查询

函数查询允许用一个或多个数值字段生成一个相关度的分数，标准查询解析器、DisMax、eDisMax都支持函数查询

函数查询使用函数，函数可以是一个常数（数字或字符串），一个字段，另一个函数，或参数替换参数（不懂...），可以用函数来修改结果的排名(Ranking)，可以用于对于根据用户的位置，或其他的考虑，来改变结果的排名

## 使用函数查询

函数表现为函数调用，例如 `sum(a,b)` 而不是 `a+b`

在 solr 查询里，有几种使用函数查询的方法

* 通过一个明确的、期待函数参数的 QParser，如 `func` 或 `frange`，例如
  * `q={!func}div(popularity,price)&fq={!frange l=1000}customer_ratings`
* 在一个排序表达式中，例如
  * `sort=div(popularity,price) desc, score desc`
* 将函数结果值作为查询结果文档的伪字段（区别与真实字段），例如
  * `&fl=sum(x, y),id,a,b,c,score`
* 明确需要函数的参数，如 eDisMax 的 `boost` 参数，或 DisMax 的 `bf` 参数（注意 bf 参数接收多个以空格分隔的函数查询，每个函数查询有一个可选的 boost 因子，确保在单个函数内的空格被排除），例如
  * `q=dismax&bf="ord(popularity)^0.5 recip(rord(price),1,1000,1000)^0.3"`
* 使用 Lucene QParser 的 \_val\_ 关键字引入函数查询，例如
  * `q=_val_:mynumericfield _val_:"recip(rord(myfield),1,2,3)"`

只推荐可快速随机存取的函数（？）

## 可用的函数

| 函数 | 说明 | 语法示例 |
| -- | -- | -- |
| abs | 绝对值 | `abs(x)` <br> `abs(-5)` |
| and | 仅当所有操作数都为 true 时返回 true | `and(not(exists(popularity)),exists(price))`<br>&nbsp;任何文档，字段 price 有值，且字段 popularity 没有值时返回 true |
| "constant" | 浮点常数 | 1.5 |
| def | default 的简写，若值 1 不存在则返回值 2 | `def(rating,5)`：如果 rating 存在则返回之，否则返回 5 <br>`def(myfield, 1.0)`：如果字段 myfield 存在返回其值，否则返回 1.0 |
| div | 除法，div(x,y) 表示 x 除以 y | `div(1,y)`<br>`div(sum(x,100),max(y,1))` |
| dist | 返回 n 维空间里 2 个矢量的距离 | `dist(2, x, y, 0, 0)`：(0,0) 和 (x,y) 之间的欧氏距离<br>`dist(1, x, y, 0, 0)`: (0,0) 和 (x,y) 之间的曼哈顿距离<br>`dist(2, x,y,z,0,0,0)`：(0,0,0) 和 (x,y,z) 之间的欧氏距离<br>`dist(1,x,y,z,e,f,g)`: (x,y,z) 和 (e,f,g) 之间的曼哈顿距离，每个字符是一个字段名 |
| docfreq(field,val) | 返回这个字段里包含这个值的文档数量，这是个常数 | `docfreq(text,'solr')`<br>`defType=func&q=docfreq(text,$myterm)&myterm=solr` |
| exists | 如果任意字段存在就返回 true | `exists(author)` 任何文档在 author 字段有值就返回 true<br>`exists(query(price:5.00))` price 匹配 5.00 就返回 true |
| field | 返回字段的 docValues 或 indexed 值（数量？），对于 docValues，可以添加可选参数 min 或 max，返回 0 表示文档木有该字段 | 如下示例是等价的<br> `myFloatFieldName`<br>`field(myFloatFieldName)`<br>`field("myFloatFieldName")`<br>如果字段名是非典型的（例如包含了空格）用第三种写法<br>对于多值的 docValues 字段，示例如下<br>`field(myMultiValuedFloatField,min)`<br>`field(myMultiValuedFloatField,max)`|
| hsin |  |  |
| idf |  |  |
| if | 条件判断，语法为<br>`if(test,value1,value2)`<br>`test` 是个逻辑值或逻辑表达式<br>`value1` test = true 时返回<br>`values` test = false 时返回<br>表达式可以是任意返回逻辑值的函数，或是返回数值的函数，此时 0 表示 false，或是 string，此时空串表示 false | `if(termfreq(cat,'electronics'),popularity,42)`<br>&nbsp;该函数检查每一文档，字段 cat 是否包含词条 "electronics"，如果包含，返回 popularity 字段的值，否则返回 42 |
| linear | `m*x+c` 的函数形式，m、c 为常数，x 为变量<br>等价于 `sum(product(m,x),c)` 但是性能好些，因为只有一次函数调用 | `linear(x,m,c)`<br>`linear(x,2,4)` 返回 `2*x+4` |
| log | 对数，以 10 为底数 |  |
| map | 根据输入是否落在某个范围来返回值，范围参数 min，max 必须是常数。如果输入 x 不在 min 和 max 之间，返回值为 x 或 default（如果指定了该参数） | `map(x,min,max,target)`<br>`map(x,0,0,1)` - 将 0 改为 1，处理默认值 0 时有用<br>`map(x,min,max,target,default)`<br>`map(x,0,100,1,-1)` - 将 0~100 之间的值改为 1，其他值 改为 -1 |
| max | 返回最大值 `max(x,y,...)` | `max(myfield,myotherfield,0)` |
| maxdoc | 返回索引的文档数量，包括已标记为删除但还没有物理删除的文档，这是个常数，对于索引里的任何文档都是同样的值 | `maxdoc()` |
| min | 返回最小值 |  |
| ms | 返回参数之间的时间差异，以毫秒为单位，参数可以是索引的 TrieDateField 字段，或NOW，或基于日期常数的日期计算<br>`ms()` 等价于 `ms(NOW)` 当前时间与 epoch 之间的毫秒差<br>epoch 1970-1-1 00:00:00.000<br>`ms(a)` a 与 epoch 之间的毫秒差<br>ms(a,b) : 时间 a 与 b 之间的毫秒差，注意 a 发生在 b 之后，即 a - b | `ms(NOW/DAY)`<br> `ms(2000-01-01T00:00:00Z)`<br> `ms(mydatefield)`<br>`ms(NOW,mydatefield)`<br> `ms(mydatefield,2000-01-01T00:00:00Z)`<br> `ms(datefield1,datefield2)` |
| norm(field) |  |  |
| not | 逻辑非 |  |
| numdocs | 返回索引的文档数量，不包括已标记为删除但还没有物理删除的文档，这是个常数，对于索引里的任何文档都是同样的值 | `numdocs()` |
| or | 逻辑或 | `or(value1,value2)` value1 和 value2 都为 true 时返回 true |
| ord | 返回索引字段的值的序号，从 1 开始。排序是依字典顺序，该字段为单值字段。若字段没有值返回 0 |  |
| pow | 幂值，`pow(x,y) =x^y` | `pow(x,y)`<br>`pow(x,log(y))`<br>`pow(x,0.5)` 等价于 `sqrt` |
| product | 乘积，参数为逗号分隔的值或函数，等价于 `mul(...)` | `product(x,y,...)` |
| query | 返回指定的子查询的分数，如果子查询未匹配到文档的话返回一个默认值 | `query(subquery, default)`<br>`q=product(popularity, query({!dismaxv='solr rocks'})` 返回 popularity 和 DisMax 查询的分数的乘积<br>`q=product(popularity, query($qq))&qq={!dismax}solr rocks`: equivalent to the previous query, using parameter de-referencing.<br>`q=product(popularity, query($qq,0.1))&qq={!dismax}solr rocks`: specifies a default score of 0.1 for documents that don't match the DisMax query |
| recip | `recip(x,m,a,b) = a/(m*x+b)`  m,a,b 为常数, x 是个函数 | `recip(myfield,m,a,b)`<br>`recip(rord(creationDate),1,1000,1000)` |
| rord | ord 的反转 | `rord(myDateField)` |
| scale | 将输入 x 缩放到 minTarget 和 maxTarget 之间，当前的实现会遍历所有的 x 值来确定 min 和 max 的值<br>如果文档被删除或者没有值，则会视同为0 | `scale(x,minTarget,maxTarget)`<br>`scale(x,1,2): scales the values of x such that all values will be between 1 and 2 inclusive.` |
| sqedist | 欧氏距离的平方， | `sqedist(x_td, y_td, 0, 0)` |
| sqrt | 平方根 | `sqrt(x)sqrt(100)sqrt(sum(x,100))` |
| strdist | 计算 2 个字符串的距离，格式为 `strdist(string1, string2, distance measure)` | `strdist("SOLR",id,edit)`<br>&nbsp;&nbsp;distance measure 取值为<br>`jw`: Jaro-Winkler<br>`edit`: Levenstein or Edit distance<br>`ngram`: NGramDistance, 可以指定一个可选的 ngram size 参数，默认为 2<br>`FQN`: 实现了 StringDistance 接口的类名，该类必须有一个无参的构造函数 |
| sub | 减法，sub(x,y) 即为 x - y | `sub(myfield,myfield2)` |
| sum | 求和，add() 可以作为别名使用 | `sum(x,y,...)` |
| sumtotaltermfreq | 返回 `totaltermfreq` 之和 | `sttf()` |
| termfreq | 词条在字段里出现的次数 | `termfreq(text,'memory')` |
| tf | term frequence，返回一个给定词条的词条频率因子 | `tf(text,'solr')` |
| top | 貌似是在顶层索引里取值，同一个值在某个段的序号和其在整个索引里的序号是不一样的 | `ord()`，`rord()` 实际是隐含了 `top()` 调用，即 `ord()` 等价于 `top(rod())` |
| totaltermfreq | 整个索引里，词条在字段里出现的次数 | `ttf(text,'memory')` |
| xor() | 逻辑异或 | `xor(field1,field2)` field1 ，field2 都为 true 则返回 false，否则返回 true |

有 2 个文档，doc1，doc2，都有字段 fieldX，其中
* doc1.fieldX = [A B C]
* doc2.fieldX = [A A A A]

则

* `docFreq(fieldX:A) = 2` A 在 2 个文档里出现
* `freq(doc2, fieldX:A) = 4`  A 在 doc2 里出现 4 次
* `totalTermFreq(fieldX:A) = 5`  A 在所有文档里出现 5 次
* `sumTotalTermFreq(fieldX) = 7`  对于 fieldX，5 个 A，1 个 B，1 个 C

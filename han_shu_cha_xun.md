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
| docfreq(field,val) | 返回这个字段里包含这个值的文档数量，这是个常数 | `docfreq(text,'solr')`<br>`...&defType=func&q=docfreq(text,$myterm)&myterm=solr` |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |
|  |  |  |



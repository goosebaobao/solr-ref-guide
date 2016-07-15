# 扩展的 DisMax 查询解析器

扩展的 DisMax（eDisMax）是一个 DisMax 的优化版本，除了支持 DisMax 的所有参数之外，eDisMax

 * 支持完整的 Lucene 查询语法
 * 支持 AND, OR, NOT, +, -
 * Lucene 语法模式下，and, or 被视同为 AND, OR
 * 支持魔法字段 `_val_`, `_query_`。这2个字段不是在 schema 里定义的真实的字段，但是如果使用的话，可以用来干一些特别的事：


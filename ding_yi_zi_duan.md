# 定义字段

字段在 `schema.xml` 的 `field` 元素里定义。定义好字段类型后，定义字段很简单。

## 例子

下面例子定义了一个名为 `price`，类型名为 `float`，默认值为 `0.0` 的字段，`indexed` 和 `stored` 属性明确设定为 `true`，同时，其他的属性都继承自 `float` 字段类型

```xml
<field name="price" type="float" default="0.0" indexed="true" stored="true"/>
```

## 字段属性

| 属性 | 描述 |
| name | 字段名。应该是字母数字下划线组成，不能以数字开头。名字前后都是下划线的(如 `_version_`)为保留字。每个字段都要有名字 |
| type | 这个字段的类型名。是在 `fieldType` 里定义的 `name`。每个字段都要有类型名 |
| default | 索引时，如果该字段没有值，一个默认值会自动添加到文档里的该字段。如果未指定则没有默认值 |

## 可选的字段类型覆盖属性


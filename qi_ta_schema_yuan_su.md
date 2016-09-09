# 其他 schema 元素

## Unique Key 唯一键

`uniqueKey` 元素指示文档的唯一标识是哪个字段。虽然 `uniqueKey` 不是必须的，但几乎是你的应用程序设计中总是要保证的。例如，如果想要更新索引里的文档应该用到 `uniqueKey` 

定义

```xml
<uniqueKey>id</uniqueKey>
```

`copyField` 不能用于 `uniqueKey` 字段。也不能用 `UUIDUpdateProcessorFactory` 自动生成 `uniqueKey` 值

如果字段是多值的或继承自多值字段类型，用作 `uniqueKey` 字段会导致操作失败。但是，`uniqueKey` 会继续工作，只要该字段被正确的使用。

## 默认的搜索字段 和 查询操作

## 相似性


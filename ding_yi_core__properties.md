# 定义 core.properties

core discovery(core 发现/感知)表示创建一个 core 就和磁盘上的 `core.properties` 一样简单(这 tmd 什么比喻...)。`core.properties` 文件是个简单的 JAVA 属性文件，每行是一个 k-v 对，例如 `name=core1`。注意不需要引号

一个最小化的 `core.properties` 文件看上去是这样的(但是，也可以是空的)

```ini
name=my_core_name
```

## core.properties 位置

在 `solr.home` 的子目录下放置名为 `core.properties` 的文件来配置 solr core。树的深度没有限制，可定义的 core 的数量也没有限制。core 可以在树的任何地方，但不能定义在一个已存在的 core 下。即，如下是不允许的

```bash
./cores/core1/core.properties
./cores/core1/coremore/core5/core.properties
```

这个例子里，将在 core1 停止枚举

如下是合法滴

```bash
./cores/somecores/core1/core.properties
./cores/somecores/core2/core.properties
./cores/othercores/core3/core.properties
./cores/extracores/deepertree/core4/core.properties
```




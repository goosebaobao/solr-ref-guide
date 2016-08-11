# 集合(collection) API

集合 api 让你能够创建，移除，重载集合，但是在 SolrCloud 环境，你也能用它创建指定数目的分片和副本的集合

## api 入口

下面的 api 都基于 url `http://<hostname>:<port>/solr`

`/admin/collections?action=CREATE`：创建集合

`/admin/collections?action=MODIFYCOLLECTION`：修改集合某些属性

`/admin/collections?action=RELOAD`：重载集合

`/admin/collections?action=SPLITSHARD`：将一个分片切分为 2 个新的分片

`/admin/collections?action=CREATESHARD`：创建一个新的分片

`/admin/collections?action=DELETESHARD`：删除一个不活动的分片

`/admin/collections?action=CREATEALIAS`：创建或修改集合的别名

`/admin/collections?action=DELETEALIAS`：删除集合别名

`/admin/collections?action=DELETE`：删除集合

`/admin/collections?action=DELETEREPLICA`：删除分片的一个副本

`/admin/collections?action=ADDREPLICA`：给分片添加一个副本

`/admin/collections?action=CLUSTERPROP`：添加/修改/删除一个集群级别的属性

`/admin/collections?action=MIGRATE`：迁移文档到另一个集合

`/admin/collections?action=ADDROLE`：添加一个特定的角色到集群的一个节点

`/admin/collections?action=REMOVEROLE`：移除一个已分配的角色

`/admin/collections?action=OVERSEERSTATUS`：获得观察者的状态和统计信息

`/admin/collections?action=CLUSTERSTATUS`：获得集群的状态

`/admin/collections?action=REQUESTSTATUS`：获取一个之前的异步请求的状态

`/admin/collections?action=DELETESTATUS`：删除之前的异步请求的已存储的状态

`/admin/collections?action=LIST`：列出所有集合

`/admin/collections?action=ADDREPLICAPROP`：添加一个属性到集合/分片/副本指定的副本

`/admin/collections?action=DELETEREPLICAPROP`：从集合/分片/副本指定的副本上删除一个属性

`/admin/collections?action=BALANCESHARDUNIQUE`：发布一个属性到集合的节点上的每一个分片

`/admin/collections?action=REBALANCELEADERS`：基于已分配的 'preferredLeader' 发布一个领导角色

`/admin/collections?action=FORCELEADER`：如果领导已遗失，强制发起一个领导选举

`/admin/collections?action=MIGRATESTATEFORMAT`：从已分片的 clusterstate.json 迁移集合到每个集合 state.json(?)
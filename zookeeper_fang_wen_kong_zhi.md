# ZooKeeper 访问控制

ACLs:ZooKeeper access control lists 

## 关于 ZooKeeper ACLs

SolrCloud 使用 ZooKeeper 来共享信息和协调

默认情况下，Solr 的 zk 节点是开放的，不安全的，不需认证的。本小节讲述了如何配置 Solr 来对其在 zk 上的创建的内容进行访问控制，及如何告诉 Solr 在访问这些内容时需要的认证信息。如果要在你的 zk 节点上使用 ACLs，你必须激活这个功能

变更 zk 上的 Solr 相关内容，可能会对 SolrCloud 集群造成损害。例如

* 变更配置可导致 Solr 失败或意料之外的行为
* 将集群状态信息错误的变更或不一致，可导致 SolrCloud 集群奇怪的行为
* 添加一个删除集群的工作在观察者(Overseer)上运行，可导致集群的数据被删除

你可能想要在 Solr 上开启 ZooKeeper ACLs，以授权你不信任的实体访问你的 zk，或降低恶意操作风险，例如

* 你系统上的恶意软件
* 使用同一个 zk 集群的其他应用程序

你可能想要限制读取操作，如果你觉得 zk 上有些内容不应对所有人开放

保护 ZooKeeper 本身意味着很大的不同。**本小节是关于保护 zk 上的 Solr 相关内容的**。zk 的内容基本上是在硬盘，部分在 zk 进程的内存里。**本小节也不是关于保护存储在硬盘上的 zk 数据或者 zk 进程级别的数据。**-这是 ZooKeeper 要对付的

这些内容也可以对于使用 zk api 的外来者也是可用的。外部进程可以连接到 zk 并创建，更新，删除，读取内容；例如，一个 SolrCloud 节点想要创建，更新，删除，读取，一个 SolrJ 客户端想要读取。

## 如何启用 ACLs

你想要做到

1. 控制 Solr 使用该证书来连接 zk，证书是用来获得 zk 上的操作许可的
2. 控制哪些 ACLs 可以被 Solr 添加到它创建的 zk 节点
3. 控制外部访问，以避免开启一个 Solr 节点时不得不修改或重编译它

Solr 节点，客户端，工具(如 ZkCLI)总是使用 java 类 SolrZkClient 来处理 zk 的工作。

### 控制证书

在 `solr.xml` 的 `<solrcloud>` 节使用 `zkCredentialsProvider` 属性来指定类名字或类路径以控制使用哪个证书提供者，类需要实现下面的接口

```java
package org.apache.solr.common.cloud;
public interface ZkCredentialsProvider {
  public class ZkCredentials {
    String scheme;
    byte[] auth;
    public ZkCredentials(String scheme, byte[] auth) {
      super();
      this.scheme = scheme;
      this.auth = auth;
    }
    String getScheme() {
      return scheme;
    }
    byte[] getAuth() {
      return auth;
    }
  }
  Collection<ZkCredentials> getCredentials();
}
```

Solr 调用给定的证书提供者的 `getCredentials()` 方法来决定使用哪个证书。如果未配置提供者，会使用默认的实现，`DefaultZkCredentialsProvider` 
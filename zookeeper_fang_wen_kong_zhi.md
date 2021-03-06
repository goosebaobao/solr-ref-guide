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

#### 开箱即用的实现

你可以自己实现，但 Solr 附带了 2 个实现

* `org.apache.solr.common.cloud.DefaultZkCredentialsProvider`: 它的 `getCredentials()` 方法返回一个空的 list，表示没有可用的证书。如果没有在 `solr.xml` 里配置提供者，默认就是这个
* `org.apache.solr.common.cloud.VMParamsSingleSetCredentialsDigestZkCredentialsProvider`: 它让你用系统属性来自定义证书。它支持最多一组证书
 * schema 为 "digest". 用户名和密码由系统变量 "`zkDigestUsername`" and "`zkDigestPassword`" 分别定义。如果用户名和密码都提供的话，这组证书通过 `getCredentials()` 方法返回。
 * 如果上述这组证书没有返回，将会使用默认的行为返回一个空列表，即 `DefaultZkCredentialsProvider`

### 控制 ACLs

在 `solr.xml` 的 `<solrcloud>` 节使用 `zkACLProvider` 属性来指定类名字或类路径以控制哪些 ACLs 将被添加，类需要实现下面的接口

```java
package org.apache.solr.common.cloud;

public interface ZkACLProvider {
  List<ACL> getACLsToAdd(String zNodePath);
}
```

当 Solr 要添加一个新的 zk 节点(znode)，调用给定的 acl 提供者的 `getACLsToAdd()` 方法来决定哪些 ACLs 添加到这个 znode。如果未配置提供者，使用默认的实现，`DefaultZkACLProvider`

#### 开箱即用的实现

你可以自己实现，但 Solr 附带了

* `org.apache.solr.common.cloud.DefaultZkACLProvider`: 返回只有一个元素的列表：`zNodePath`。这个唯一的 ACL 记录 是开放且不安全的。如果未在 `solr.xml` 里配置提供者，这就是默认值
* `org.apache.solr.common.cloud.VMParamsAllAndReadonlyDigestZkACLProvider`: 让你用户系统属性自定义 ACLs。它的 `getACLsToAdd()` 实现不使用 `zNodePath`, 所以所有的 znode 得到相同的 ACLs，可添加如下的选项
 * 可以做任何事的用户 
   * 权限为 "`ALL`" (相当于所有的 `CREATE`, `READ`, `WRITE`, `DELETE`, `ADMIN`)，且 schema 为 "digest"
   * 用户名和密码分别用系统属性 "`zkDigestUsername`" 和 "`zkDigestPassword`" 定义
   * 除非用户名和密码都提供，否则该 ACL 不会被添加到 ACLs 里
  * 只读的用户
   * 权限为 "`READ`"，且 schema 为 "digest". 
   * 用户名和密码分别用系统属性 "`zkDigestUsername`" 和 "`zkDigestPassword`" 定义
   * 除非用户名和密码都提供，否则该 ACL 不会被添加到 ACLs 里

    如果上述的 ACLs 都没有添加到列表，那么 `DefaultZkACLProvider` 的空 ACL 列表将作为默认值

注意系统属性名和证书提供者 `VMParamsSingleSetCredentialsDigestZkCredentialsProvider` 重名了。这是为了让这 2 个提供者以一种可能的方式协作：我们总是用 2 种限制来保护对内容的访问——一个管理员用户和一个只读的用户，同时我们总是用这个管理员用户连接到 zk，这样我们就可以在内容和 我们自己创建的 znode 上做任何事了

你可以将只读证书授予 SolrCloud 集群的客户，例如 SolrJ 客户端。它们将能够读取任何需要的内容，但不能修改任何内容。

### 变更 ACL Schemes

在你的 Solr 集群的运行生命期，也许想要从一个不安全的 zk 迁移到一个安全的 zk。变更 `solr.xml` 里的 `zkACLProvider` 配置能确保最新创建的节点是安全的，但不能保护已存在的数据。要修改所有存在的 ACLs，你可以用 `ZkCLI -cmd updateacls /zk-path`

只有你的 SolrCloud 集群停止时，才能变更 zk 里的 ACLs。强行在 Solr 运行时变更会导致状态不一致和一些节点不可达。要配置新的 ACLs，运行 ZkCli，附带如下 vm 属性：`-DzkACLProvider=...`， `-DzkCredentialsProvider=....`

* 证书提供者必须在当前节点上有管理员权限。如果省略，该操作将不使用证书(适用于不安全的配置)
* ACL 提供者将用来计算新的 ACLs。如果省略，该操作会将所有权限赋予任意用户，移除现有的安全控制

你可以用之前介绍过的 `VMParamsSingleSetCredentialsDigestZkCredentialsProvider` 和 `VMParamsAllAndReadonlyDigestZkACLProvider` 作为上面的属性

变更 zk ACLs后，确保 `solr.xml` 内容匹配。

### 例子

你想要保护 zk 里所有 Solr 相关的内容。你想要一个 'admin' 用户可以对 zk 里的内容做任何事，该用户被用于在 zk 初始化 Solr 内容，及服务端的 Solr 节点。你还想要一个 'readonly' 用户，只能读取 zk 里的内容，这个用户由客户端使用

下面的例子

* 'admin' 用户的用户名和密码为 admin-user/admin-password
* 'readonly' 用户的用户名和密码为 readonly-user/readonly-password

提供者的类名必须第一个在 `solr.xml` 里配置

```xml
...
<solrcloud>
...
  <str name="zkCredientialsProvider">org.apache.solr.common.cloud.VMParamsSingleSetCredentialsDigestZkCredentialsProvider</str>
  <str name="zkACLProvider">org.apache.solr.common.cloud.VMParamsAllAndReadonlyDigestZkACLProvider</str>
```
#### 使用 ZkCli

```shell
SOLR_ZK_CREDS_AND_ACLS="-DzkDigestUsername=admin-user
-DzkDigestPassword=admin-password \
-DzkDigestReadonlyUsername=readonly-user
-DzkDigestReadonlyPassword=readonly-password"

java ... $SOLR_ZK_CREDS_AND_ACLS ... org.apache.solr.cloud.ZkCLI -cmd ...
```

#### 要使用 bin/solr，添加下面内容到 bin/solr.in.sh 末尾

```shell
SOLR_ZK_CREDS_AND_ACLS="-DzkDigestUsername=admin-user
-DzkDigestPassword=admin-password \
-DzkDigestReadonlyUsername=readonly-user
-DzkDigestReadonlyPassword=readonly-password"

SOLR_OPTS="$SOLR_OPTS $SOLR_ZK_CREDS_AND_ACLS"
```

#### 要使用客户端(SolrJ)

```shell
SOLR_ZK_CREDS_AND_ACLS="-DzkDigestUsername=readonly-user
-DzkDigestPassword=readonly-password"

java ... $SOLR_ZK_CREDS_AND_ACLS ...
```

或许你使用自己开发的 SolrZkClient 客户端，你可以覆盖提供者的代码


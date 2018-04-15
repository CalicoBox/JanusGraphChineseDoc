# 16. Apache Cassandra

> 当需要可扩展性、高度可靠性和毫不妥协的性能时，Apache Cassandra 是您正确的选择。线性增长的可扩展性与在商业级硬件与云端设备上经过验证的容错性使之成为处理关键任务的数据的绝佳平台。Cassandra 对于跨多个数据中心备份的支持是同类数据库中最好的，Cassandra 为您的用户提供更低的延迟，并且可以在地区性停电事故安然存活。已知最大的 Cassandra 集群在超过 400 台机器中拥有超过 300 TB 的数据。
> -- [Apache Cassandra 主页](http://cassandra.apache.org/)

接下来的各小节概述了 JanusGraph 与 Apache Cassandra 共同使用的各种可行方式。

:notebook: **注意**
*如果你使用了 Cassandra 2.2 或者更高的版本，你需要明确允许"节省"(thrift)来使 JanusGraph 连接至集群。*
*执行下面的命令来开启 thrift:*
```
/bin/nodetool enablethrift
```

## 16.1. 本地服务器模式
![modes-local](http://docs.janusgraph.org/latest/images/modes-local.png)
Cassandra 可以作为一个独立的数据库，与 JanusGraph 和最终用户应用程序跑在同一台主机上。在这种模式下，JanusGraph 和 Cassandra 通过一个 `localhost` 接口彼此沟通。遵循如下步骤即可在 Cassandra 之上运行 JanusGraph:
1. [下载 Cassandra](http://cassandra.apache.org/download/) 并解压，并在 `conf/cassandra.yaml` 和 `conf/log4j-server.properties` 中设置文件系统路径。
2. 在 Cassandra 的解压路径下进行命令行调用 `bin/cassandra -f` 来启动 Cassandra。读取输出以检查 Cassandra 是否成功启动。
现在，你可以像下面这样创建一个 Cassandra JanusGraph 实例:

```java
JanusGraph g = JanusGraphFactory.build().
set("storage.backend","cassandra").
set("storage.hostname","127.0.0.1").
open();
```
在 Gremlin 控制台中，你无法定义变量 `conf` 和 `g`的类型。简单地放弃声明类型即可。

## 16.2. 远程服务器模式
![modes-distributed](http://docs.janusgraph.org/latest/images/modes-distributed.png)
当图表需要扩展到单个机器的范围之外时，Cassandra 和 JanusGraph 在逻辑上被划分为了不同的机器。在该模式下，Cassandra 集群负责维护图表的呈现，同时任意数量的 JanusGraph 实例对 Cassandra 进行基于套接字进行读/写操作。最终用户程序可以在 JanusGraph 运行的 JVM 虚拟机中直接与 JanusGraph 进行交互。

比如说，假设我们正在运行的 Cassandra 集群中有一台机器的 IP 地址为 77.77.77.77，那么连接JanusGraph和集群的过程如下: (用逗号分割 IP 地址来引用一台以上的机器)

```java
JanusGraph graph = JanusGraphFactory.build().
set("storage.backend", "cassandra").
set("storage.hostname", "77.77.77.77").
open();
```
在 Gremlin 控制台中，你无法定义变量 `conf` 和 `g`。简单地放弃声明类型即可。

## 16.3. 使用 Gremlin Server 的远程服务器
Gremlin 服务器可以包裹在任一如上一小节定义的 JanusGraph 之外。通过这种方式，最终用户程序不需要是基于 Java 的应用，因为应用此时可以直接作为客户端与 Gremlin 服务器进行交互。这种类型的部署对于多语言体系结构非常适用，在这种结构下，用不同语言编写的各种组件都需要在图上进行引用和计算。

调用 `/bin/gremlin-server.sh` 启动 Gremlin 服务器，并在你可以接下来通过一个外部 Gremlin 终端会话来发送远程 Gremlin 指令

```gremlin
:plugin use tinkerpop.server
:remote connect tinkerpop.server conf/remote.yaml
:> g.addV()
```

在这种情况下，每个 Gremlin 服务器都将配置为连接到 Cassandra 群集。下面显示了 Gremlin 服务器特定图片段的配置。有关如何配置服务器的完整示例和更多信息，请参阅 [7. JanusGraph Server](.\ii-janusgraph-basics\7-janusgraph-server.md)。

```
...
graphs: {
  g: conf/janusgraph-cassandra.properties}
plugins:
  - janusgraph.imports
...
```

更多关于 Gremlin 服务器的信息参见 [Apache TinkerPop documentation](http://tinkerpop.apache.org/docs/3.2.6/reference#gremlin-server)

## 16.4. JanusGraph 嵌入模式
![modes-embedded](http://docs.janusgraph.org/latest/images/modes-embedded.png)
最后，JanusGraph 还可以嵌入到 Cassandra 中，该模式意味着 JanusGraph 和 Cassandra 将会运行在同一个 JVM 上，通过进程调用而不是网络来进行交互。这种方式消除了(反)序列化和网络协议带来的开销并且可带来可观的性能提升。在这种部署模式下， JanusGraph 在内部开启一个 Cassandra 守护进程并且不会连接除了自身所在集群以外的其它任何可用集群。

要在嵌入模式下使用 JanusGraph，只需将存储后端设置为`embeddedcassandra`。 下一部分显示的参数选项对嵌入模式的 Cassandra 同样适用。创建 JanusGraph 集群时， 需要确保个体结点间彼此能通过 Gossip 协议互相发现， 除此之外， 创建一个 Janus-with-Cassandra-embedded 集群和创建一个单独的 Cassandra 集群几乎一样。 在以嵌入式模式运行 JanusGraph 时， Cassandra 需要使用额外的配置项 `storage.conf-file` 设置完整的 url 来指定 yaml 文件。比如 `storage.conf-file = file:///home/cassandra.yaml`。

建议您在运行一个包含嵌入 JanusGraph 的 Cassandra 的集群时，通过 Gremlin 服务器来发现 JanusGraph 服务， 这样，应用可以远程连接至 JanusGraph 数据库并且执行查询请求。

注意， 运行包含嵌入 JanusGraph 的 Cassandra 服务需要对 GC 调整。 虽然嵌入模式下的 Cassandra 查询应答更低，但是 GC 的行为将会更加不可预测。

## 16.5. Cassandra 的特殊配置
参阅 [13. 配置参考](./ii-janusgraph-basics/13-config-ref.md)，以获取包含 Cassandra 限定配置在内的所有的 JanusGraph 配置的完整列表。

在配置 Cassandra 时，建议考虑以下 Cassandra 特定的配置选项:

+ **read-consistency-level:** Cassandra 读操作的一致性级别。
+ **write-consistency-level:** Cassandra 写操作的一致性级别。
+ **replication-factor:** 要使用的复制因子。复制因子越高，机器故障时图数据库数据复制的代价越低。**在生产系统中覆盖(Override)复制因子的默认值以保证健壮性。推荐值是3。** 此复制因子只能在最初创建密钥空间时设置。若已经存在密钥空间，该值会被忽略。
+ **thrift.frame_size_mb:** 使用 thrift 传输帧的最大个数。在检索非常大的结果集时增加此值。**只在 `storage.backend=cassandrathrift` 时生效。
+ **keyspace:** 用于存储 JanusGraph 图的键空间的名称。同一个 Cassandra 集群中允许多个JanusGraph图形共存。

有关 Cassandra 一致性级别和可接受值的更多信息，请参阅 [Cassandra Thrift API](http://wiki.apache.org/cassandra/API10)。通常来说，越高级别的一致性越高，健壮性也越高，但是同时有更高的延迟。

## 16.6. 全局图操作
JanusGraph 在 Cassandra 上支持全局顶点和边迭代。但是，请注意，所有这些顶点和/或边将被加载到内存中，这可能会导致 `OutOfMemoryException`。使用 [35. JanusGraph with TinkPop's Hadoop-Gremlin](35-janusgraph-with-tinkpops-hadoop-gremlin) 来在大型图中高效遍历所有的顶点和边。

## 16.7. 在 Amazon EC2 上部署
> Amazon Elastic Compute Cloud (Amazon EC2) 是一款提供在云上弹性计算容量的 web 服务，它旨在让开发人员更轻松地进行网络计算。
> --[Amazon EC2](http://aws.amazon.com/ec2/)

按照以下步骤在 EC2 上设置 Cassandra 集群，并在 Cassandra 上部署 JanusGraph。在此之前，您需要一个建好身份认证凭证的 Amazon AWS 账号和一些 AWS 和 EC2 的基础知识。

### 16.7.1. 安装 Cassandra 集群
这些用于配置和启动 DataStax Cassandra Community Edition AMI 的说明基于 DataStax AMI Docs，并且专注于 JanusGraph  部署相关的方面。

### 16.7.2. 建立安全组

+ 导航至 EC2 控制台仪表板，然后点击 「Network & Security」 下的 「Security Groups」。
+ 创建一个新的安全组。点击进入。将「Create a new rule」下拉菜单设置为「Custom TCP rule」。从源 0.0.0.0/0. 添加端口 22 的规则。从安全组成员添加端口 1024 - 65535 的规则。如果您不希望在安全组成员中打开所有非特权端口，那么至少需要在安全组成员间打开 7000，7199 和 9160 端口。
> 小提示：一旦「sg」输入框中，「Source」下拉菜单将自动填充安全组标识符，因此您无需事先准备好准确的值。

### 16.7.3. 启动 DataStax Cassandra AMI
+ 在您希望的区域内启动 [DataStax AMI](https://aws.amazon.com/amis/datastax-auto-clustering-ami-2-2)
+ 在「Request Instances Wizard」的「Instance Details」页中，将「Number of Instances」设置为所需数量的 Cassandra 节点。将「Instance Type」设置为「at least m1.large」。推荐设置为「m1.large」。
+ 在「Advanced Instance Options」的「Advanced Instance Options」页中，设置「User Data」单选按钮为「as text」，然后在文本框中输入:
```
--clustername [cassandra-cluster-name]
--totalnodes [number-of-instances]
--version community
--opscenter no
```

在这个配置中的 `[number-of-instances]` 必须与之前在「Request Instances Wizard」中设置的 EC2 实例数量匹配。`[cassandra-cluster-name]` 用来作为识别符，可以是任意字符串。
比如:
```
--clustername janusgraph
--totalnodes 4
--version community
--opscenter no
```

+ 在「Request Instances Wizard」的「Tags」页上您可以应用任一希望的配置。这些标签仅仅存在于 EC2 的用户级别上，不会作用于 Cassandra 守护进程的配置或操作。
+ 在「Request Instances Wizard」的「Create Key Pair」页上，可以选择已有的密钥对或是创建一个新的密钥对。包含所选密钥对的私有一半的 PEM 文件将被要求连接到这些实例。
+ 在「Request Instances Wizard」的「Configure Firewall」页上选择先前创建的安全组。
+ 在最终向导页面上查看并启动实例。

### 16.7.4. 验证实例启动成功
+ SSH 到任一一个 Cassandra 实例结点: `ssh -i [your-private-key].pem ubuntu@[public-dns-name-of-any-cassandra-instance]`
+ 运行 Cassandra nodetool `nodetool -h 127.0.0.1 ring` 来检查 Cassandra 令牌环的状态。您应该在此命令的输出中看到与先前步骤中启动的实例相同数量的节点。

注意，AMI 需要花费数分钟去配置每一个结点。当 SSH 进入成功配置的实例时会出现 shell 提示符。

### 16.7.5. 启动 JanusGraph 实例
启动额外的 EC2 实例来运行 JanusGraph，如上文所述，这些实例既可以在远程服务器模式下配置，也可以在 Gremlin-Server 中配置为远程服务器模式。您只需记下其中一个 Cassandra 群集实例的 IP 地址并将其配置为主机名(host)。要运行特定的 EC2 实例和特定配置取决于您的用例。

### 16.7.6. 在 Amazon Linux AMI 上部署 JanusGraph 实例的示例
+ 在 Cassandra 集群的同一域中启动 [Amazon Linux AMI](http://aws.amazon.com/amazon-linux-ami)。根据您需要的资源选择您喜欢的 EC2 实例类型。使用默认配置，并选择和先前配置的 Cassandra 集群相同的密钥对和安全组。
+ 使用 `ssh -i [your-private-key] .pem ec2-user @ [public-dns-name-of-the-instance]` ssh 进入新创建的实例。您可能必须得等待一小会儿实例启动。
+ 使用 wget [下载](https://github.com/JanusGraph/janusgraph/releases) 当前 JanusGraph 的发行版，并将文档本地解压到主目录。启动 Gremlin 控制台以验证 JanusGraph 成功运行。有关如何解压 JanusGraph 并启动 Gremlin 控制台的更多信息，请参阅[入门指南](http://docs.janusgraph.org/latest/getting-started.html).
+ 使用 `vi janusgraph.properties` 创建配置文件并添加如下行:
  ```yaml
  storage.backend = cassandra
  storage.hostname = [IP-address-of-one-Cassandra-EC2-instance]
  ```

也许你需要添加此页面或 (13. Configuration Reference)[13-configuration-reference] 中额外的配置选项。

+ 再次启动 Gremlin Console 并且输入下面的内容:
  ```yaml
  gremlin> graph = JanusGraphFactory.open('janusgraph.properties')
  ==>janusgraph[cassandra:[IP-address-of-one-Cassandra-EC2-instance]]
  ```
您已成功将此 JanusGraph 实例连接到 Cassandra 集群，并可以开始在图上进行操作。
### 16.7.7. 从 EC2 外部连接到 EC2 中的 Cassandra 集群
由于 Cassandra 结点默认只在它们的 EC2 内部 IP 间，而不是公网 IP中广播，仅在安全组中开启 (9160, 7000, 7199) 端口是不够的。

由此产生的现象是，您可以通过连接任何 Cassandra 节点上的 9160 端口在集群上打开 JanusGraph 图表，但是对该图表的所有请求都会超时。这是因为 Cassandra 告诉客户端连接到一个无法访问的IP。

修复方法是，为每个实例在 /etc/cassandra/cassandra.yaml 中设置「boardcast-address」属性， 然后重启实例。为集群每个实例重复这个步骤。一旦集群回应，nodetool 会报告允许来自本地机器的连接的正确的公网 IP。

修改 「boardcast-address」属性使您可以从 EC2 外部访问集群，但这也可能意味着 EC2 内产生的流量必须往返于互联网并在其获得集群前返回。所以，这种方法通常只对开发和测试有用。

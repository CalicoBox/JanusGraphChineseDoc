# 17. Apache HBase
> Apache HBase 是依据 [Google’s Bigtable: A Distributed Storage System for Structured Data by Chang et al.][bigtable]模型建立的一款开源、分布式的、版本化的非关系型数据库。Apache HBase 在 Hadoop 和 HDFS 之上提供了类似 Bigtable 那样利用 Google 文件系统提供的分布式数据存储的功能。
> -- [Apache HBase Homepage][apache-hbase]

## 17.1. 安装 HBase
以下各节概述了 JanusGraph 与 Apache HBase 配合使用的各种可行方式。

### 17.1.1. 本地服务器模式
运行 HBase 之上的 JanusGraph 需要遵循如下安装步骤：
![modes-local](http://docs.janusgraph.org/latest/images/modes-local.png)
HBase 可以作为独立的数据库与 JanusGraph 和用户端应用程序在同一本地主机上运行。在这种模式下，JanusGraph 和 HBase 通过一个 `localhost` 的 socket 彼此交互。在HBase上运行JanusGraph需要以下设置步骤：

+ 从 [http://www.apache.org/dyn/closer.cgi/hbase/stable/](http://www.apache.org/dyn/closer.cgi/hbase/stable/) 下载并且解压稳定版的 HBase。
+ 调用解压后的 HBase 的 bin 目录下 `start-hbase.sh` 脚本以启动 HBase 。如要停用 HBase，使用 `stop-hbase.sh`

```bash
$ ./bin/start-hbase.sh
starting master, logging to ../logs/hbase-master-machine-name.local.out
```

现在，你可以如下所示创建 HBase JanusGraph：

```java
JanusGraph graph = JanusGraphFactory.build()
	.set("storage.backend", "hbase")
	.open();
```

Note, that you do not need to specify a hostname since a localhost connection is attempted by default. Also, in the Gremlin Console, you can not define the type of the variables conf and g. Therefore, simply leave off the type declaration.
注意，你并不需要指定一个主机名，在缺省时会默认尝试进行本地连接。此外，你不能在 Gremlin 终端中定义 `conf` 和 `g` 变量的类型。简单地放弃声明类型即可。

### 17.1.2. 远程服务器模式
![modes-distributed](http://docs.janusgraph.org/latest/images/modes-distributed.png)
当图需要扩展到单个机器的范围之外时，HBase 和 JanusGraph 在逻辑上被分离到不同的机器中。在这个模型中，HBase 集群承担图表的呈现，同时任意数量的所有 JanusGraph 实例承担对  HBase 集群的基于 socket 的读/写访问。用户端程序可以在 JanusGraph 所运行的 JVM 上直接和 JanusGraph 进行交互。

比如，假设我们有一个 ZooKeeper 限制为三台机器组成的 HBase 集群，其 IP 地址分别是 77.77.77.77, 77.77.77.78, 77.77.77.79，将集群和 JanusGraph 连接的方式如下：
```java
JanusGraph g = JanusGraphFactory.build()
	.set("storage.backend", "hbase")
	.set("storage.hostname", "77.77.77.77, 77.77.77.78, 77.77.77.79")
	.open();
```

`storage.hostname` 接受 JanusGraph 应该连接到的 HBase 集群中的任一的机器(子集)的主机名和 IP 地址列表，并以逗号分隔。此外，你不能在 Gremlin 终端中定义 `conf` 和 `g` 变量的类型。简单地放弃声明类型即可。

### 17.1.3. 使用 Gremlin Server 的远程服务器
![modes-rexster](http://docs.janusgraph.org/latest/images/modes-rexster.png)
最后，前面小节中提到任一 JanusGraph 实例都可以使用 Gremlin Server 包装起来。通过使用这种应用作为客户端与 Gremlin Server 交互的方式，用户端程序可以不再限于 Java 应用。对于需要使用不同语言编写的各种组件，在图上进行引用和计算的类型，这种多语言体系结构的部署非常适用。

```url
http://gremlin-server.janusgraph.machine1/mygraph/vertices/1
http://gremlin-server.janusgraph.machine2/mygraph/tp/gremlin?script=g.v(1).out('follows').out('created')
```

该模式下，每个 Gremlin 服务器都将被配置为连接到 HBase 群集。下面展示了 Gremlin 服务器上配置图的特殊配置。有关如何配置服务器的完整示例和更多信息，请参阅 [7. JanusGraph Server][7]。

```
...
graphs: {
  g: conf/janusgraph-hbase.properties}
plugins:
  - janusgraph.imports
...
```

## 17.2. HBase 特殊配置
参阅 [13. 配置参考][13]，以获取包含所有 HBase 限定配置在内的全部 JanusGraph 配置的完整列表。

在配置 HBase 时推荐遵循下面的 HBase 特殊配置选项：

+ **tablename**: 存储 JanusGraph 的 HBase 表名。一个 HBase 集群中允许多个 JanusGraph 同时存在。

更多 HBase 的配置项及其描述请参阅 [HBase 配置文档][apache-hbase-config]。配置 JanusGraph 时，在 HBase 配置项前加上 `storage.hbase.ext`，可以在初始化时将配置项传给 HBase。比如，设置属性 `storage.hbase.ext.zookeeper.znode.parent=/hbase-secure` 来设置 HBase 的 znode/hbase-secure。通过前缀使得 JanusGraph 可以配置任意 HBase 配置项。

## 17.3. 全局图操作
运行在 HBase 之上的 JanusGraph 支持顶点和边全局迭代。尽管如此，请注意将所有顶点和/或边载入内存可能会导致 `OutOfMemoryException`。 使用 [35. JanusGraph with TinkPop's Hadoop-Gremlin][35] 来在大型图中高效遍历所有的顶点和边。

## 17.4. 在 Amazon EC2 上部署
> Amazon Elastic Compute Cloud (Amazon EC2) 是一款提供在云上弹性计算容量的 web 服务，它旨在让开发人员更轻松地进行网络计算。
> --[Amazon EC2][amazon-ec2]

按照以下步骤在 EC2 上安装 HBase 集群，并在 HBase 上部署 JanusGraph。在此之前，您需要一个建好身份认证凭证的 Amazon AWS 账号和一些 AWS 和 EC2 的基础知识。

下面的命令首先通过 [Whirr][whirr] 在 EC2 上启动拥有四个结点的 HBase 集群， 然后使用这个集群运行一个基础的 JanusGraph 测试。

下面介绍的配置使一个 HBase 主服务器负责三个 HBase 区域服务器。Zookeeper 中 JanusGraph 连接到 HBase 的法定人数的唯一成员是主服务器。

Whirr 0.7.1 运行在 NAT WHIRR-459 的机器上时偶尔会失败。因此，推荐至少使用 Whirr 0.7.2。在一台运行 Amazon Linux 2012.03 的 t1.micro 实例上使用 Whirr 0.8.0 测试过下面的命令。运行环境为运行 Amazon Linux 2012.03 的 t1.micro 实例时，为了产生预期的结果，这些命令可能需要进行一定的调整。

```bash
# These commands were executed on a t1.micro instance running Amazon Linux 2012.03 x86_64.
# The AMI identifier for Amazon Linux 2012.03 x86_64 is ami-aecd60c7.
# https://console.aws.amazon.com/ec2/home?region=us-east-1#launchAmi=ami-aecd60c7
export AWS_ACCESS_KEY_ID=... # Set your Access Key here
export AWS_SECRET_ACCESS_KEY=... # Set your Secret Key here
curl -O http://www.apache.org/dist/whirr/whirr-0.8.0/whirr-0.8.0.tar.gz
tar -xzf whirr-0.8.0.tar.gz && cd whirr-0.8.0
# Generate an SSH keypair with which Whirr will deploy and manage instances
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa_whirr
# Download a Whirr recipe for deploying HBase 0.94.1 with hadoop-core 1.0.3
pushd recipes && wget 'https://raw.github.com/JanusGraph/janusgraph/master/config/whirr-hbase.properties' ; popd
bin/whirr launch-cluster --config recipes/whirr-hbase.properties --private-key-file ~/.ssh/id_rsa_whirr
# Run a superficial health check on the hbase-master node (this should print "imok")
echo "ruok" | nc $(awk '{print $3}' ~/.whirr/hbase-testing/instances | head -1) 2181; echo
# Login to the HBase master node to run the remaining commands
ssh -i ~/.ssh/id_rsa_whirr -o "UserKnownHostsFile /dev/null" \
      -o StrictHostKeyChecking=no \
      `grep hbase-master ~/.whirr/hbase-testing/instances \
      | awk '{print $3}'`
# Maven 2 is available through the package manager, but an incompatibility
# with surefire 2.12 makes it a pain to use; here we download Maven 3 without
# the OS package manager
wget 'http://archive.apache.org/dist/maven/maven-3/3.0.4/binaries/apache-maven-3.0.4-bin.tar.gz'
tar -xzf apache-maven-3.0.4-bin.tar.gz
# Install git
sudo apt-get install -y git-core
# Clone JanusGraph
git clone 'git://github.com/JanusGraph/janusgraph.git' && cd janusgraph
# Run a HBase-backed test of JanusGraph
#
# This test should produce pages of output ending in something like this:
#
# -------------------------------------------------------
#  T E S T S
# -------------------------------------------------------
# Running org.janusgraph.graphdb.hbase.ExternalHBaseGraphPerformanceTest
# Starting trial 1/1
# 10000
# 20000
# 30000
# 40000
# 50000
# Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 303.659 sec
#
# Results :
#
# Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
#
# [INFO] ------------------------------------------------------------------------
# [INFO] BUILD SUCCESS
# [INFO] ------------------------------------------------------------------------
~/apache-maven-3.0.4/bin/mvn test -Dtest=ExternalHBaseGraphPerformanceTest#unlabeledEdgeInsertion
# Check on hadoop
hadoop version # Should print 1.0.3
# List the hadoop root; should print something like:
#
# Found 4 items
# drwxr-xr-x   - hadoop supergroup          0 2012-09-20 00:20 /hadoop
# drwxr-xr-x   - hadoop supergroup          0 2012-09-20 00:42 /hbase
# drwxrwxrwx   - hadoop supergroup          0 2012-09-20 00:20 /tmp
# drwxrwxrwx   - hadoop supergroup          0 2012-09-20 00:20 /user
hadoop fs -ls /
```

## 17.5. 管理 HBase 集群的小技巧
主服务器上的 [HBase shell](habse-shell) 可用于获取
群集的整体状态检查。

```bash
$HBASE_HOME/bin/hbase shell
```

通常以下 shell 命令对了解群集的状态很有用。

```bash
status 'janusgraph'
status 'simple'
status 'detailed'
```

上述命令可以识别区域服务器是否已关闭。如果已关闭，可以将 `ssh` 进入故障的 区域服务器机器并执行以下操作：

```bash
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh stop regionserver
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh start regionserver
```

使用 [pssh][pssh] 可以简化步骤，因为没有必要一个个地登入每台机器机器以执行这些命令。将区域服务器的 IP 地址放进 `host.txt` 文件中然后执行下面的命令：

```bash
pssh -h host.txt sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh stop regionserver
pssh -h host.txt sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh start regionserver
```

然后，有时候你需要重启主服务器 (例如，连接被拒绝异常)。此时你需要执行下面的命令：

```
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh stop master
sudo -u hadoop $HBASE_HOME/bin/hbase-daemon.sh start master
```

最后，如果一个 HBase 集群已经被部署， 并且主服务器或域服务器需要更多的内存，只要对应的服务器的 `$HBASE_HOME/conf/hbase-env.sh` 文件中编辑必需的 `-Xmx -Xms` 参数即可。编辑后，参照前面所述，停止/启动主服务器和(或)区域服务器。


[apache-hbase]:http://hbase.apache.org/
[apache-hbase-config]:(http://hbase.apache.org/book/config.files.html)
[bigtable]:http://static.googleusercontent.com/media/research.google.com/en/us/archive/bigtable-osdi06.pdf
[hbase-shell]:http://wiki.apache.org/hadoop/Hbase/Shell
[pssh]:http://code.google.com/p/parallel-ssh/
[whirr]:http://whirr.apache.org/
[7]:./ii-janusgraph-basics/7-janusgraph-server.md
[13]:./ii-janusgraph-basics/13-config-ref.md
[35]:./v-advanced-topics/35-janusgraph-with-tinkpops-hadoop-gremlin

# 4.Configuration

一个JanusGraph图形数据库集群包含一个或多个JanusGraph实例。要打开一个JanusGraph实例，必须提供一个配置，该配置指定如何设置JanusGraph。

JanusGraph配置指定了JanusGraph应该使用哪些组件，控制了JanusGraph部署的所有操作方面，并提供了一些调优选项以使JanusGraph集群获得最大性能。

至少，JanusGraph配置必须定义JanusGraph作为存储后端使用的数据库。

“三、后端存储”

列出了所有支持的数据库以及如何分别配置它们。如果高级图查询支持\(例：全文搜索、geo搜索或范围查询\)需要一个额外的后端索引的支持（例如：Apache Solr）。详见

“四、后端索引”

。如果查询性能是一个问题，那么应该启用缓存。缓存配置和调优在

“11.JanusGraph缓存”

中进行了描述。

实例配置

下面是一些示例配置文件，以演示如何配置最常用的后端存储、索引系统和性能组件。这只覆盖了可用配置选项的一小部分。参考

“13.配置参考列表”

。

Cassandra+Elasticsearch

设置JanusGraph使用本地的Cassandra和远程的ElasticSearch。

storage.backend=cassandra storage.hostname=localhost index.search.backend=elasticsearch index.search.hostname=100.100.101.1, 100.100.101.2 index.search.elasticsearch.client-only=true

HBase+Caching

设置使用远程的HBase并使用JanusGraph缓存组件提升性能。

storage.backend=hbase storage.hostname=100.100.101.1 storage.port=2181 cache.db-cache = true cache.db-cache-clean-wait = 20 cache.db-cache-time = 180000 cache.db-cache-size = 0.5

BerkeleyDB【没使用】

设置使用BerkeleyDB和ElasticSearch并指定相关文件存储位置。

storage.backend=berkeleyje storage.directory=/tmp/graph index.search.backend=elasticsearch index.search.directory=/tmp/searchindex index.search.elasticsearch.client-only=false index.search.elasticsearch.local-mode=true

“13.配置参考列表”

详细描述了所有这些配置选项。JanusGraph的conf目录包含额外的配置示例。

进一步的实例

在conf/目录中有几个示例配置文件，可以用来快速启动JanusGraph。这些文件的路径可以传递给JanusGraphFactory.open\(…\)，如下所示:

// Connect to Cassandra on localhost using a default configuration

 graph = JanusGraphFactory.open\("conf/janusgraph-cassandra.properties"\) 

// Connect to HBase on localhost using a default configuration

 graph = JanusGraphFactory.open\("conf/janusgraph-hbase.properties"\)

使用配置文件

JanusGraphFactory

* 在Gremlin Console中使用JanusGraphFactory

JanusGraph的部件中包含一个命令行Gremlin控制台，它使得启动JanusGraph和与JanusGraph交互很容易。调用bin/gremlin.sh打开控制台，然后使用JanusGraphFactory通过可访问的配置文件打开JanusGraph图:

graph = JanusGraphFactory.open\('path/to/configuration.properties'\)

* 在代码中使用JanusGraphFactory

JanusGraphFactory还可以用于在基于jvm的用户应用程序中打开嵌入式JanusGraph图形实例。在这种情况下，JanusGraph是用户应用程序的一部分，应用程序可以通过它的公共API直接调用JanusGraph。

* 命令简化

如果JanusGraph图集群已经配置好了，并且/或仅需要定义后端存储，JanusGraphFactory将只需要接受后端存储名称和主机名或目录（使用“:”分隔字符串）。

graph = JanusGraphFactory.open\('cassandra:localhost'\)

graph = JanusGraphFactory.open\('berkeleyje:/tmp/graph'\)

JanusGraph Server

JanusGraph本身是一组没有执行线程的Jar文件。有两种基本的连接并使用JanusGraph的方法：

1. JanusGraph可以通过嵌入到拥有执行线程的客户端程序中进行调用。
2. JanusGraph打包了一个能持续运行的服务端程序，当程序启动时，允许通过远程客户端连接或单独的程序程序中连接来调用JanusGraph。这个可持续运行的服务端程序就是JanusGraph Server。

JanusGraph Server使用ThinkerpPop技术体系中的Gremlin Server来与客户端交互。JanusGraph提供一个可直接使用的配置文件来快速启动JanusGraph Server，并且可以通过修改配置来提供更多的功能。

配置JanusGraph Server是通过./conf/gremlin-server/中相关的yaml配置文件完成的。要配置带有一个JanusGraph实例的JanusGraph Server，需要以下设置：

... graphs: { graph: conf/janusgraph-berkeleyje.properties } plugins: - janusgraph.imports ...

“graphs”项指定了生成JanusGraph实例的配置文件。上面例子中，指定的配置文件是conf/janusgraph-berkeleyje.properties。

“plugins”项支持指定JanusGraph Gremlin插件。它允许自动导入JanusGraph类，以便在远程提交的脚本中引用它们。

在

“7.JanusGraph Server”

中了解更多关于JanusGraph Server的知识。

服务器文件位置

JanusGraph包含一个快速启动服务器组件，可以帮助您更容易地开始使用Gremlin服务器和JanusGraph。调用bin/janusgraph.sh启动Gremlin Server和Cassandra和ElasticSearch。

\[注：出于安全原因，ElasticSearch 和 janusgraph.sh必须在非root帐户下运行。\]

全局配置

JanusGraph将配置分为本地和全局配置选项。本地配置选项适用于单个JanusGraph实例。全局配置选项适用于集群中的所有实例。更具体地说，JanusGraph有以下5种配置选项：

LOCAL:这些选项只适用于单个的JanusGraph实例，并在初始化JanusGraph实例时提供的配置中指定。

MASKABLE:可以将这些配置选项覆盖到本地配置文件中的单个JanusGraph实例。如果本地配置文件没有指定选项，则从全局JanusGraph集群配置读取其值。

GLOBAL:这些选项总是从集群配置中读取，不能在实例基础上覆盖。

GLOBAL\_OFFLINE:和全局一样，但是更改这些选项需要重新启动集群，以确保整个集群的值相同。

FIXED:像全局变量一样，但是一旦JanusGraph集群被初始化，值就无法更改。

当集群中的第一个JanusGraph实例启动时，从提供的本地配置文件中初始化全局配置选项。

随后更改全局配置选项是通过JanusGraph的管理API完成的。

要访问管理API，要在可用的JanusGraph实例g上调用g.getManagementSystem\(\)，例如，在JanusGraph集群中更改默认的缓存行为:

mgmt = graph.openManagement\(\) mgmt.get\('cache.db-cache'\) 

// Prints the current config setting

 mgmt.set\('cache.db-cache', true\) 

// Changes option

 mgmt.get\('cache.db-cache'\) 

// Prints 'true'

 mgmt.commit\(\) 

// Changes take effect

修改GLOBAL\_OFFLINE配置

更改的配置选项不影响正在运行的实例，只适用于新启动的实例。更改GLOBAL\_OFFLINE配置选项需要重新启动集群，以便对所有实例立即生效。

要改变GLOBAL\_OFFLINE选项，请遵循以下步骤：

* 在集群中留下一个实例，关闭其他所有的JanusGraph实例。
* 连接到单个实例。
* 确保所有正在运行的事务都已关闭。
* 确保不启动新的事务\(即集群必须脱机\)
* 打开管理API
* 改变配置选项\(s\)
* 调用commit，它将自动关闭实例。
* 重启所有实例

请参考

“13.配置参考列表”

，以获取更多信息，包括每个选项的配置范围。


# 配置

一个JanusGraph图形数据库集群包含一个或多个JanusGraph实例。要打开一个JanusGraph实例，必须提供一个配置，该配置指定如何设置JanusGraph。

JanusGraph配置指定了JanusGraph应该使用哪些组件，控制了JanusGraph部署的所有操作方面，并提供了一些调优选项以使JanusGraph集群获得最大性能。

至少，JanusGraph配置必须定义JanusGraph作为存储后端使用的数据库。“第三节 后端存储”列出了所有支持的数据库以及如何分别配置它们。如果高级图查询支持\(例：全文搜索、geo搜索或范围查询\)需要一个额外的后端索引的支持（例如：Apache Solr）。详见“第四节 后端索引”。如果查询性能是一个问题，那么应该启用缓存。缓存配置和调优在“第十一节 JanusGraph缓存”中进行了描述。

实例配置

下面是一些示例配置文件，以演示如何配置最常用的后端存储、索引系统和性能组件。这只覆盖了可用配置选项的一小部分。参考

“第十三节 配置参考列表”

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

“第十三节 配置参考列表”

详细描述了所有这些配置选项。JanusGraph的conf目录包含额外的配置示例。

进一步的实例

在conf/目录中有几个示例配置文件，可以用来快速启动JanusGraph。这些文件的路径可以传递给JanusGraphFactory.open\(…\)，如下所示:

// Connect to Cassandra on localhost using a default configuration

graph = JanusGraphFactory.open\("conf/janusgraph-cassandra.properties"\)

// Connect to HBase on localhost using a default configuration

graph = JanusGraphFactory.open\("conf/janusgraph-hbase.properties"\)


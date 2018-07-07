# 26. Apache Solr

> Solr是基于 Apache Lucene 构建的流行，快速，开源的企业搜索平台。Solr 是一款使用 REST-like 的 API 的独立的企业搜索服务器。Solr 具有高可靠性，高可扩展性和高容错性的特点，可提供分布式索引，复制和负载均衡查询，自动故障转移和恢复，集中配置等功能。
> -- [Apache Solr Homepage](http://lucene.apache.org/solr/)

JanusGraph 支持以 [Apache Solr](http://lucene.apache.org/solr/) 作为存储后端。下面是一些 JanusGraph 支持的 Solr 特性。
+ **全文**：支持所有搜索给定单词，前缀或正则表达式匹配的文本属性的 `Text` 谓词。
+ **Geo**：支持所有搜索给定查询几何体相交，相交或包含的地理属性的 `Geo` 谓词。支持点，圈，框，线和多边形索引。支持查询点属性的圆形，方框和多边形以及查询非点属性的所有形状。请注意，使用线形和多边形时需要 JTS (参阅 [Geoshape 文档](https://docs.janusgraph.org/latest/search-predicates.html#geoshape)以获取更多信息)。
+ **数字范围**:  支持 `Compare` 中所有的数字比较符。
+ **灵活的配置**： 支持远程操作和开放式自定义化设置。
+ **TTL**： 支持索引元素自动过期。
+ **集合**： 支持索引 SET 和 LIST 基数属性。
+ **瞬时**：纳秒粒度的时间索引。
+ **自定义分析器**： 选择使用自定义分析器。

请参阅 [附录2. 版本适配](https://docs.janusgraph.org/latest/version-compat.html) 以获取 ES 与 JanusGraph 一起使用的版本信息。

## 26.1. Solr Configuration Overview
JanusGraph 支持既支持 SolrCloud 也支持 Solr Standalone（HTTP）配置来运行 Solr 以便使用**混合索引**（请参见[第 9.1.2 节 「混合索引」](https://docs.janusgraph.org/latest/indexes.html#index-mixed)）。推荐的连接模式是通过参数 `mode` (只被设置为 `cloud` 或 `http`，默认为前者) 来设置。 例如，在 JanusGraph 配置属性中指定以下属性，来显式指定 Solr 使用 SolrCloud 配置运行：
```
index.search.solr.mode=cloud
```

下面是一些关键的 Solr 术语：
+ **核心 Core**: 单个机器中的*单个索引*。
+ **配置 Configuration**: *solrconfig.xml*, *schema.xml*, 等其它定义一个核心所需要的配置。
+ **集合 Collection**: 一个*单独的逻辑索引*，可跨域不同计算机上多个核心。
+ **配置集 Configset**: 多个核心可复用的共享*配置*。

## 26.2. 连接到 SolrCloud
在设置 `mode` 为 `cloud` 连接到 SolrCloud 集群时，必须指定 Zookeeper URL (和可能的端口) 以使 JanusGraph 可以发现集群并与之进行交互。

```scala
index.search.backend=solr
index.search.solr.mode=cloud
index.search.solr.zookeeper-url=localhost:2181
```

可对与创建新集合有关的多个其他配置选项进行配置（仅在 SolrCloud 操作模式下支持）以控制分片等行为。请参阅[第13章，配置参考](https://docs.janusgraph.org/latest/config-ref.html)获取有关这些选项的完整列表。

Zookeeper 在 SolrCloud 协调 Solr 服务器之间的集合和配置集信息时起到杠杆的作用。Zookeeper 和 SolrCloud 一起使用可以显着减少使用 Solr 作为 JanusGraph 的后端索引所需的手动配置的量。

### 26.2.1. 配置集配置
配置集被用于创建新的集合。配置集存放在 Zookeeper 中，允许跨 Solr 服务器进行访问。
+ 在创建时每一个集合都可以提供它自己的创建集，所以每个集合的配置可能会有所不同。使用这种方法，每个集合都必须手动来建立。
+ 如果多个集合重用配置集，可以将共享配置集单独上载到 Zookeeper。使用这种方法， JanusGraph 可以通过使用共享配置集自动创建集合。另外还有一个优点是，复用配置集显著降低了 Zookeeper 的数据储存总量。

#### 26.2.1.1. 使用单个配置集
在本例子中，用 JanusGraph 用于的分布式系统中的 Solr 的默认配置，来手工创建了名为 `verticesByAge` 的集合。一旦该集合创建，其与集合 `verticesByAge` 同名的配置集会被上传到 Zookeeper。请参阅[Solr Reference Guide](https://lucene.apache.org/solr/guide/6_6/solr-control-script-reference.html#SolrControlScriptReference-CollectionsandCores)获取可用参数。

```shell
# create the collection
$SOLR_HOME/bin/solr create -c verticesByAge -d $JANUSGRAPH_HOME/conf/solr
```

使用 JanusGraphManagement 和同名集合定义混合索引
```java
mgmt = graph.openManagement()
age = mgmt.makePropertyKey("age").dataType(Integer.class).make()
mgmt.buildIndex("verticesByAge", Vertex.class).addKey(age).buildMixedIndex("search")
mgmt.commit()
```
#### 26.2.1.2. 使用共享配置集
使用共享配置集时，将配置上载作为一次性操作首先进行是最为方便的。在下例中，将一个用该分布式系统中 Solr 的默认 JanusGraph 配置 `janusgraph-configset` 上传至 Zookeeper 中。请参阅[Solr Reference Guide](https://lucene.apache.org/solr/guide/6_6/solr-control-script-reference.html#SolrControlScriptReference-CollectionsandCores)获取可用参数。
```bash
# upload the shared configset into Zookeeper
# Solr 5
$SOLR_HOME/server/scripts/cloud-scripts/zkcli.sh -cmd upconfig -z localhost:2181 \
    -d $JANUSGRAPH_HOME/conf/solr -n janusgraph-configset
# Solr 6 and higher
$SOLR_HOME/bin/solr zk upconfig -d $JANUSGRAPH_HOME/conf/solr -n janusgraph-configset \
    -z localhost:2181
```

在 JanusGraph 上配置 SolrCloud 索引后端时，一定要用 `index.search.solr.configset` 属性来提供共享配置集的名字。

```bash
index.search.backend=solr
index.search.solr.mode=cloud
index.search.solr.zookeeper-url=localhost:2181
index.search.solr.configset=janusgraph-configset
```

使用 JanusGraphManagement 和同名集合定义混合索引
```java
mgmt = graph.openManagement()
age = mgmt.makePropertyKey("age").dataType(Integer.class).make()
mgmt.buildIndex("verticesByAge", Vertex.class).addKey(age).buildMixedIndex("search")
mgmt.commit()
```

## 26.3 连接到 Solr Standalone (HTTP)
当设置 `mode` 为 `http` 连接到 Solr Standalone 时，必须提供一个或多个 Solr 实例的 URLs。
```scala
index.search.backend=solr
index.search.solr.mode=http
index.search.solr.http-urls=http://localhost:8983/solr
```

HTTP 模式还有用于控制最大连接数，连接超时和传输压缩的等配置选项可用。完整的选项列表请参阅 [第十三章. 参考配置](https://docs.janusgraph.org/latest/config-ref.html)。

### 26.3.1 核心配置
Solr Standalone 用于单个实例，它将配置信息保存在文件系统中。每个混合索引的核心必须手动创立。

创建核心需要一个 `core_name` 和一个 `configuration` 字典。请参阅[Solr Reference Guide](https://lucene.apache.org/solr/guide/6_6/solr-control-script-reference.html#SolrControlScriptReference-CollectionsandCores)获取可用参数。下面例子使用分布式系统中的 Solr 的默认 JanusGraph 配置创建了名为 `verticesByAge` 的核心

```scala
$SOLR_HOME/bin/solr create -c verticesByAge -d $JANUSGRAPH_HOME/conf/solr
```

使用 JanusGraphManagement 和同一核心名定义混合索引

```python
mgmt = graph.openManagement()
age = mgmt.makePropertyKey("age").dataType(Integer.class).make()
mgmt.buildIndex("verticesByAge", Vertex.class).addKey(age).buildMixedIndex("search")
mgmt.commit()
```

## 26.4 Solr 模式设计
### 26.4.1 动态字段定义
JanusGraph 默认使用 Solr 的 [Dynamic Fields](https://cwiki.apache.org/confluence/display/solr/Dynamic+Fields) 来定义所有索引键的字段类型。属性键添加到 Solr 支持的混合索引时，这不需要额外的配置，并且提供比无模式模式(Schemaless mode)更好的性能。

JanusGraph 假设在后端 Solr 集合的 schema.xml 文件中定义了如下动态字段标记。请注意，Solr 的 schema.xml 文件中还需要以下这些附加的 xml 定义才能够使用。相关详细信息请参阅 JanusGraph 安装时 ./conf/solr/schema.xml 目录中提供的示例 schema.xml 文件。
```xml
<dynamicField name="*_i"    type="int"          indexed="true"  stored="true"/>
<dynamicField name="*_s"    type="string"       indexed="true"  stored="true" />
<dynamicField name="*_l"    type="long"         indexed="true"  stored="true"/>
<dynamicField name="*_t"    type="text_general" indexed="true"  stored="true"/>
<dynamicField name="*_b"    type="boolean"      indexed="true" stored="true"/>
<dynamicField name="*_f"    type="float"        indexed="true"  stored="true"/>
<dynamicField name="*_d"    type="double"       indexed="true"  stored="true"/>
<dynamicField name="*_g"    type="geo"          indexed="true"  stored="true"/>
<dynamicField name="*_dt"   type="date"         indexed="true"  stored="true"/>
<dynamicField name="*_uuid" type="uuid"         indexed="true"  stored="true"/>
```

利用Solr的动态字段功能，在 JanusGraph 的默认配置中，属性键名称不必以类型相应的后缀结束。JanusGraph 通过对属性键定义的数字标识符和类型相应的后缀进行编码，从属性键名生成 Solr 字段名称。这意味着 JanusGraph 在后台使用带有类型相应后缀的合成的字段名，而不管 JanusGraph 应用里定义和使用的属性键的名称。可以通过非默认配置来覆盖此字段名称的映射，在下一节中将进行描述。

### 26.4.2. 手动字段定义
如果用户更喜欢手动定义集合中的每个索引字段，那么需要禁用配置选项 `dyn-fields`。在将属性键添加到索引之前，必须保证后端每一个 Solr 模式的属性键已经定义。

使用此方案时，建议在名称到字段映射启用显式属性键，用显式定义以便修复字段名。这可以通过以下两种方式之一实现：

1. 向索引添加属性键时提供 `mapped-name` 参数，以设置字段名。查看 [23.1. 单个字段映射](https://docs.janusgraph.org/latest/field-mapping.html#index-local-field-mapping)获取更多信息。
2. 在 Solr 索引设置中允许 `map-name` 选项，这会使 Solr 把属性键视为字段名。查阅[23.2. 全局字段映射] 获取更多信息。

### 26.4.3. 无模式模式
JanusGraph 也可以与为[无模式模式](https://cwiki.apache.org/confluence/display/solr/Schemaless+Mode)配置的 SolrCloud 集群进行交互。在这种情况下，应禁用配置选项 `dyn-fields`，否则 Solr 将从值而不是字段名称推断字段类型。

但请注意，无模式模式仅建议用于原型设计和初始应用程序开发，不建议用于生产用途。

## 26.5 故障排除
### 26.5.1. Collection Does Not Exist
必须在定义的索引可以使用集合之前将这个集合（以及所有必需的配置文件）初始化。更多相关信息请参阅[配置集设置](.)。

使用 SolrCloud 时，可用 Zookeeper zkCli.sh 命令行工具检查加载到 Zookeeper 中的配置。还要验证默认的 JanusGraph 配置文件是否已复制到 solr 下的正确位置，以及复制文件的目录是否正确。

### 26.5.2. Cannot Find the Specified Configset
使用 SolrCloud 时，需要用一个配置集来为 JanusGraph 创建一个混合索引。有关更多信息，请参阅 [配置集设置](.)。

+ 如果使用单个配置集，则必须首先手动创建集合。
+ 如果使用共享配置集，则必须首先将配置集上载到 Zookeeper。

您可以验证配置集及其配置文件是否位于 `/configs` 下的 Zookeeper 中。有关其他 Zookeeper 操作，请参阅[Solr参考指南](https://lucene.apache.org/solr/guide/6_6/solr-control-script-reference.html#SolrControlScriptReference-ZooKeeperOperations)。

```BASH
# verify the configset in Zookeeper
# Solr 5
$SOLR_HOME/server/scripts/cloud-scripts/zkcli.sh -cmd list -z localhost:2181
# Solr 6 and higher
$SOLR_HOME/bin/solr zk ls -r /configs/configset-name -z localhost:2181
```
### 26.5.3. HTTP Error 404
使用Solr Standalone（HTTP）模式时可能会遇到此错误。一个错误例子：

```
20:01:22 ERROR org.janusgraph.diskstorage.solr.SolrIndex  - Unable to save documents
to Solr as one of the shape objects stored were not compatible with Solr.
org.apache.solr.client.solrj.impl.HttpSolrClient$RemoteSolrException: Error from server
at http://localhost:8983/solr: Expected mime type application/octet-stream but got text/html.
<html>
<head>
<meta http-equiv="Content-Type" content="text/html;charset=utf-8"/>
<title>Error 404 Not Found</title>
</head>
<body><h2>HTTP ERROR 404</h2>
<p>Problem accessing /solr/verticesByAge/update. Reason:
<pre>    Not Found</pre></p>
</body>
</html>
```
在尝试将数据存储到索引之前，请确保已经手动创建核心。有关更多信息，请参阅[核心配置](.)

### 26.5.4. Invalid core or collection name
核心或集合的名称应是标识符。它必须完全由英语句点，下划线，连字符和"/"以及字母数字组成，也可能没有以连字符开头。

### 26.5.5. Connection Problems
无论哪种操作模式，Solr实例或 Solr 实例集群必须正在运行中，并且可从 JanusGraph 实例访问，JanusGraph 才能将 Solr 用作索引后端。检查Solr集群是否正常运行并且可达，并且 JanusGraph 实例可以通过网络（或本地）访问到。

### 26.5.6. JTS ClassNotFoundException with Geo Data
Solr 使用 Spatial4j 进性对 geo 的处理。 Spatial4j 声明了对 JTS （「JTS 拓扑套件」）的可选依赖。 JTS 对于一些 geo 字段定义和函数式查询来说是必须的。如果 JTS jar 包不在 Solr 守护进程的 classpath 内，并且 schema.xml 中的字段又使用了 geo 类型，那么 Solr 可能会由于缺少 JTS 类而扔出 `ClassNotFoundException`。该异常也可能在使用为 JanusGraph 设计的 schema.xml 去启动 Solr 服务器时。还有可能出现在 [Solr CoreAdmin API]https://wiki.apache.org/solr/CoreAdmin 的 `CREATE` 方法时。虽然客户端和服务器端该异常的格式略有不同，根本原因是相同的。

下面是 Solr 服务器日志的示例：
```
ERROR [http-8983-exec-5] 2014-10-07 02:54:06, 665 SolrCoreResourceManager.java (line 344) com/vividsolutions/jts/geom/Geometry
java.lang.NoClassDefFoundError: com/vividsolutions/jts/geom/Geometry
        at com.spatial4j.core.context.jts.JtsSpatialContextFactory.newSpatialContext(JtsSpatialContextFactory.java:30)
        at com.spatial4j.core.context.SpatialContextFactory.makeSpatialContext(SpatialContextFactory.java:83)
        at org.apache.solr.schema.AbstractSpatialFieldType.init(AbstractSpatialFieldType.java:95)
        at org.apache.solr.schema.AbstractSpatialPrefixTreeFieldType.init(AbstractSpatialPrefixTreeFieldType.java:43)
        at org.apache.solr.schema.SpatialRecursivePrefixTreeFieldType.init(SpatialRecursivePrefixTreeFieldType.java:37)
        at org.apache.solr.schema.FieldType.setArgs(FieldType.java:164)
        at org.apache.solr.schema.FieldTypePluginLoader.init(FieldTypePluginLoader.java:141)
        at org.apache.solr.schema.FieldTypePluginLoader.init(FieldTypePluginLoader.java:43)
        at org.apache.solr.util.plugin.AbstractPluginLoader.load(AbstractPluginLoader.java:190)
        at org.apache.solr.schema.IndexSchema.readSchema(IndexSchema.java:470)
        at com.datastax.bdp.search.solr.CassandraIndexSchema.readSchema(CassandraIndexSchema.java:72)
        at org.apache.solr.schema.IndexSchema.<init>(IndexSchema.java:168)
        at com.datastax.bdp.search.solr.CassandraIndexSchema.<init>(CassandraIndexSchema.java:54)
        at com.datastax.bdp.search.solr.core.CassandraCoreContainer.create(CassandraCoreContainer.java:210)
        at com.datastax.bdp.search.solr.core.SolrCoreResourceManager.createCore(SolrCoreResourceManager.java:256)
        at com.datastax.bdp.search.solr.handler.admin.CassandraCoreAdminHandler.handleCreateAction(CassandraCoreAdminHandler.java:117)
        ...
```
以下是通常出现在客户端向 CoreAdmin API 发出关联的 CREATE 命令的输出中的内容：
```
org.apache.solr.common.SolrException: com/vividsolutions/jts/geom/Geometry
        at com.datastax.bdp.search.solr.core.SolrCoreResourceManager.createCore(SolrCoreResourceManager.java:345)
        at com.datastax.bdp.search.solr.handler.admin.CassandraCoreAdminHandler.handleCreateAction(CassandraCoreAdminHandler.java:117)
        at org.apache.solr.handler.admin.CoreAdminHandler.handleRequestBody(CoreAdminHandler.java:152)
        ...
```
将 JTS jar 包添加到 JanusGraph 和/或 Solr 服务器的 classpath 下即可解决该问题。由于 JTS 使用了 LGPL 协议， JanusGraph 发行版中默认不包括 JTS。用户必须另行下载 JTS jar 文件，并将其复制到 JanusGraph 和/或 Solr 服务器的 lib 目录下。重启 Solr 后异常应该就解决了。 为了建立 example/solr-webapp/webapp/WEB-INF/lib 目录，必须先用正确的 schema.xml 文件启动 Solr 一次。[存疑]
> Solr must be started once with the correct schema.xml file in place first, for the example/solr-webapp/webapp/WEB-INF/lib directory to exist.

要确定理想的 Solr 服务器的 JTS 版本，首先检查 Solr 集群使用的 Spatial4j 的版本，然后根据 Spatial4j 编译版本确定 JTS 版本。JTS 在 [pom for the `com.spatial4j:spatial4j` artifact.](http://search.maven.org/#search|gav|1|g%3A%22com.spatial4j%22%20AND%20a%3A%22spatial4j%22) 上公布了其目标的 JTS 版本。在安装 solr 时将 JTS jar 复制到 server/solr-webapp/webapp/WEB-INF/lib 目录中。
## 26.6. Solr 进阶配置
### 26.6.1. DSE 搜索
本节介绍使用 DataStax Enterprise（DSE）搜索的 JanusGraph 的安装和配置。有多种方法可以安装 DSE，但本节重点介绍 DSE 在 Linux 上的二进制 tarball 安装(binary tarball install)选项。本节中的大多数步骤可以推广到 DSE 的其他安装选项。

按照页面[Installing DataStax Enterprise using the binary tarball](http://www.datastax.com/documentation/datastax_enterprise/4.5/datastax_enterprise/install/installTARdse.html)的指示直接安装

导出 DSE_HOME 加到 shell 环境的 PATH 中。以下是使用 Bash 语法的示例：
```bash
export DSE_HOME=/path/to/dse-version.number
export PATH="$DSE_HOME"/bin:"$PATH"
```
为Solr安装JTS。适配版本因 Spatial4j 版本而异。DSE 4.5.2 后的 JTS 适配版本是 1.13。
```bash
cd $DSE_HOME/resources/solr/lib
curl -O 'http://central.maven.org/maven2/com/vividsolutions/jts/1.13/jts-1.13.jar'
```
启动 DSE Cassandra Solr 单个后台守护程序：
```
# The "dse-data" path below was chosen to match the
# "Installing DataStax Enterprise using the binary tarball"
# documentation page from DataStax.  The exact path is not
# significant.
dse cassandra -s -Ddse.solr.data.dir="$DSE_HOME"/dse-data/solr
```
上一个命令会将一些启动信息写入控制台以及在 `$DSE_HOME/resources/cassandra/ conf/log4j-server.properties` 中配置的日志文件 `log4j.appender.R.File` 路径。
一旦使用 Cassandra 和 Solr 的 DSE 正常启动，请使用 nodetool status 检查群集运行状况。单实例环应该显示一个带有标志`* U * p`和`* N * ormal`的节点：
```
nodetool status
Note: Ownership information does not include topology; for complete information, specify a keyspace
= Datacenter: Solr
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address    Load       Owns   Host ID                               Token                                    Rack
UN  127.0.0.1  99.89 KB   100.0%  5484ef7b-ebce-4560-80f0-cbdcd9e9f496  -7317038863489909889                     rack1
```
接下来，切换到 Gremlin Console 并为 DSE 实例打开 JanusGraph 数据库。这将创建 JanusGraph 的键空间和列族。
```
cd $JANUSGRAPH_HOME
bin/gremlin.sh

         \,,,/
         (o o)
-----oOOo-(3)-oOOo-----
gremlin> graph = JanusGraphFactory.open('conf/janusgraph-cassandra-solr.properties')
==>janusgraph[cassandrathrift:[127.0.0.1]]
gremlin> g = graph.traversal()
==>graphtraversalsource[janusgraph[cassandrathrift:[127.0.0.1]], standard]
gremlin>
```
保持这个 Gremlin 控制台打开。回头安装一个 Solr 核心。然后我们回到此控制台以加载一些示例数据。然后，上传 JanusGraph 的 Solr 集合的配置文件，然后在DSE中创建核心：
```
# Change to the directory where JanusGraph was extracted.  Later commands
# use relative paths to the Solr config files shipped with the JanusGraph
# distribution.
cd $JANUSGRAPH_HOME

# The name must be URL safe and should contain one dot/full-stop
# character. The part of the name after the dot must not conflict with
# any of JanusGraph's internal CF names.  Starting the part after the dot
# "solr" will avoid a conflict with JanusGraph's internal CF names.
CORE_NAME=janusgraph.solr1
# Where to upload collection configuration and send CoreAdmin requests.
SOLR_HOST=localhost:8983

# The value of index.[X].solr.http-urls in JanusGraph's config file
# should match $SOLR_HOST and $CORE_NAME.  For example, given the
# $CORE_NAME and $SOLR_HOST values above, JanusGraph's config file would
# contain (assuming "search" is the desired index alias):
#
# index.search.solr.http-urls=http://localhost:8983/solr/janusgraph.solr1
#
# The stock JanusGraph config file conf/janusgraph-cassandra-solr.properties
# ships with this http-urls value.

# Upload Solr config files to DSE Search daemon
for xml in conf/solr/{solrconfig, schema, elevate}.xml ; do
    curl -v http://"$SOLR_HOST"/solr/resource/"$CORE_NAME/$xml" \
      --data-binary @"$xml" -H 'Content-type:text/xml; charset=utf-8'
done
for txt in conf/solr/{protwords, stopwords, synonyms}.txt ; do
    curl -v http://"$SOLR_HOST"/solr/resource/"$CORE_NAME/$txt" \
      --data-binary @"$txt" -H 'Content-type:text/plain; charset=utf-8'
done
sleep 5

# Create core using the Solr config files just uploaded above
curl "http://"$SOLR_HOST"/solr/admin/cores?action=CREATE&name=$CORE_NAME"
sleep 5

# Retrieve and print the status of the core we just created
curl "http://localhost:8983/solr/admin/cores?action=STATUS&core=$CORE_NAME"
```
现在，JanusGraph 数据库和后台 Solr 核心已经可以使用了。我们可以使用 [Graph of the Gods](https://docs.janusgraph.org/latest/getting-started.html) 数据集对其进行测试。接着用上面的 Gremlin 控制台会话：
```
// Assuming graph = JanusGraphFactory.open('conf/janusgraph-cassandra-solr.properties')...
gremlin> GraphOfTheGodsFactory.load(graph)
==>null
```
现在我们可以运行[第3章“入门”]中提到的任何查询。Solr将提供涉及文本和地理谓词的查询。有关 JanusGraph 和 Solr 客户端的更详细报告，请运行`gremlin.sh -l DEBUG`并发出一些索引支持的查询。

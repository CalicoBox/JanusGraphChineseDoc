# 25. Elasticsearch
> Elasticsearch 是一种应对日益增长的用例的，分布式的 RESTful 搜索和分析引擎。它作为 Elastic Stack 的核心，集中存储您的数据，以便您可以作出预期并防患于未然。
> -- [Elasticsearch Overview](https://www.elastic.co/products/elasticsearch/)

JanusGraph 支持 Elasticsearch 作为索引后端。下面是 JanusGraph 所支持的一部分 Elasticsearch 特性。

+ **全文**：支持所有搜索给定单词，前缀或正则表达式匹配的文本属性的 `Text` 谓词。
+ **Geo**：支持所有搜索给定查询几何体相交，相交或包含的地理属性的 `Geo` 谓词。支持点，圈，框，线和多边形索引。支持查询点属性的圆形，方框和多边形以及查询非点属性的所有形状。请注意，使用线形和多边形时需要 JTS (参阅 [Geoshape 文档](https://docs.janusgraph.org/latest/search-predicates.html#geoshape)以获取更多信息)。
+ **数字范围**:  支持 `Compare` 中所有的数字比较符。
+ **灵活的配置**： 支持远程操作和开放式自定义化设置。
+ **TTL**： 支持索引元素自动过期。
+ **集合**： 支持索引 SET 和 LIST 基数属性。
+ **瞬时**：纳秒粒度的时间索引。
+ **自定义分析器**： 选择使用自定义分析器。

请参阅 [附录2. 版本适配](https://docs.janusgraph.org/latest/version-compat.html) 以获取 ES 与 JanusGraph 一起使用的版本信息。

> **重要**
> 从 Elasticsearch 5.0 开始，JanusGraph 使用沙箱化的[无痛脚本](https://www.elastic.co/guide/en/elasticsearch/reference/master/modules-scripting-painless.html) 进行内联更新，并在 Elasticsearch 5.x 中默认启用。
> 使用带有 Elasticsearch 2.x 的 JanusGraph 需要通过设置 `script.engine.groovy.inline.update` 为 `true` 以在 Elasticsearch 集群上允许 Groovy 内联脚本。(参阅[动态脚本文档](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/modules-scripting.html#enable-dynamic-scripting)以获取更多信息)

## 25.1. 运行 Elasticsearch
JanusGraph 支持连接到运行中的 Elasticsearch 集群。 JanusGraph 提供两种方式来在运行中的本地 Elasticsearch 实例上快速开始。 JanusGraph 服务器(参阅[第7.1节 "开始"]())会自动启动一个本地 Elasticsearch 实例。或者 JanusGraph 包包含完整的 Elasticsearch 发行版，允许用户手动启用本地的 Elasticsearch 实例 (参阅[本节]()以获取更多信息)。
```bash
$ elasticsearch/bin/elasticsearch
```

> **注意**
> 出于安全因素，Elasticsearch 只能运行在非 root 用户下。

## 25.2. Elasticsearch 配置概览
JanusGraph 支持使用 HTTP 客户端连接到正在运行的 Elasticsearch 集群。请参阅[附录B，版本兼容性]获取关于哪些版本的 ES 将与 JanusGraph 中的不同客户端类型一起使用的详细信息。

> **注意**
> JanusGraph 中的索引选项以 「`index.[X]`」(X是用户定义的后端名) 开头。用户定义的名字必须像 [第9.1.2节，"混合索引"]中所述那样递交给 JanusGraph 管理系统接口，以便 JanusGraph 知晓多个配置的索引后端中哪个可能会被使用。本章中的配置片段使用名为 `search`，即片段讨论时选项通常用 `[X]` 描述。只要在 JanusGraph 的配置中以及管理索引时使用一致即可，确切的索引名称并不重要。

> **贴士**
> 推荐索引名只包含小写字母、数字和连字符，并以小写字母开头。

### 25.2.1. 连接到 Elasticsearch
如下，指定 Elasticsearch 客户端：
```scala
index.search.backend=elasticsearch
```
连接到 Elasticsearch 时，必须提供 Elasticsearch 实例的单个主机名或主机名列表。其由 JanusGraph 的 `index.[X].hostname` 关键字提供。
```scala
index.search.backend=elasticsearch
index.search.hostname=10.0.0.10:9200
```
此处指定的每一个主机或主机:端口都会被添加到 HTTP 客户端的请求目标的循环列表中。这有一个将会循环请求 10.0.0.10 的默认 Elasticsearch 端口(9200)和 10.0.0.20 的 7777 端口的极简配置：
```scala
index.search.backend=elasticsearch
index.search.hostname=10.0.0.10, 10.0.0.20:7777
```

### 25.2.1. JanusGraph `index.[X]` 和 `index.[X].elasticsearch` 项
JanusGraph 只对 `index-name` 和 `health-request-timeout` 使用默认值。参阅 [第13章]() 获取这些项和它们接受的值的说明。


+ `index.[X].elasticsearch.index-name`
+ `index.[X].elasticsearch.health-request-timeout`

### 25.2.2. REST Client Options

REST 客户端接受 `index.[X].bulk-refresh` 项。该项控制了何时可对搜索进行更改。参阅 [?refresh documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)获取更多信息。

### 25.2.3. 预处理管道
若使用 Elasticsearch 5.0或更高版本，则可以为每个混合索引设置不同的预处理管道。预处理管道可用于在编制索引之前预处理文档。管道由一系列处理器组成。每个处理器都以某种方式转换文档。例如，[日期处理器](https://www.elastic.co/guide/en/elasticsearch/reference/current/date-processor.html)可以从文本中提取日期，变为日期字段。因此，您可以使用 JanusGraph 查询此日期，而不必将其存储在主存储中。
+ `index.[X].elasticsearch.ingest-pipeline.[mixedIndexName] = pipeline_id`

查看[预处理文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)获取更多关于预处理管道的信息，查看[处理器文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest-processors.html)获取更多关于预处理器的信息。

## 25.3. Elasticsearch 安全
Elasticsearch 不执行身份验证或授权。可访问 ES 的客户端是被 ES 信任的。当 Elasticsearch 运行在不安全或公开的网络中，特别是互联网中时，应该部署一些外部安全措施。这通常通过 Elasticsearch 端口的防火墙和隧道的组合来完成。Elasticsearch 有两个面向客户端的端口需要考虑：
+ HTTP REST API，通常使用 9200 端口。
+ 原生 「传输」协议，通常使用 9300 端口。

客户端使用其中一个协议/端口，但不能同时使用。通常使用防火墙和具有 SSL 加密和 HTTP 身份验证的反向代理的组合来保护 HTTP 协议端口。

隧道 ES 的原生「传输」协议：
  这种方法可以通过 SSL / TLS 隧道（例如通过 [stunnel](https://www.stunnel.org/index.html)），VPN 或 SSH 端口转发来实现。 SSL / TLS 隧道需要进行不同寻常的配置和监视：隧道的一端或两端都需要证书，并且需要配置并持续运行 stunnel 进程。大多数安全 VPN 的配置同样重要。一些 Elasticsearch 服务提供商处理服务器端隧道管理并提供自定义 Elasticsearch `transport.type` 以简化客户端配置。
添加防火墙规则，该规则仅允许受信任的客户端在 Elasticsearch 的本机协议端口上进行连接
  这通常在主机防火墙级别完成。易于配置，但本身安全性非常差。

## 25.4. 索引创建选项
JanusGraph 在用于创建 Elasticsearch 索引时支持自定义。其允许在 JanusGraph 发布的 [Elasticsearch `create index` request](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 中的 `settings` 对象中随意使用键值对。以下是一个可以使用此机制自定义的Elasticsearch索引设置的非详尽示例：

```java
index.number_of_replicas
index.number_of_shards
index.refresh_interval
```

通过此机制自定义的设置仅在 JanusGraph 尝试在 Elasticsearch 中创建其索引时应用。如果JanusGraph发现其索引已经存在，那么它不会尝试重新创建它，并且这些设置将会无效。

### 25.4.1. 使用 `create.ext` 的嵌入式 ES 索引设置
JanusGraph 会对所有以 `index.[X].elasticsearch.create.ext` 为前缀的属性进行迭代，其中`[X]`是像 `search` 这样的索引名。JanusGraph 从每个属性键中去除前缀​​。剥离出的键的其余部分将被解释为 Elasticsearch 索引创建设置。与密钥相关的值不会被修改。被剥离的密钥和未修改的值会作为 Elasticsearch 创建索引请求中的 `settings` 对象的一部分被传递，并在 JanusGraph 在 ES 上引导时发出。从而允许在属性中嵌入任意 JanusGraph 属性中的索引创建设置的设置。下面是一个示例配置片段，它使用 `create.ext` 配置机制自定义了三个 Elasticsearch 索引设置：

```scala
index.search.backend=elasticsearch
index.search.elasticsearch.create.ext.number_of_shards=15
index.search.elasticsearch.create.ext.number_of_replicas=3
index.search.elasticsearch.create.ext.shard.check_on_startup=true
```

上面列出的配置片段利用了 Elasticsearch 的推断机制，在服务器端实现， `create index` 设置键是无修饰的，带有 `index.` 前缀。也可以明确地拼出索引前缀。下面是一个 JanusGraph 配置文件，除显式声明了索引创建设置前的 `index.` 外功能上与上面列出的配置文件相同：

```scala
index.search.backend=elasticsearch
index.search.elasticsearch.create.ext.index.number_of_shards=15
index.search.elasticsearch.create.ext.index.number_of_replicas=3
index.search.elasticsearch.create.ext.index.shard.check_on_startup=false
```

> 小贴士
> 用于指定索引创建设置的 `create.ext` 机制与 JanusGraph 的 Elasticsearch 配置兼容。

## 25.5. 故障排除
### 25.5.1. 连接到远程 Elasticsearch 集群的问题
检查 JanusGraph 节点的 HTTP 协议端口是否可以访问 Elasticsearch 集群节点。检查 Elasticsearch 节点配置的日志或使用 `netstat` 等常规诊断实用程序来检查节点侦听端口。检查 JanusGraph 配置。

## 25.6. 优化 Elasticsearch
### 25.6.1. 写优化
对于[批量加载](https://docs.janusgraph.org/latest/bulk-loading.html)或其他写密集型应用程序，请考虑增加 Elasticsearch 的刷新间隔。关于如何增加刷新间隔以及其对写性能的影响，参考[这个讨论](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-indexing-speed.html)。请注意，刷新间隔越高意味着图形突变在索引中可用的时间越长。

关于如何在 Elasticsearch 中提高写性能的其他建议及详细说明，请阅读[此博客文章](http://blog.bugsense.com/post/35580279634/indexing-bigdata-with-elasticsearch)。

### 25.6.2. 进一步阅读
请参阅[Elasticsearch 主页](https://www.elastic.co/)和可用文档， 获取有关 Elasticsearch 以及如何设置 Elasticsearch 集群的更多信息。

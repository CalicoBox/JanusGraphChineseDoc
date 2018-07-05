# 28. 高级架构
本页介绍了一些 JanusGraph 提供的高级架构定义设置项。有关 JanusGraph 一般架构以及如何定义一般架构的更多信息，参考[5. 架构和数据模型](https://docs.janusgraph.org/latest/schema.html)。

## 28.1. 静态顶点
顶点标签可以定义为 **静态**，意思是具有该标签的顶点除了创建它们的事务之外，其它事务无法进行对它们修改。
```java
mgmt = graph.openManagement()
tweet = mgmt.makeVertexLabel('tweet').setStatic().make()
mgmt.commit()
```
静态顶点标签是一种控制数据生命周期的方法，在加载那些在创建后不应对其进行修改的图中数据时非常有用。

## 28.2. 边和顶点 TTL
### 28.2.1. 边 TTL
边 TTL 基于每条边都有的标签来定义，这意味着所有的标签相同的边具有相同的存活时间(time-to-live, TTL)。注意，后端必须支持单元级别的 TTL。目前只有 Cassandra 和 HBase 支持单元级 TTL。
```java
mgmt = graph.openManagement()
visits = mgmt.makeEdgeLabel('visits').make()
mgmt.setTTL(visits, Duration.ofDays(7))
mgmt.commit()
```
注意，修改一条边会重置这条边的 TTL。此外还要注意， 边标签的 TTL 是可修改的， 但是这需要一些时间将该更改传播到所有的 JanusGraph 实例上，这可能导致对于同一个标签，短时间内有两个不同的可用 TTL 的情况。
### 28.2.2. 属性 TTL
属性 TTL 与边 TTL 非常相似，基于每个属性都有的标签来定义，这意味着所有的标签相同的所有属性具有相同的存活时间(time-to-live, TTL)。目前只有 Cassandra 和 HBase 支持单元级 TTL。
```java
mgmt = graph.openManagement()
sensor = mgmt.makePropertyKey('sensor').cardinality(Cardinality.LIST).dataType(Double.class).make()
mgmt.setTTL(sensor, Duration.ofDays(21))
mgmt.commit()
```
同边的 TTL 一样，修改已存在的属性会重置其 TTL，但可能需要一些时间来生效。

### 28.2.3. 顶点 TTL
点 TTL 基于每个点都有的标签来定义，这意味着所有的标签相同的边具有相同的存活时间(time-to-live, TTL)。TTL 配置会应用于顶点、顶点的属性和所有入射边，以确保可以从图中完全删除该顶点。因此，必须先把顶点的标签定义为 static 的，然后才能设置 TTL，以避免任何可能导致点 TTL 失效的修改。顶点的 TTL 只用于静态顶点的标签。注意，后端必须支持单元级别的 TTL。目前只有 Cassandra 和 HBase 支持存储级 TTL。
```java
mgmt = graph.openManagement()
tweet = mgmt.makeVertexLabel('tweet').setStatic().make()
mgmt.setTTL(tweet, Duration.ofHours(36))
mgmt.commit()
```
此外还要注意， 点标签的 TTL 是可修改的， 但是这需要一些时间将该更改传播到所有的 JanusGraph 实例上，这可能导致对于同一个标签，短时间内有两个不同的可用 TTL 的情况。

## 28.3. 多重属性
如同 [第5章，架构和数据模型](https://docs.janusgraph.org/latest/schema.html) 中所述，JanusGraph 支持使用 SET 和 LIST 基数的属性键。因此，JanusGraph 支持单个顶点的单键上有多重属性。除此之外，同 JanusGraph 处理边相似，在处理属性时允许在属性上使用单值属性注解。如下示例所示：
```java
mgmt = graph.openManagement()
mgmt.makePropertyKey('name').dataType(String.class).cardinality(Cardinality.LIST).make()
mgmt.commit()
v = graph.addVertex()
p1 = v.property('name', 'Dan LaRocque')
p1.property('source', 'web')
p2 = v.property('name', 'dalaro')
p2.property('source', 'github')
graph.tx().commit()
v.properties('name')
==> Iterable over all name properties
```
这些特性对许多应用而言都是有用的，例如某些需要附加出处信息的属性(比如，谁在何时何地添加属性?)。对更高级的基数属性和属性上的属性注解的支持，在如[第 29 章，最终一致的存储后端](https://docs.janusgraph.org/latest/eventual-consistency.html)中描述高并发，横向扩展设计模式上也很有用处。

与其对边的支持一样，也支持对属性以顶点为中心的索引和全局图索引。关于定义边的索引和使用通信 API 方法定义属性的索引的更多信息，请参考 [第 9 章，索引——提高性能](https://docs.janusgraph.org/latest/indexes.html)。

## 28.4. 单向边
单向边是只能朝外方向移动的边。单向边占用的存储空间更小，但在进行遍历时支持的类型会受到限制。单向边在概念上与万维网的超链接类似，即外顶点可以遍历边，但内顶点并不知晓其存在。
```java
mgmt = graph.openManagement()
mgmt.makeEdgeLabel('author').unidirected().make()
mgmt.commit()
```
可在边和属性上添加单向边，从而为 JanusGraph 提供了对超边的有限支持。比如下例中，我们在 `knows` 边上添加单向的 `author` 边，以存储 `user` 将此边存储到图的事实，这对于捕边的作者出处信息非常有用。
```java
user = graph.addVertex()
book = graph.addVertex()
author = graph.addVertex()
user.addE('knows', book).property('author', author)
```
注意，删除其顶点时，不会自动删除以该顶点作为内点的单向边。用户必须显式检查事务中的顶点的存在情况，以确保不会出现不一致，或在查询时处理掉这些不一致。更多信息请见[29.2.2. 「访客顶点」](https://docs.janusgraph.org/latest/eventual-consistency.html#ghost-vertices)中的讨论。

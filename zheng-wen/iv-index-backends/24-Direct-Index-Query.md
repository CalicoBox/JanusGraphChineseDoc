# 24.直接索引查询
JanusGraph 标准的全图查询机制支持对边和顶点进行布尔查询。也就是，一个元素匹配查询结果与否。没有部分匹配或结果评分的说法。

一些存储后端附加了对模糊搜索查询的支持。进行这些查询时，会对每个匹配对象的「好值」(goodness)进行计算，并返回匹配分数的有序结果。在进行以匹配单词越多越好的全文搜索查询的处理时，模糊搜索尤其有用。

由于各个存储后端的模糊搜索的实现和分数算法有着显著差异，JanusGraph 不提供模糊匹配的原生支持。但是，JanusGraph 提供一个直接索引查询机制，其允许搜索查询直接发送到(那些支持的)索引后端进行评估。

使用 `Graph.indexQuery()` 来组成一个直接针对索引后端执行的查询。该查询生成器需要两个参数。

1. 查询的存储后端的名字。该名字必须与 JanusGraph 配置中的名字及属性键索引定义中使用的名字所适配。
2. 查询字符串。

该生成器允许使用 `limit(int)` 方法配置返回的最大元素数。通过 `offset(int)` 控制结果中要跳过的初始匹配数。调用 `vertices()` 或 `edges()` 来获取指定索引后端中所有符合给定查询条件的顶点或边。一次无法既获取边又获取顶点。这些方法通过 `Result` 对象返回一个 `Iterable`。一个结果(`Result`)对象包含匹配句柄，可通过 `getElement()` 获取，还包括其关联的分数，可通过 `getScore` 获取。

考虑如下例子：
```java
ManagementSystem mgmt = graph.openManagement();
PropertyKey text = mgmt.makePropertyKey("text").dataType(String.class).make();
mgmt.buildIndex("vertexByText", Vertex.class).addKey(text).buildMixedIndex("search");
mgmt.commit();
// ... Load vertices ...
for (Result<Vertex> result : graph.indexQuery("vertexByText", "v.text:(farm uncle berry)").vertices()) {
   System.out.println(result.getElement() + ": " + result.getScore());
}
```

# 24.1. 查询字符串
字符串查询直接交递给索引后端进行处理，因此查询字符串的句法取决于索引后端支持什么。对于点的查询，JanusGraph 将分析以「`v.`」开头的属性键引用的查询字符串，并用属性键对应的索引字段的句柄替换它。同样，对于边的查询，JanusGraph 会替换以「`e.`」开头的属性键的引用，因此，在查询字符串时使用「`v.[KEYNAME]`」来获取一个顶点的属性。同样，使用「`e.[KEYNAME]`」来获取一个边的属性。

[Elasticsearch](https://docs.janusgraph.org/latest/elasticsearch.html) 和 [Lucene](https://docs.janusgraph.org/latest/lucene.html) 支持 [Lucene 查询句法](http://lucene.apache.org/core/4_10_4/queryparser/org/apache/lucene/queryparser/classic/package-summary.html)。更多信息请参阅 [Lucene 文档](http://lucene.apache.org/core/4_1_0/queryparser/org/apache/lucene/queryparser/classic/package-summary.html) 和 [Elasticsearch 文档](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)。上面的例子就使用了 Lucene 查询语法

```java
graph.indexQuery("vertexByText", "v.text:(farm uncle berry)").vertices()
```

查询将匹配所有包含这三个词语的顶点(由括号分组)，并且匹配的单词数越多，匹配得分也越高。

此外 [Elasticsearch](https://docs.janusgraph.org/latest/elasticsearch.html) 还支持通配符查询， 在查询字符串时使用「`v.*`」或「`e.*`」来查询元素上的每一个属性是否能够匹配。

# 24.2. 查询总和
有时只需要知道从查询返回的总结果的数量，而无需获取所有结果。幸运的是 Elasticsearch 和 Solr 提供了捷径，可以不涉及对所有文件进行获取和排序。该捷径通过 "`.vertexTotals()`", "`.edgeTotals()`", and "`.propertyTotals()`" 方法暴露。

# 24.3. 陷阱
## 24.3.1. Property Key Names
属性键的名字不能包含非字母符号且必须放在引号内，以保证查询请求可被正确解析。
```java
graph.indexQuery("vertexByText", "v.\"first_name\":john").vertices()
```

## 24.3.2. Element Identifier Collision
字符串 "`v.`", "`e.`", and "`p.`" 分别用于标识查询中的顶点，边或属性元素。 如果字段名称或查询值包含相同的字符序列，则会导致查询字符串中的冲突和解析错误，如下例所示：
```java
graph.indexQuery("vertexByText", "v.name:v.john").vertices() //DOES NOT WORK!
```
使用 `setElementIdentifier` 方法来定义一个独有的，在查询任何地方都决不会出现的标识符字符串，来避免上述标识符冲突：
```java
graph.indexQuery("vertexByText", "$v$name:v.john").setElementIdentifier("$v$").vertices()
```

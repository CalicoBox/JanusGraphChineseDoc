# 22. 索引参数和全文搜索

当定义一个混合索引时，可以为添加到索引中的每一个属性键指定一个参数列表。这些参数决定控制了特殊键如何被索引。 JanusGraph 承认下文所列的索引参数，是否支持这些索引参数则取决于存储后端的具体配置。对于一个特定的存储后端而言，除了支持在此列出的参数，还可以支持自定义的参数。

## 22.1. 全文搜索
默认情况下，字符串被索引为文本。若要设置显式的索引选项，可在将属性键索引为文本时定义映射关系。
```java
mgmt = graph.openManagement()
summary = mgmt.makePropertyKey('booksummary').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(summary, Mapping.TEXT.asParameter()).buildMixedIndex("search")
mgmt.commit()
```

与标准的混合索引定义相同，只是增加了一个额外的参数来指定索引中的映射 —— 即例子中的`Mapping.TEXT`。

当一个字符串属性作为文本被索引时，字符串值被标记为一包令牌。其确切的标记由索引后端及其配置来决定。JanusGraph 默认的标记化方式是将字符串分割为非字母数字字符，并删除其中少于2个字符的任何标记。使用索引后端产生的标记可能会不同(例如停止词会被删除)，这可能导致处理事务内的修改以及索引后端中的已提交数据的全文本搜索查询的细微差别。

当字符串属性被索引为文本时，在索引后端的图形查询中只支持全文搜索谓词。全文搜索不区分大小写。
+ `textConains`: 为真, 当文本中(至少)包含一个与查询字符串相等的单词时。
+ `textContainsPrefix`: 为真, 当文本中(至少)包含一个使用查询字符串开头的单词时。
+ `textContainsRegex`: 为真, 当文本中(至少)包含一个完全匹配所给正则表达式的单词时。
+ `textContainsFuzzy`: 为真, 当文本中(至少)包含一个与查询字符串相似的单词时(基于 Levenshtein edit distance)。

```java
import static org.janusgraph.core.attribute.Text.*;

g.V().has('booksummary', textContains('unicorns'))
g.V().has('booksummary', textContainsPrefix('uni'))
g.V().has('booksummary', textContainsRegex('.*corn.*'))
g.V().has('booksummary', textContainsFuzzy('unicorn'))
```

字符串搜索谓词（见下文）可用于查询，但是其需要在内存中进行过滤，可能会开销很大。

## 22.1.2. 字符串搜索
指定映射方式为`Mapping.STRING`可将字符串属性索引为字符序列，而不使用任何分析和标记。
```java
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('bookname').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(name, Mapping.STRING.asParameter()).buildMixedIndex("search")
mgmt.commit()
```
配置字符串映射后，字符串值被编入索引，并且可“按原样”查询 - 包括停止词和非字母字符。但是，在这种情况下，查询必须匹配整个字符串的值。因此，当索引被认为是一个标记的短字符序列时，字符串映射非常有用。

当一个字符串属性被索引为字符串时，索引后端在图的查询中仅支持以下谓词。字符串搜索区分大小写。
+ `eq`, 为真, 当字符串与查询字符串完全相同时
+ `neq`, 为真, 当字符串与查询字符串不同时
+ `textPrefix`: 为真, 当文本使用查询字符串开头时。
+ `textRegular`: 为真, 当文本完全匹配所给的正则表达式时。
+ `textFuzzy`: 为真, 当文本与查询字符串相似时(基于 Levenshtein edit distance)。

```java
import static org.apache.tinkerpop.gremlin.process.traversal.P.*;
import static org.janusgraph.core.attribute.Text.*;
g.V().has('bookname', eq('unicorns'))
g.V().has('bookname', neq('unicorns'))
g.V().has('bookname', textPrefix('uni'))
g.V().has('bookname', textRegex('.*corn.*'))
g.V().has('bookname', textFuzzy('unicorn'))
```
全文搜索谓词（见下文）可用于查询，但是其需要在内存中进行过滤，可能会开销很大。

## 22.1.3. 全文和字符串搜索
如果您使用的是 Elasticsearch，可以将属性索引为文本和字符串，从而允许您使用所有谓词进行精确匹配和模糊匹配。

```java
mgmt = graph.openManagement()
summary = mgmt.makePropertyKey('booksummary').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(summary, Mapping.TEXTSTRING.asParameter()).buildMixedIndex("search")
mgmt.commit()
```

注意，数据会在索引中存储两次，一次为了精准的搜索，另一次为了模糊匹配。

## 22.2. Geo 映射
JanusGraph 默认支持使用点类型对 geo 进行索引和使用圆和盒对 geo 数据进行查询。使用 `Mapping.PREFIX_TREE` 来对无点数据的 geo 属性进行索引，并支持使用任何 geoshape 类型进行查询。

```java
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('border').dataType(Geoshape.class).make()
mgmt.buildIndex('borderIndex', Vertex.class).addKey(name, Mapping.PREFIX_TREE.asParameter()).buildMixedIndex("search")
mgmt.commit()
```
可以指定其他参数来调整基础前缀树映射的配置。这些可选参数包括前缀树中使用的级别数以及关联的精度。
```java
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('border').dataType(Geoshape.class).make()
mgmt.buildIndex('borderIndex', Vertex.class).addKey(name, Mapping.PREFIX_TREE.asParameter(),
 Parameter.of("index-geo-max-levels", 18), Parameter.of("index-geo-dist-error-pct", 0.0125)).buildMixedIndex("search")
mgmt.commit()
```
请注意，某些索引后端（例如Solr）可能需要额外的外部模式配置才能支持并调整索引非点属性。

# 27. Apache Lucene

> Apache Lucene 是一款完全使用 Java 编写的性能优秀、功能齐全的文字搜索引擎库。其适用于几乎所有需要全文搜索，特别是跨平台搜索的应用。Apache Lucene 是一款开源软件，可以免费下载获取。
> -- [Apache Lucene 主页](http://lucene.apache.org/)

JanusGraph 支持 [Apache Lucene] 作为单机嵌入式索引后端。 相较于 [Elasticsearch](https://docs.janusgraph.org/latest/elasticsearch.html)，Lucene 拥有稍微扩展的功能集和小尺度应用上更好的性能表现，但其仅限于单机部署。

## 27.1. Lucene 嵌入式配置
单机部署时，Lucene 运行嵌入了 JanusGraph。JanusGraph 在 Lucene 的内部启动和接口交互。

将下面配置选项添加到 `/data/searchindex` 中指定用于 Lucene 存放索引数据的目录的图配置文件下，以运行嵌入式

```
index.search.backend=lucene
index.search.directory=/data/searchindex
```

上面配置中，该索引名为 `search`，用其它名字把 `search` 替换掉，则索引的名字也会相应变化。

## 27.2. 支持的特性
+ **全文**：支持所有搜索给定单词，前缀或正则表达式匹配的文本属性的 `Text` 谓词。
+ **Geo**：支持所有搜索给定查询几何体相交，相交或包含的地理属性的 `Geo` 谓词。支持点，圈，框，线和多边形索引。支持查询点属性的圆形，方框和多边形以及查询非点属性的所有形状。请注意，使用线形和多边形时需要 JTS (参阅 [Geoshape 文档](https://docs.janusgraph.org/latest/search-predicates.html#geoshape)以获取更多信息)。
+ **数字范围**:  支持 `Compare` 中所有的数字比较符。
+ **瞬时**：纳秒粒度的时间索引。

## 27.3. 配置选项
参阅 [13. 配置相关](https://docs.janusgraph.org/latest/config-ref.html) 以包括所有 Lucene 特有配置设置项的完整列表在内的 JanusGraph 一般配置设置项列表。

注意， 每一个索引后端设置项都应使用 `index.[INDEX-NAME]` 作为前缀。`[INDEX-NAME]` 代表索引后端的名字。比如说，有一个叫做 search 的索引后端，那么配置项就应该以 index.search. 作为前缀。 将一个名为 search 的索引后端设置为使用 Lucene 作为索引系统，配置项应当如下设置：
```
index.search.backend=lucene
```
## 27.4. 拓展阅读
请参考 [Apache Lucene 主页](http://lucene.apache.org/) 及其可用文档获取更多关于 Lucene 的信息。

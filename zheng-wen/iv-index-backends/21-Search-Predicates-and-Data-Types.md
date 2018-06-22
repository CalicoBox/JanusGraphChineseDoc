# 21. 搜索谓词和数据类型
本页列出了 JanusGraph 进行全局图搜索和本地遍历时所支持的所有比较谓词。

## 21.1. 比较 (Compare) 谓词
`Compare` 枚举值指定了以下用于索引查询构造和上面例子的比较谓词：
+ `eq`, equal, 等于
+ `neq`, not equal, 不等于
+ `gt`, great than, 大于
+ `gte`, great than or equal, 大于等于
+ `lt`, less than, 小于
+ `lte`, lees than or equal, 小于等于

## 21.2. 文本 (Text) 谓词
`Text` 枚举值指定了用于查询匹配文本和字符串的[搜索操作符](https://docs.janusgraph.org/latest/index-parameters.html#text-search)。我们对两种类型做如下区分:

+ 文本匹配谓词，匹配目标字符串中的单个词语，这些谓词不是大小敏感的。
  + `textConains`: 为真, 当文本中(至少)包含一个与查询字符串相等的单词时。
  + `textContainsPrefix`: 为真, 当文本中(至少)包含一个使用查询字符串开头的单词时。
  + `textContainsRegex`: 为真, 当文本中(至少)包含一个完全匹配所给正则表达式的单词时。
  + `textContainsFuzzy`: 为真, 当文本中(至少)包含一个与查询字符串相似的单词时(基于 Levenshtein edit distance)。
+ 字符串搜索谓词，匹配整个字符串的值。
  + `textPrefix`: 为真, 当文本使用查询字符串开头时。
  + `textRegular`: 为真, 当文本完全匹配所给的正则表达式时。
  + `textFuzzy`: 为真, 当文本与查询字符串相似时(基于 Levenshtein edit distance)。

关于全文和字符串搜索的更多信息请参阅 [22.1][1] 节「全文搜索」。

## 21.3. Geo 谓词
Geo 谓词枚举了一些特殊的地理位置用谓词。

+ `geoIntersect` 当两个几何物体至少有一个公共点时为真(与 `geoDisjoint` 相反)。
+ `geoWithin` 当一个几何物体包含另一个时为真。
+ `geoDisjoint` 当两个几何物体没有任何公共点时为真(与 `geoIntersect` 相反)。
+ `geoContains` 当一个几何物体**被**另一个包含时为真。

关于 geo 搜索的更多信息请参阅 [22.2][2] 节「Geo 映射」

## 21.4. 查询示例
以下查询示例演示了教程的图上的一些谓词操作。

```java
g.V().has("name", "hercules")
// 2) 查询所有度大于 50 的顶点
g.V().has("age", gt(50))
// 查询所有 age 在 1000(包括) 到 5000(不包括) 之间的顶点，并按 age 的增序排列
g.V().has("age", inside(1000, 5000)).order().by("age", incr)
// 返回同样的查询结果, 但是以递减序排列
g.V().has("age", inside(1000, 5000)).order().by("age", decr)
// 3) 查找距离给定的经纬度 50 公里内的所有的边。
g.E().has("place", geoWithin(Geoshape.circle(37.97, 23.72, 50)))
// 4) 查找所有 reason 中包含单词 love 的边。
g.E().has("reason", textContains("loves"))
// 查找所有包含两个单词的边 (需要分词)
g.E().has("reason", textContains("loves")).has("reason", textContains("breezes"))
// 查找所有包含以 lov 开头的单词的边
g.E().has("reason", textContainsPrefix("lov"))
// 或查找所有包含与正则表达式 "br[ez]*s" 完全匹配的单词的边
g.E().has("reason", textContainsRegex("br[ez]*s"))
// 查找所有包含与 love 相似的单词的边
g.E().has("reason", textContainsFuzzy("love"))
// 5) 查找所有 age 大于 1000 年并且名字叫做 saturn 的顶点
g.V().has("age", gt(1000)).has("name", "saturn")
```
(看起来官方文档这里就少了一行注释，不关我的事)

## 21.5. 支持的数据类型
尽管 JanusGraph 的复合索引(composite indexes)支持任何可以存储在 JanusGraph 中的数据类型，但是混合索引(mixed indexes)仅限于以下的数据类型：
+ Byte
+ Short
+ Integer
+ Long
+ Float
+ Double
+ String
+ Geoshape
+ Date
+ Instant
+ UUID

将来将会支持更多的数据类型。

## 21.6. Geoshape数据类型
Geoshape 数据类型支持表示点，圆，框，线，多边形，多点，多线和多边形。索引后端目前支持索引点，圆，框，线，多边形，多点，多线，多面和几何集合。仅通过混合索引(mixed indexes)支持 Geospatial 索引查找。

> 注意：目前Java 拓扑套件（JTS）可以使用带多边形的多边形，多边形和几何集合。由于 JTS 使用了 LGPL协议，其不默认包含在 JanusGraph 发行版中。用户如果需要完整的几何图形支持时，必须另外自行下载 JTS JAR文件然后放在 classpath 下。

用下面的方法构建 Geoshape ：
```java
//lat, lng
Geoshape.point(37.97, 23.72)
//lat, lng, radius in km
Geoshape.circle(37.97, 23.72, 50)
//SW lat, SW lng, NE lat, NE lng
Geoshape.box(37.97, 23.72, 38.97, 24.72)
//WKT (requires JTS for polygon geometries)
Geoshape.fromWkt("POLYGON ((35.4 48.9, 35.6 48.9, 35.6 49.1, 35.4 49.1, 35.4 48.9))")
//MultiPoint
Geoshape.geoshape(Geoshape.getShapeFactory().multiPoint().pointXY(60.0, 60.0).pointXY(120.0, 60.0)
 .build())
//MultiLine
Geoshape.geoshape(Geoshape.getShapeFactory().multiLineString()
 .add(Geoshape.getShapeFactory().lineString().pointXY(59.0, 60.0).pointXY(61.0, 60.0))
 .add(Geoshape.getShapeFactory().lineString().pointXY(119.0, 60.0).pointXY(121.0, 60.0)).build())
//MultiPolygon (requires JTS)
Geoshape.geoshape(Geoshape.getShapeFactory().multiPolygon()
 .add(Geoshape.getShapeFactory().polygon().pointXY(59.0, 59.0).pointXY(61.0, 59.0)
   .pointXY(61.0, 61.0).pointXY(59.0, 61.0).pointXY(59.0, 59.0))
 .add(Geoshape.getShapeFactory().polygon().pointXY(119.0, 59.0).pointXY(121.0, 59.0)
   .pointXY(121.0, 61.0).pointXY(119.0, 61.0).pointXY(119.0, 59.0)).build())
//GeometryCollection (may requires JTS)
Geoshape.geoshape(Geoshape.getGeometryCollectionBuilder()
 .add(Geoshape.getShapeFactory().pointXY(60.0, 60.0))
 .add(Geoshape.getShapeFactory().lineString().pointXY(119.0, 60.0).pointXY(121.0, 60.0).build())
 .add(Geoshape.getShapeFactory().polygon().pointXY(119.0, 59.0).pointXY(121.0, 59.0)
   .pointXY(121.0, 61.0).pointXY(119.0, 61.0).pointXY(119.0, 59.0)).build())
```
此外，当使用 GraphSON 导入图信息时可用 GeoJSON 表示：
```json
// string
"37.97, 23.72"
// list
[37.97, 23.72]
// GeoJSON feature
{
 "type": "Feature",
 "geometry": {
   "type": "Point",
   "coordinates": [125.6, 10.1]
 },
 "properties": {
   "name": "Dinagat Islands"
 }
}
// GeoJSON geometry
{
 "type": "Point",
 "coordinates": [125.6, 10.1]
}
```

[GeoJSON](http://geojson.org/) 可以被指定为点，圆，线性字符串或多边形。多边形必须是闭合的。请注意，与 JanusGraph API 不同，GeoJSON 将坐标指定为 lng lat。

## 21.7. 集合
如果您在使用 [Elasticsearch](https://docs.janusgraph.org/latest/elasticsearch.html) ，那么您可以使用 SET 和 LIST 基数来索引属性。例如：
```java
mgmt = graph.openManagement()
nameProperty = mgmt.makePropertyKey("names").dataType(String.class).cardinality(Cardinality.SET).make()
mgmt.buildIndex("search", Vertex.class).addKey(nameProperty, Mapping.STRING.asParameter()).buildMixedIndex("search")
mgmt.commit()
//插入点
person = graph.addVertex()
person.property("names", "Robert")
person.property("names", "Bob")
graph.tx().commit()
//查询该点
g.V().has("names", "Bob").count().next() //1
g.V().has("names", "Robert").count().next() //1
```
[1]:/22.md
[2]:/22.md

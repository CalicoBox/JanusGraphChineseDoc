# 23. 字段映射
## 23.1. 单个字段映射
JanusGraph 默认会将属性键进行编码，以生成混合索引中的属性键唯一的字段名。如果想要在外部索引后端直接查询混合索引，后端则很难处理并且会变得难以辨认。对于这种用例，可以通过参数显式指定字段名。

```java
mgmt = graph.openManagement()
name = mgmt.makePropertyKey('bookname').dataType(String.class).make()
mgmt.buildIndex('booksBySummary', Vertex.class).addKey(name, Parameter.of('mapped-name', 'bookname')).buildMixedIndex("search")
mgmt.commit()
```

通过将此字段映射定义为参数，JanusGraph 将为外部索引系统中创建的 `booksBySummary` 索引中的字段使用与属性键相同的名称。

## 23.2. 全局字段映射
除了单独地调整每一个添加到混合索引的键的字段映射，还可以指示 JanusGraph 始终将外部索引中的字段名设置为与属性键名相同。这是通过启用配置项 `map-name` (为每个存储后端配置选项) 来配置的。若对特定的存储后端启用此选项，则将针对所述后端的所有混合索引，使用与属性键名称相同的字段名。

但这种途径有两个限制：1) 用户必须保证作为属性键名的索引后端字段名是合法的，2)对属性键的重命名**不会**导致索引中的字段名发生重命名，这会发生用户理应知道和避免的命名冲突。

注意，上面所述的单个字段映射可用于覆盖特定键的默认名称。

## 23.2.1. 自定义分析器
JanusGraph 默认使用索引后端的默认分析器来分析 Mapping.TEXT的属性，并没有分析器作用于 Mapping.STRING 的属性的分析器。如果有人想使用别的分析器，可使用一个参数来进行显式的声明： Mapping.TEXT 的 ParameterType.TEXT_ANALYZER 和 Mapping.STRING 的 ParameterType.STRING_ANALYZER。

### 23.2.1.1. 用于 Elasticsearch
必须设置为参数值，参是分析器名称。
```java
mgmt = graph.openManagement()
string = mgmt.makePropertyKey('string').dataType(String.class).make()
text = mgmt.makePropertyKey('text').dataType(String.class).make()
textString = mgmt.makePropertyKey('textString').dataType(String.class).make()
mgmt.buildIndex('string', Vertex.class).addKey(string, Mapping.STRING.asParameter(), Parameter.of(ParameterType.STRING_ANALYZER.getName(), 'standard')).buildMixedIndex("search")
mgmt.buildIndex('text', Vertex.class).addKey(text, Mapping.TEXT.asParameter(), Parameter.of(ParameterType.TEXT_ANALYZER.getName(), 'english')).buildMixedIndex("search")
mgmt.buildIndex('textString', Vertex.class).addKey(text, Mapping.TEXTSTRING.asParameter(), Parameter.of(ParameterType.STRING_ANALYZER.getName(), 'standard'), Parameter.of(ParameterType.TEXT_ANALYZER.getName(), 'english')).buildMixedIndex("search")
mgmt.commit()
```
使用如上设置， JanusGraph 将会使用标准分析器来分析属性键字符串，使用英文分析器来分析属性键文本。

### 23.2.1.2 用于 Solr
必须设置参数值，参是标记器的类。
```java
mgmt = graph.openManagement()
string = mgmt.makePropertyKey('string').dataType(String.class).make()
text = mgmt.makePropertyKey('text').dataType(String.class).make()
mgmt.buildIndex('string', Vertex.class).addKey(string, Mapping.STRING.asParameter(), Parameter.of(ParameterType.STRING_ANALYZER.getName(), 'org.apache.lucene.analysis.standard.StandardTokenizer')).buildMixedIndex("search")
mgmt.buildIndex('text', Vertex.class).addKey(text, Mapping.TEXT.asParameter(), Parameter.of(ParameterType.TEXT_ANALYZER.getName(), 'org.apache.lucene.analysis.core.WhitespaceTokenizer')).buildMixedIndex("search")
mgmt.commit()
```
使用如上设置， JanusGraph 将使用标准标记生成器来标识属性键字符串，使用空白标识生成器来标识属性键文本。

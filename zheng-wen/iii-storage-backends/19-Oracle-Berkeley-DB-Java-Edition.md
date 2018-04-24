# 19. Oracle Berkeley DB Java Edition
> Oracle Berkeley DB Java Edition 是一个完全用 Java 开发的开源的嵌入式事务驱动存储引擎。充分利用 Java 环境的优势来简化开发和部署。Oracle Berkeley DB Java Edition 对读密集型和写密集型的工作都可提供极高的性能和并发支持。
> --[Oracle Berkeley DB Java Edition Homepage][obdb-j]

Oracle Berkeley DB Java Edition 存储后端与 JanusGraph 运行在同一个 JVM 上，并在同一台机器上提供本地持久化。因此，Oracle Berkeley DB Java Edition 需要图的所有数据都存储在本地硬盘上，需要所有会被频繁访问的图的要素装载进内存中。受限硬件商品性能，这相当于给图强加了百万级到千万级的结点数量的限制。

## 19.1. 安装 BerkeleyDB Java 版

BerkeleyDB 运行在 JVM 中所以只需要直接进行简单的配置来连接，而不需要进行额外的安装。
```java
JanusGraph g = JanusGraphFactory.build().
set("storage.backend", "berkeleyje").
set("storage.directory", "/data/graph").
open();
```

你不能在 Gremlin 终端中定义 `conf` 和 `g` 变量的类型。简单地放弃声明类型即可。

## 19.2. BerkeleyDB 特殊配置
参阅 [13. 配置参考][13]，以获取包含所有 BerkeleyDB 限定配置在内的全部 JanusGraph 配置的完整列表。

在配置 BerkeleyDB 时推荐遵循下面的 BerkeleyDB 特殊配置选项：

+ **transactions:** 允许事务(transactions) 并在进行数据库操作时探测冲突。**当心:**
虽然禁用事务可能会使性能得到提升，但如果多个 JanusGraph 实例与 BerkeleyDB 的同一实例交互，则会导致数据不一致甚至损坏数据库。
+ **cache-percentage:** JVM 堆空间 (通过 -Xmx 配置)分配给 BerkeleyDB 高速缓存的百分比。请在不导致 JanusGraph 内存问题的情况下尽可能多地分配给 BerkeleyDB。举个例子，如果 JanusGraph 只运行较短的事务，使用 80 甚至更高的值。

## 19.3. 理想用例

BerkeleyDB 存储后端最适合用于在商业硬件上运行的不超过 1 亿条边的中小型图。对于这样大小的图，BerkeleyDB 相较于分布式存储后端通常提供更高的性能。注意，BerkeleyDB 运行在一台单独的机器上，所以同时也限制了 BerkeleyDB 可以同时处理的请求数。因此，哪怕是中小型的图，BerkeleyDB 也并不适用于大量用户同时操作图的应用。

由于 BerkeleyDB 和 JanusGraph 运行在同一个 JVM 上，这个存储后端非常适合使用 JanusGraph 对应用程序代码进行单元测试。

## 19.4. 全局图操作
使用 BerkeleyDB 作为后端的 JanusGraph 支持像遍历边或顶点这样的全图操作。不过这样的操作需要扫描整个数据库，对于一个大图来说需要耗费很长的时间。

为了防止内存溢出，建议在遍历整个大图的时候禁用事务(`storage.transactions=false`)。BerkeleyDB 启用事务需要在读取数据时请求读锁，在遍历整个图时，这些读锁会额外耗费更多内存。

[obdb-j]:http://www.oracle.com/technetwork/database/berkeleydb/overview/index-093405.html
[13]:./ii-janusgraph-basics/13-config-ref.md

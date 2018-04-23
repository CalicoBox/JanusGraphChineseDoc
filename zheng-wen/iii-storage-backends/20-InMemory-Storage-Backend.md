# 20. 内存存储后端
JanusGraph 附带一个内存存储后端，可以通过以下配置使用：

```bash
storage.backend=inmemory
```

或者，可以直接在 Gremlin 控制台中打开内存中的 JanusGraph 图：
```java
graph = JanusGraphFactory.build().set('storage.backend', 'inmemory').open()
```

对于内存存储后端没有额外的配置项。正如名字(InMemory)所述，这个后端的所有数据都在内存中，关闭图或者结束 JanusGraph 的宿主进程将会不可撤销地删除图中的所有数据。对于特定的 JanusGraph 图实例而言，该后端是本地的，并且该后端不能在多个 JanusGraph 图之间共享。

## 20.1. 理想用例
内存存储后端主要用于开发时进行简单的测试 (如果不需要持久化的话) 和图的探索。内存存储后端**不**适用于生产环境，大图或高性能用例。内存存储后端也没有性能和内存上的优化。所有数据都直接存储在 Java 虚拟机分配的堆空间中。

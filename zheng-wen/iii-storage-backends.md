**目录**

16. Apache Cassandra
16.1. 本地服务器模式
16.2. 远端服务器模式
16.3. 集成 Gremlin Server 的远端服务器模式
16.4. JanusGraph 嵌入模式
16.5. Cassandra 的特殊配置
16.6. 全局 Graph 操作
16.7. 在 Amazon EC2 上部署
17. Apache HBase
17.1. 在 HBase 上安装
17.2. 在 HBase 的特殊配置
17.3. 全局 Graph 操作
17.4. 在 Amazon EC2 上部署
17.5. 管理 HBase 集群的小技巧
18. Google Cloud Bigtable
18.1. 在 Bigtable 上安装
19. Oracle Berkeley DB Java Edition
19.1. 在 BerkeleyDB JE 上安装
19.2. 在 BerkeleyDB 的特殊配置
19.3. 理想用例
19.4. 全局 Graph 操作
20. InMemory Storage Backend
20.1. 理想用例

JanusGraph 的存储层是可拔插式的。可拔插式的存储层被称为存储后端 —— 存储层是负责告知 JanusGraph 如何与存储的数据交互的软件组件。

本节文章描述了 JanusGraph 的存储后端的标准的配置和管理。

本节仅讨论 JanusGraph 附带提供的存储后端的配置。

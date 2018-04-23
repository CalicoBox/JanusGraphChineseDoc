# 18. Google Cloud Bigtable
> Cloud Bigtable 是谷歌的 NoSQL 型大型数据库服务，包括搜索，分析，地图和 Gmail 在内的多项 Google 核心业务都使用了该数据库。
> Bigtable旨在以一致的低延迟和高吞吐量处理大量工作负载。因此，无论对于操作型还是分析型的应用都是理想选择，包括物联网，用户分析和财务数据分析。
>--[Google Cloud Bigtable Homepage][bigtable]

## 18.1. 安装 Bigtable

Bigtable 为所有数据访问操作实现 HBase 接口，此外需要几个配置选项才能连接。

## 18.1.1 连接到 Bigtable

JanusGraph 使用包含以下三个配置部分的 `hbase` 后端，自定义实现的连接，运行 Bigtable 实例的 Google 云平台上的项目的 id，和你连接到的 Cloud Bigtable 的实例的 id，来实现连接至 Bigtable。

例如：
```bash
storage.backend=hbase
storage.hbase.ext.hbase.client.connection.impl=com.google.cloud.bigtable.hbase1_x.BigtableConnection
storage.hbase.ext.google.bigtable.project.id=<Google Cloud Platform project id>
storage.hbase.ext.google.bigtable.instance.id=<Bigtable instance id>
```


[bigtable]:https://cloud.google.com/bigtable/

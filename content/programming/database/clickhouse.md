---
title: "ClickHouse"
type: "docs"
weight: 2
---

[https://clickhouse.com/docs/zh](https://clickhouse.com/docs/zh)

ClickHouse是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)，由俄罗斯Yandex于2016年开源，具有高性能、高可靠性、高压缩率、高扩展性等特点。

## 存储引擎

MergeTree系列中有几种常见的表引擎：

1. MergeTree：基本的表引擎，不支持复制。
2. ReplicatedMergeTree：支持复制的表引擎，需要指定一个zookeeper集群来管理元数据和协调复制操作。
3. SummingMergeTree：在合并数据时，可以对某些列进行求和操作，以减少存储空间和提高聚合查询效率。
4. AggregatingMergeTree：在合并数据时，可以对某些列进行聚合函数操作（如avg、min、max等），以实现预计算功能。
5. CollapsingMergeTree：在合并数据时，可以根据某些列的正负符号来抵消相同记录，以实现增量更新或删除功能。

除了MergeTree系列外，clickhouse还有其他类型的表引擎：

1. Log系列：用于存储小规模且不需要排序或索引的数据。例如TinyLog、StripeLog等。
2. Memory系列：用于存储内存中临时或易变的数据。例如Memory、Set等。
3. File系列：用于从文件中读取或写入数据。例如File、URL等。
4. Integration系列：用于与其他数据库或系统集成。例如MySQL、Kafka等。
5. Distributed：用于创建分布式表，在多个节点上执行查询，并将结果汇总返回。

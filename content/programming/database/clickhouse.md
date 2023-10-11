---
title: "ClickHouse"
type: "docs"
weight: 2
---

[https://clickhouse.com/docs/zh](https://clickhouse.com/docs/zh)

ClickHouse 是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)，由俄罗斯 Yandex 于 2016 年开源，具有高性能、高可靠性、高压缩率、高扩展性等特点。

由于列式存储的特性，ClickHouse 适合聚合，单行插入更新查询性能差，建议批量操作(> 1000行)。

ClickHouse 支持一种 SQL 方言，它在许多情况下与 ANSI SQL 标准相同，具体请查看 [兼容性差异](https://clickhouse.com/docs/zh/sql-reference/ansi)。

## 表引擎

### MergeTree 系列

1. MergeTree：基本的表引擎，不支持复制。
2. ReplicatedMergeTree：支持复制的表引擎，需要指定一个 zookeeper 集群来管理元数据和协调复制操作。
3. SummingMergeTree：在合并数据时，可以对某些列进行求和操作，以减少存储空间和提高聚合查询效率。
4. AggregatingMergeTree：在合并数据时，可以对某些列进行聚合函数操作（如 avg、min、max 等），以实现预计算功能。
5. CollapsingMergeTree：在合并数据时，可以根据某些列的正负符号来抵消相同记录，以实现增量更新或删除功能。

### 其他系列

1. Log 系列：用于存储小规模且不需要排序或索引的数据。例如 TinyLog、StripeLog 等。
2. Memory 系列：用于存储内存中临时或易变的数据。例如 Memory、Set 等。
3. File 系列：用于从文件中读取或写入数据。例如 File、URL 等。
4. Integration 系列：用于与其他数据库或系统集成。例如 MySQL、Kafka 等。
5. Distributed：用于创建分布式表，在多个节点上执行查询，并将结果汇总返回。

## 数据类型

### 数值类型

Int8：有符号 8 位整数  
Int16：有符号 16 位整数  
Int32：有符号 32 位整数  
Int64：有符号 64 位整数  
UInt8：无符号 8 位整数  
UInt16：无符号 16 位整数  
UInt32：无符号 32 位整数  
UInt64：无符号 64 位整数  
Float：单精度浮点数  
Double：双精度浮点数

### 布尔类型

Boolean

### 字符串类型

String：可变长字符串  
FixedString(N)：固定长度字符串，长度 N 可以指定

### 日期和时间类型

DateTime：日期和时间（从 1970 年 1 月 1 日开始）  
DateTime64(3)：带有毫秒的日期和时间（从 1970 年 1 月 1 日开始）  
Date：日期（从 1970 年 1 月 1 日开始）  
Time：时间（从 00:00:00 开始）  
Timestamp(3)：时间戳（从 1970 年 1 月 1 日开始），带有毫秒

### 数组类型

Array(T)，其中 T 是数组元素的数据类型，例如 Array(Int32)，表示一个包含 Int32 类型元素的数组。

### 字典类型

Dictionary(T)，其中 T 是字典键和值的数据类型，例如 Dictionary(String, Int32)，表示一个以 String 为键，Int32 为值的字典。

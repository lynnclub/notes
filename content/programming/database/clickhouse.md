---
title: "ClickHouse"
type: "docs"
weight: 2
---

[https://clickhouse.com/docs/zh](https://clickhouse.com/docs/zh)

ClickHouse是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)，由俄罗斯Yandex于2016年开源，具有高性能、高可靠性、高压缩率、高扩展性等特点。

## 存储引擎

### MergeTree系列

1. MergeTree：基本的表引擎，不支持复制。
2. ReplicatedMergeTree：支持复制的表引擎，需要指定一个zookeeper集群来管理元数据和协调复制操作。
3. SummingMergeTree：在合并数据时，可以对某些列进行求和操作，以减少存储空间和提高聚合查询效率。
4. AggregatingMergeTree：在合并数据时，可以对某些列进行聚合函数操作（如avg、min、max等），以实现预计算功能。
5. CollapsingMergeTree：在合并数据时，可以根据某些列的正负符号来抵消相同记录，以实现增量更新或删除功能。

### 其他系列

1. Log系列：用于存储小规模且不需要排序或索引的数据。例如TinyLog、StripeLog等。
2. Memory系列：用于存储内存中临时或易变的数据。例如Memory、Set等。
3. File系列：用于从文件中读取或写入数据。例如File、URL等。
4. Integration系列：用于与其他数据库或系统集成。例如MySQL、Kafka等。
5. Distributed：用于创建分布式表，在多个节点上执行查询，并将结果汇总返回。

## 数据类型

### 数值类型

Int8：有符号8位整数  
Int16：有符号16位整数  
Int32：有符号32位整数  
Int64：有符号64位整数  
UInt8：无符号8位整数  
UInt16：无符号16位整数  
UInt32：无符号32位整数  
UInt64：无符号64位整数  
Float：单精度浮点数  
Double：双精度浮点数  

### 布尔类型

Boolean

### 字符串类型

String：可变长字符串  
FixedString(N)：固定长度字符串，长度N可以指定  

### 日期和时间类型

DateTime：日期和时间（从1970年1月1日开始）  
DateTime64(3)：带有毫秒的日期和时间（从1970年1月1日开始）  
Date：日期（从1970年1月1日开始）  
Time：时间（从00:00:00开始）  
Timestamp(3)：时间戳（从1970年1月1日开始），带有毫秒  

### 数组类型

Array(T)，其中T是数组元素的数据类型，例如Array(Int32)，表示一个包含Int32类型元素的数组。

### 字典类型

Dictionary(T)，其中T是字典键和值的数据类型，例如Dictionary(String, Int32)，表示一个以String为键，Int32为值的字典。

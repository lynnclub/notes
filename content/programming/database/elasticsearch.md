---
title: "ElasticSearch"
type: "docs"
weight: 2
---

官方文档  
[Quick start | Elasticsearch Guide [7.17] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/getting-started.html)

ElasticSearch 是一个开源的实时分布式搜索引擎，面向文档（document oriented），存储了完整内容，可以对文档（而非成行成列的数据）做索引、搜索、排序、过滤，适用于全文检索、结构化搜索、分析以及这三个功能的组合。

可视化界面 kibana  
浏览器插件 elasticvue

## 概念

1. index 索引，类似数据库
2. type 类型，类似表
3. document 文档，类似行
4. field 字段
5. mapping 映射，相当于主键外键

Relational DB ‐> Database ‐> Table ‐> Row ‐> Column  
Elasticsearch ‐> Index ‐> Type ‐> Document ‐> Field

### RESTful API

创建索引 PUT /{index}  
创建映射 PUT /{index}/{type}/\_mapping  
删除索引 DELETE /{index}  
创建修改文档 POST /{index}/{type}/{id}  
删除文档 DELETE /{index}/{type}/{id}  
查询文档 GET /{index}/{type}/{id}  
条件查询 POST /{index}/{type}/\_search  

## 安装

```shell
docker network create somenetwork
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag
```

## 数据类型

ElasticSearch 支持多种不同的数据类型，这些数据类型用于定义索引中字段的性质和行为。

### Text

`text` 数据类型用于存储可搜索的文本数据。文本字段通常会被分词，以支持全文搜索和分析。

示例：`"description": "This is a text field"`

### Keyword

`keyword` 数据类型用于存储结构化文本数据，通常不会被分词。它们用于精确匹配、排序和聚合操作。

示例：`"status": "published"`

### Numeric

ElasticSearch 支持不同类型的数值字段，包括整数和浮点数。这些字段用于存储数值数据。

示例：`"age": 30`, `"price": 19.99`

### Date

`date` 数据类型用于存储日期和时间信息。它们支持日期范围查询和聚合操作。

示例：`"timestamp": "2023-09-07T12:00:00"`

### Boolean

`boolean` 数据类型用于存储布尔值（true 或 false）。

示例：`"is_active": true`

### Object

`object` 数据类型用于存储嵌套的 JSON 对象，可以包含多个字段。

示例：`"user": {"name": "John", "age": 30}`

### Array

`array` 数据类型用于存储数组或列表。在 ElasticSearch 中，数组中的元素通常具有相同的数据类型。

示例：`"tags": ["tag1", "tag2", "tag3"]`

### Geo

ElasticSearch 支持地理空间数据类型，用于存储地理位置信息（如经纬度坐标点）。

示例：`"location": {"lat": 40.7128, "lon": -74.0060}`

### Binary

`binary` 数据类型用于存储二进制数据，例如图像或文档。

示例：`"document": <binary data>`

### IP

`ip` 数据类型用于存储 IPv4 或 IPv6 地址。

示例：`"ip_address": "192.168.1.1"`

这些数据类型允许您定义索引中不同字段的性质和用途，以满足各种查询和分析需求。在创建索引映射时，选择适当的数据类型对于数据的正确存储和查询至关重要。

## 索引

## 模板

## 策略

30 天过期策略

```json
PUT /_ilm/policy/30-days-default
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "30d"
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {
            "delete_searchable_snapshot": true
          }
        }
      },
      "warm": {
        "min_age": "2d",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          },
          "shrink": {
            "number_of_shards": 1
          }
        }
      }
    },
    "_meta": {
      "description": "built-in ILM policy using the hot and warm phases with a retention of 30 days",
      "managed": true
    }
  }
}
```

## 搜索

ElasticSearch 提供了多种 API 来执行不同类型的搜索操作，包括全文搜索、聚合、过滤等。

### 全文搜索

使用`match`查询来执行全文搜索，搜索指定字段中包含特定关键字的文档。

```json
GET /your_index/_search
{
  "query": {
    "match": {
      "your_field": "your_keyword"
    }
  }
}
```

### 精确搜索

使用`term`查询来执行精确搜索，搜索指定字段中包含精确值的文档。

```json
GET /your_index/_search
{
  "query": {
    "term": {
      "your_field": "exact_value"
    }
  }
}
```

### 范围搜索

使用`range`查询来执行范围搜索，搜索指定字段中在一定范围内的文档。

```json
GET /your_index/_search
{
  "query": {
    "range": {
      "your_field": {
        "gte": "start_value",
        "lte": "end_value"
      }
    }
  }
}
```

### 布尔查询

使用布尔查询组合多个查询条件，例如`must`、`should`和`must_not`。

```json
GET /your_index/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "field1": "value1" } },
        { "range": { "field2": { "gte": "value2" } } }
      ],
      "should": [
        { "term": { "field3": "value3" } }
      ],
      "must_not": [
        { "match": { "field4": "value4" } }
      ]
    }
  }
}
```

### 聚合

使用聚合来获取数据的汇总信息，如平均值、总和、最大值、最小值等。

```json
GET /your_index/_search
{
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

### 过滤

使用过滤器来排除不符合特定条件的文档。

```json
GET /your_index/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "category": "electronics" } },
        { "range": { "price": { "gte": 500 } } }
      ]
    }
  }
}
```

这些只是一些 ElasticSearch 搜索 API 的示例。您可以根据您的需求和数据结构来构建不同类型的查询和聚合。要执行这些 API 请求，您可以使用 HTTP 客户端（如 Curl 或 HTTP 库），或者使用 ElasticSearch 的官方客户端库（如 ElasticSearch-PHP）来编写和执行 API 请求。

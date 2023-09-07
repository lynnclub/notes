
Elasticsearch提供了多种API来执行不同类型的搜索操作，包括全文搜索、聚合、过滤等。以下是一些常用的Elasticsearch搜索API示例：

1. **全文搜索**：使用`match`查询来执行全文搜索，搜索指定字段中包含特定关键字的文档。

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

2. **精确搜索**：使用`term`查询来执行精确搜索，搜索指定字段中包含精确值的文档。

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

3. **范围搜索**：使用`range`查询来执行范围搜索，搜索指定字段中在一定范围内的文档。

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

4. **布尔查询**：使用布尔查询组合多个查询条件，例如`must`、`should`和`must_not`。

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

5. **聚合**：使用聚合来获取数据的汇总信息，如平均值、总和、最大值、最小值等。

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

6. **过滤**：使用过滤器来排除不符合特定条件的文档。

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

这些只是一些Elasticsearch搜索API的示例。您可以根据您的需求和数据结构来构建不同类型的查询和聚合。要执行这些API请求，您可以使用HTTP客户端（如Curl或HTTP库），或者使用Elasticsearch的官方客户端库（如Elasticsearch-PHP）来编写和执行API请求。


Elasticsearch支持多种不同的数据类型，这些数据类型用于定义索引中字段的性质和行为。以下是一些常见的Elasticsearch数据类型：

1. **Text 数据类型：**
   - `text` 数据类型用于存储可搜索的文本数据。文本字段通常会被分词，以支持全文搜索和分析。
   - 示例：`"description": "This is a text field"`

2. **Keyword 数据类型：**
   - `keyword` 数据类型用于存储结构化文本数据，通常不会被分词。它们用于精确匹配、排序和聚合操作。
   - 示例：`"status": "published"`

3. **Numeric 数据类型：**
   - Elasticsearch支持不同类型的数值字段，包括整数和浮点数。这些字段用于存储数值数据。
   - 示例：`"age": 30`, `"price": 19.99`

4. **Date 数据类型：**
   - `date` 数据类型用于存储日期和时间信息。它们支持日期范围查询和聚合操作。
   - 示例：`"timestamp": "2023-09-07T12:00:00"`

5. **Boolean 数据类型：**
   - `boolean` 数据类型用于存储布尔值（true或false）。
   - 示例：`"is_active": true`

6. **Object 数据类型：**
   - `object` 数据类型用于存储嵌套的JSON对象，可以包含多个字段。
   - 示例：`"user": {"name": "John", "age": 30}`

7. **Array 数据类型：**
   - `array` 数据类型用于存储数组或列表。在Elasticsearch中，数组中的元素通常具有相同的数据类型。
   - 示例：`"tags": ["tag1", "tag2", "tag3"]`

8. **Geo 数据类型：**
   - Elasticsearch支持地理空间数据类型，用于存储地理位置信息（如经纬度坐标点）。
   - 示例：`"location": {"lat": 40.7128, "lon": -74.0060}`

9. **Binary 数据类型：**
   - `binary` 数据类型用于存储二进制数据，例如图像或文档。
   - 示例：`"document": <binary data>`

10. **IP 数据类型：**
    - `ip` 数据类型用于存储IPv4或IPv6地址。
    - 示例：`"ip_address": "192.168.1.1"`

这些数据类型允许您定义索引中不同字段的性质和用途，以满足各种查询和分析需求。在创建索引映射时，选择适当的数据类型对于数据的正确存储和查询至关重要。


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
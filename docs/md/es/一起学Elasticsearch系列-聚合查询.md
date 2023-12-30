本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)

[TOC]

聚合查询是 Elasticsearch 中一种强大的数据分析工具，用于从索引中提取和计算有关数据的统计信息。聚合查询可以执行各种聚合操作，如计数、求和、平均值、最小值、最大值、分组等，以便进行数据汇总和分析。

下面是一些常见的聚合查询类型：

- **Metric Aggregations（指标聚合）**：这些聚合操作返回基于字段值的度量结果，如求和、平均值、最小值、最大值等。常见的指标聚合包括 Sum、Avg、Min、Max、Stats 等。
- **Bucket Aggregations（桶聚合）**：类比SQL中的group by，主要用于统计不同类型数据的数量，这些聚合操作将文档划分为不同的桶（buckets），并对每个桶中的文档进行聚合计算。常见的桶聚合包括 Terms（按字段值分组）、Date Histogram（按时间间隔分组）、Range（按范围分组）等。
- **Pipeline Aggregations（管道聚合）**：这些聚合操作通过在其他聚合结果上执行额外的计算来产生新的聚合结果。例如，使用 Moving Average 聚合可以计算出移动平均值。

聚合查询通常与查询语句结合使用，可以在查询结果的基础上进行进一步的数据分析和统计。聚合查询语法使用 JSON 格式，可以通过 Elasticsearch 的 REST API 或各种客户端库进行发送和解析。

聚合查询支持嵌套，即一个聚合内部可以包含别的子聚合，从而实现非常复杂的数据挖掘和统计需求。

在ES中，用于进行聚合的字段可以是exact value也可以是分词字段，对于分词字段，可以使用特定的聚合操作来进行分组聚合，例如Terms Aggregation、Date Histogram Aggregation等。

对于text字段的聚合，可以通过开启fielddata来实现，但通常不建议这样做，因为fielddata会将聚合使用的数据结构从磁盘（doc_values）转换为堆内存（field_data），在处理大量数据时容易导致内存溢出（OOM）问题。

如果需要在text字段上执行聚合，可以考虑在该字段上添加.keyword子字段，并使用该子字段进行聚合操作，以获得更准确的结果。

## doc_values & fielddata

在 Elasticsearch 中，聚合操作主要依赖于 doc_values 或 fielddata 来进行。

- **Doc Values（文档值）**：Doc Values 是一种以列式存储格式保存字段值的数据结构，它用于支持快速的聚合、排序和统计操作。Doc Values 在磁盘上存储，并被加载到 JVM 堆内存中进行计算。它们适用于精确值（如 keyword 类型）和数字类型的字段，在大多数情况下是默认启用的。
- **Fielddata（字段数据）**：Fielddata 是一种将字段值加载到堆内存中的数据结构，它用于支持复杂的文本分析和聚合操作。Fielddata 适用于文本类型的字段，例如 text 类型，因为它们需要进行分词和分析。但是，由于 Fielddata 需要大量的堆内存资源，特别是在处理大数据集时，容易导致内存溢出（OOM）的问题，因此不建议随意启用。

在设计索引时，需要根据字段类型和使用场景的不同，合理选择是否启用 Doc Values 或 Fielddata，以平衡性能和资源消耗的需求。

当执行聚合操作时，Elasticsearch 需要访问所有匹配文档的字段值。对于非文本字段，默认情况下Elasticsearch 使用 doc values 来实现。对于文本字段，必须首先启用 fielddata。然而，由于 fielddata 占用大量内存，Elasticsearch 默认禁用了它。

如果你确实需要对一个文本字段启用 fielddata（虽然大多数场景下不推荐这么做，因为可能导致内存消耗过大），你可以通过更新映射（mapping）来实现。

以下是如何在 `my_field` 字段上启用 fielddata 的示例：

```JSON
PUT my_index/_mapping

{
  "properties": {
    "my_field": { 
      "type":     "text",
      "fielddata": true
    }
  }
}
```

**注意：更改 fielddata 设置只会影响新的数据，已经索引的数据不会受到更改。如果你想让更改生效，需要重新索引（reindex）你的数据**

另外，一般情况下，建议使用 mapping 中的 `keyword` 类型来进行聚合、排序或脚本，而不是启用 `text` 类型的 fielddata。这是因为 `keyword` 类型字段默认开启了 doc values，比在 `text` 上启用 fielddata 更加高效且节省内存。

## multi-fields

在 Elasticsearch 中，一个字段有可能是 multi-fields（多字段）类型，这意味着同一份数据可以被索引为不同类型的字段。常见的情况就是，一个字段既被索引为 `text` 类型用于全文搜索，又被索引为 `keyword` 类型用于精确值搜索、排序和聚合。

当你在一个字段名后面加上 `.keyword`（例如 `field.keyword`），这说明你是在引用这个字段的 `keyword` 子字段。这个 `keyword` 子字段在索引时并不会被分词器拆分成单独的词条，而是作为一个完整的字符串被存储。这样，你就可以对这个字段进行精确值匹配、排序或者聚合操作。

举例来说，如果你有一个 `message` 字段并且想要对其进行聚合，你应该使用 `message.keyword` 而非 `message`。因为如果你直接对 `message` 进行聚合，Elasticsearch 就会尝试对每一个独立的词条进行聚合，而不是对整个字段值进行聚合。

如果你的字段没有 `.keyword` 子字段，那可能是在定义 mapping 时没有包含这一部分，或者这个字段的类型本身就是 `keyword`。

## 分桶聚合

分桶（Bucket）聚合是一种特殊类型的聚合，它将输入文档集合中的文档分配到一个或多个桶中，每个桶都对应于一个键（key）。

下面是一些常用的分桶聚合类型：

- `terms`：基于文档中某个字段的值，将文档分组到各个桶中。
- `date_histogram`：基于日期字段，将文档按照指定的时间间隔分组到各个桶中。
- `histogram`：基于数值字段，将文档按照指定的数值范围分组到各个桶中。
- `range`：根据设置的范围，将数据分为不同的桶。

以下是一个使用 `terms` 分桶聚合的例子：

假设你有一个包含博客文章的 `blog` 索引，你想知道每个作者写了多少篇文章，可以使用以下查询：

```JSON
GET /blog/_search
{
  "size": 0,
  "aggs": {
    "authors": {
      "terms": { "field": "author.keyword" }
    }
  }
}
```

在这个查询中：

- `size: 0` 表示我们只对聚合结果感兴趣，不需要返回任何具体的搜索结果。
- `"aggs"` (或者 `"aggregations"`) 块定义了我们的聚合。
- `"authors"` 是我们自己为这个聚合命名的标签，你可以用任何你喜欢的标签名。
- `"terms": { "field": "author.keyword" }` 定义了我们要进行聚合的方式和字段。这里，我们告诉 Elasticsearch 使用 `terms` 聚合，并且使用 `author.keyword` 字段的值作为分桶的依据。

Elasticsearch 将返回一个包含每个作者以及他们所写的文章数量的列表。注意，由于 Elasticsearch 默认只返回前十个桶，如果你的数据中有更多的作者，可能需要设置 `size` 参数来获取更多的结果。

### Histogram

`histogram` 是桶聚合的一种类型，它可以按照指定的间隔将数字字段的值划分为一系列桶。每个桶代表了这个区间内的所有文档。

以下是一个例子，我们根据价格字段创建一个间隔为 50 的直方图：

```JSON
GET /products/_search
{
  "size": 0,
  "aggs" : {
    "prices" : {
      "histogram" : {
        "field" : "price",
        "interval" : 50
      }
    }
  }
}
```

在这个例子中，“prices” 是一个 histogram 聚合，它以 50 为间隔将产品的价格划分为一系列的桶。

## 指标聚合

在 Elasticsearch 中，指标聚合是对数据进行统计计算的一种方式，例如求和、平均值、最小值、最大值等。以下是一些常用的指标聚合类型：

- `avg`：计算字段的平均值。
- `sum`：计算字段的总和。
- `min`：查找字段的最小值。
- `max`：查找字段的最大值。
- `count`：计算匹配文档的数量。
- `stats`：提供了 count、sum、min、max 和 avg 的基本统计。

下面是一个示例，假设我们有一个包含售卖商品的 “sales” 索引，我们想要知道所有销售记录中的平均价格，可以使用 `avg` 聚合如下操作：

```JSON
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "average_price": {
      "avg": { "field": "price" }
    }
  }
}
```

### Percentiles

`percentiles` 是指标聚合的一种，它用于计算数值字段的百分位数。给定一个列表百分比，Elasticsearch 可以计算每个百分比下的数值。

以下是一个例子，我们计算价格字段的 1st, 5th, 25th, 50th, 75th, 95th, and 99th 百分位数：

```JSON
GET /products/_search
{
  "size": 0,
  "aggs" : {
    "price_percentiles" : {
      "percentiles" : {
        "field" : "price",
        "percents" : [1, 5, 25, 50, 75, 95, 99]
      }
    }
  }
}
```

在这个例子中，“price_percentiles” 是一个 percentiles 聚合，它计算了价格在各个百分位点的数值。

注意，对于大数据集，计算精确的百分位数可能需要消耗大量资源。因此，Elasticsearch 默认使用一个名为 `TDigest` 的算法来提供近似的计算结果，同时还能保持内存使用的可控性。

### cardinality

如果你想在 Elasticsearch 中进行去重操作，可以使用 `terms` 聚合加上 `cardinality` 聚合。这是一个示例，假设我们有一个包含user_id的 "users" 索引，并且我们想要知道有多少唯一的 user_id：

```JSON
GET /users/_search
{
  "size": 0,
  "aggs": {
    "distinct_user_ids": {
      "cardinality": {
        "field": "user_id.keyword"
      }
    }
  }
}
```

在这个查询中：

- `"distinct_user_ids"` 是我们自己为这个聚合命名的标签。
- `"cardinality": { "field": "user_id.keyword" }` 使用了 `cardinality` 聚合，该聚合会返回指定字段（在这里是 `user_id.keyword`）的不同值的数量。

Elasticsearch 将返回一个结果，告诉我们有多少个不同的 user_id。请注意，`cardinality` 聚合可能并不总是完全精确，特别是对于大型数据集，因为它在内部使用了一种叫做 `HyperLogLog` 的算法来近似计算基数，这种算法会在保持内存消耗相对较小的情况下提供接近准确的结果。如果你需要完全精确的结果，可能需要考虑其他方法，例如使用脚本或者将数据导出到外部系统进行处理。

## 管道聚合

在 Elasticsearch 中，管道聚合（pipeline aggregations）是指这样一种聚合：它以其他聚合的结果作为输入，并进行进一步处理。

常见的管道聚合包括：

- `avg_bucket`
- `sum_bucket`
- `min_bucket`
- `max_bucket`
- `stats_bucket`
- `extended_stats_bucket`
- `percentiles_bucket`

这些都是 bucket 级别的管道聚合，它们会在一组数据桶上操作。

下面给出一个示例，假设我们有一个销售记录索引 "sales"，每个销售记录都有售价 "price" 和销售日期 "date" 字段。如果我们想要计算每月平均销售价格，并找出所有月份中平均价格最高的月份，可以使用 date_histogram 聚合加上 avg 以及 max_bucket 聚合来实现：

```JSON
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "avg_price": {
          "avg": { "field": "price" }
        }
      }
    },
    "max_avg_price": {
      "max_bucket": {
        "buckets_path": "sales_per_month>avg_price"
      }
    }
  }
}
```

在这个查询中：

- `"sales_per_month"` 是一个按月聚合销售记录的 date_histogram 聚合。
- `"avg_price"` 是一个嵌套在 `"sales_per_month"` 下的 avg 聚合，用于计算每月的平均销售价格。
- `"max_avg_price"` 是一个 max_bucket 聚合，它会找出 `"sales_per_month"` 中所有子桶的 `"avg_price"` 最大值。

注意到 `"max_avg_price"` 中的 `"buckets_path": "sales_per_month>avg_price"`。`buckets_path` 参数指定了此管道聚合的输入来源，`>` 符号表示路径层次，即先取 `"sales_per_month"` 聚合的结果，再取其中的 `"avg_price"` 聚合的结果作为输入。

返回的结果中会包含每个月的平均销售价格，以及所有月份中平均销售价格的最大值。

## 嵌套聚合

嵌套聚合就是在聚合内使用聚合，在 Elasticsearch 中，嵌套聚合通常用于处理 nested 类型的字段。nested 类型允许你将一个文档中的一组对象作为独立的文档进行索引和查询，这对于拥有复杂数据结构（例如数组或列表中的对象）的场景非常有用。

假设我们有一个 `users` 索引，每个 user 文档都有一个 `purchases` 字段，该字段是一个列出用户所有购买记录的数组，每个购买记录包含 `product_id` 和 `price`。如果我们想要找出价格超过 100 的所有产品的 ID，可以使用 nested 聚合：

```JSON
GET /users/_search
{
  "size": 0,
  "aggs": {
    "all_purchases": {
      "nested": {
        "path": "purchases"
      },
      "aggs": {
        "expensive_purchases": {
          "filter": { "range": { "purchases.price": { "gt": 100 } } },
          "aggs": {
            "product_ids": { "terms": { "field": "purchases.product_id" } }
          }
        }
      }
    }
  }
}
```

在这个查询中：

- `"all_purchases"` 是一个 nested 聚合，指定了 nested 对象的路径 `purchases`。
- `"expensive_purchases"` 是一个嵌套在 `"all_purchases"` 下的 filter 聚合，它会过滤出 `price` 大于 100 的购买记录。
- `"product_ids"` 是一个嵌套在 `"expensive_purchases"` 下的 terms 聚合，它会提取出所有满足条件的 `product_id`。

返回的结果将包含所有 `price` 大于 100 的产品的 ID 列表。

请注意，在处理 nested 数据时，你需要确保 mapping 中相应的字段已经被设置为 nested 类型，否则该查询可能无法按预期工作。

## 基于查询结果的聚合 & 基于聚合结果的查询

基于查询结果的聚合：在这种情况下，我们首先执行一个查询，然后对查询结果进行聚合。

例如，如果我们要查询所有包含某关键字的文档，并计算它们的平均价格，可以这样做：

```JSON
GET /products/_search
{
  "query": {
    "match": {
      "description": "laptop"
    }
  },
  "aggs": {
    "average_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

在上述例子中，我们首先通过 `match` 查询找到描述中包含 "laptop" 的所有产品，然后对这些产品的价格进行平均值聚合。

基于聚合结果的查询：这种情况下，我们先执行聚合，然后基于聚合的结果执行过滤操作。

这通常用于在聚合结果中应用一些额外的过滤条件。例如，如果我们想对所有产品进行销售数量聚合，然后从结果中过滤出销售数量大于10的产品，可以这样做：

```JSON
GET /sales/_search
{
  "size": 0,
  "aggs": {
    "sales_per_product": {
      "terms": {
        "field": "product_id"
      }
    }
  },
  "post_filter": {
    "bucket_selector": {
      "buckets_path": {
        "salesCount": "sales_per_product._count"
      },
      "script": {
        "source": "params.salesCount > 10"
      }
    }
  }
}
```

在上述例子中，我们首先执行了一个 `terms` 聚合，按产品ID汇总销售记录。然后我们使用 `bucket_selector` post-filter 进一步筛选出销售数量大于10的桶（每个桶对应一个产品）。

## 聚合排序

### count

在 Elasticsearch 中，聚合排序允许你基于某一聚合的结果来对桶进行排序。例如，你可能希望查看销售量最高的10个产品，可以使用 `terms` 聚合以及其 `size` 和 `order` 参数来实现：

```JSON
GET /sales/_search

{
  "size": 0,
  "aggs": {
    "top_products": {
      "terms": {
        "field": "product_id",
        "size": 10,
        "order": { "_count": "desc" }
      }
    }
  }
}
```

在这个例子中，`top_products` 是一个 `terms` 聚合，用于按 `product_id` 对销售记录进行分组。

- `"size": 10` 的意思是只返回销售量最高的前10个产品（即只返回前10个桶）。
- `"order": { "_count": "desc" }` 表示按桶中文档的数量（也就是销售量）降序排序。`_count` 是一个内置的排序键，代表桶中文档的数量。

返回的结果将包含销售量最高的前10个产品的 ID 列表。

### term

`_term` 在 Elasticsearch 的聚合排序中用来指定按照词条（即桶的键）来排序。

```JSON
GET /sales/_search

{
  "size": 0,
  "aggs": {
    "products": {
      "terms": {
        "field": "product_id",
        "order": { "_term": "asc" }
      }
    }
  }
}
```

在这个例子中，通过 `"order": { "_term": "asc" }` 指定了按照 `product_id` 的值升序排序这些桶。

返回的结果将包含按照 `product_id` 升序排列的产品 ID 列表，每个产品 ID 对应一个桶，并且每个桶内包含对应产品的销售记录。

需要注意的是，在新版本的 Elasticsearch 中（7.0 以后），`_term` 已经被 `key` 替代用于排序。

```JSON
GET /sales/_search

{
  "size": 0,
  "aggs" : {
    "products" : {
      "terms" : {
        "field" : "product_id",
        "order" : { "_key" : "asc" }
      }
    }
  }
}
```
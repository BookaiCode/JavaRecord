本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)


[TOC]

Elasticsearch 提供了 `_mget` 和 `_bulk` API 来执行批量操作，它允许你在单个 HTTP 请求中进行多个索引获取/删除/更新/创建操作。这种方法比发送大量的单个请求更有效率。

## 基于 mget 的批量查询

mget（multi-get） API用于批量检索多个文档。它可以通过一次请求获取多个文档的内容，并提供了一些参数来控制检索行为。下面是mget API的请求示例、响应示例以及一些常用参数的含义：

请求示例：

```json
POST /_mget
{
  "docs": [
    {
      "_index": "my_index",
      "_id": "1"
    },
    {
      "_index": "my_index",
      "_id": "2"
    }
  ]
}
```

上述示例中，我们向`my_index`索引发出一个mget请求，要求检索id为1和2的两个文档。

响应示例：

```json
{
  "docs": [
    {
      "_index": "my_index",
      "_id": "1",
      "_source": {
        "field1": "value1",
        "field2": "value2"
      },
      "found": true
    },
    {
      "_index": "my_index",
      "_id": "2",
      "_source": {
        "field1": "value3",
        "field2": "value4"
      },
      "found": true
    }
  ]
}
```

上述示例中，响应结果中包含了每个请求文档的结果。每个结果都有`_source`字段，其中包含了文档的实际内容。同时，还有一个`found`字段指示是否找到了对应的文档。

以下是一些常用的mget参数及其含义：

- `_index`：指定索引名称，表示要检索的文档所在的索引。
- `_id`：指定文档的唯一标识符，用于唯一确定要检索的文档。
- `_source`：设置为false可以禁用返回文档的内容，只返回元数据信息。默认为true，返回完整的文档内容。
- `stored_fields`：指定要返回的存储字段（stored fields），用逗号分隔多个字段名。这些字段必须在映射中设置了`store`属性才能被返回。
- `_source_includes`和`_source_excludes`：允许选择性地包含或排除返回文档中的特定字段，以控制返回结果的内容。
- `routing`：指定文档的路由值，用于决定将文档存储在哪个分片上。如果索引设置了自定义路由策略，必须提供正确的路由值。

这些参数可以通过请求体中的每个文档对象进行设置，例如：

```json
{
  "_index": "my_index",
  "_id": "1",
  "_source": false,
  "stored_fields": "field1"
}
```

## 基于 bulk 的批量增删改

bulk API允许执行批量的索引、删除和更新操作。它可以通过一次请求同时处理多个操作，提高数据的写入效率。

bulk API中，请求是通过一行一行的JSON数据进行定义的。每个操作（索引、删除、更新）都需要按照特定格式写在一行中。

格式要求如下：

1. 每个操作必须以一个操作描述符开始，例如`index`、`delete`、`update`。
2. 操作描述符后面必须跟着一个JSON对象，该对象包含操作所需的参数和数据。
3. 每个操作及其对应的JSON数据必须用换行符分隔。

示例：

```
{操作描述符}
{JSON数据}
{操作描述符}
{JSON数据}
...
```

注意以下几点：

- 请求数据中的每一行都必须是有效的JSON格式，且不能有多余的空格或换行符。
- 在一个bulk请求中，可以包含任意数量的操作。
- bulk请求可以一次性执行多个操作，提高效率，但也会增加单个请求的复杂性和长度。

下面是bulk API的请求示例、响应示例以及一些常用参数的含义。

请求示例：

```json
POST /_bulk
{"index":{"_index":"my_index","_id":"1"}}
{"field1":"value1","field2":"value2"}
{"delete":{"_index":"my_index","_id":"2"}}
{"update":{"_index":"my_index","_id":"3"}}
{"doc":{"field1":"updated_value"}}
```

上述示例展示了一个包含三个操作的bulk请求：

1. 索引（index）操作：将一个新文档插入到`my_index`索引中，指定唯一标识符为1。
2. 删除（delete）操作：从`my_index`索引中删除唯一标识符为2的文档。
3. 更新（update）操作：将`my_index`索引中唯一标识符为3的文档进行更新。

响应示例：

```json
{
  "took": 15,
  "errors": false,
  "items": [
    {
      "index": {
        "_index": "my_index",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "status": 201
      }
    },
    {
      "delete": {
        "_index": "my_index",
        "_id": "2",
        "_version": 2,
        "result": "deleted",
        "status": 200
      }
    },
    {
      "update": {
        "_index": "my_index",
        "_id": "3",
        "_version": 2,
        "result": "updated",
        "status": 200
      }
    }
  ]
}
```

上述示例展示了每个操作的响应结果。每个结果都包含了与对应操作相关的元数据信息，如索引名称、文档ID、版本号、操作结果（如创建、删除、更新）以及HTTP状态码。

以下是一些常用的bulk参数及其含义：

- `index`：指定要执行索引操作的索引名称和文档ID。
- `delete`：指定要执行删除操作的索引名称和文档ID。
- `update`：指定要执行更新操作的索引名称和文档ID。
- `doc`：在更新操作中，用于指定要更新的字段和值。
- `retry_on_conflict`：在并发更新时，设置重试次数以处理冲突，默认为0，表示不进行重试。
- `pipeline`：指定在索引操作期间使用的管道ID，用于预处理文档。

这些参数需要在每个操作的请求行中进行设置，例如：

```json
{"index":{"_index":"my_index","_id":"1","pipeline":"my_pipeline"}}
```

## filter_path

在 Elasticsearch 中，`filter_path`参数用于过滤返回的响应内容，可以用于减小 Elasticsearch 返回的数据量。当你指明一个或多个路径时，返回的 JSON 对象就只会包含这些路径下的键，它接收一个逗号分隔的列表，其中包含了你想要返回的 JSON 对象内的路径。这个参数支持通配符（`*`）匹配和数组元素（`[]`）匹配。列如：

```
POST /_bulk?filter_path=items.*.error
```

上述请求中的 `filter_path=items.*.error` 会让 Elasticsearch 仅返回 `_bulk` API 调用结果中的错误信息。`items.*.error` 这个路径表示，在返回的响应中，匹配到所有存在 `error` 字段的 `items`。

这样做有两个主要好处：

1. 它可以提升 Elasticsearch 的性能，因为少量的数据意味着更快的序列化和反序列化。
2. 它可帮助你聚焦于感兴趣的部分，不必处理无关的数据。

请注意，`*` 是通配符，代表任何值。

以下是一些其他 `filter_path` 的示例：

- `filter_path=took`: 这个请求仅返回执行请求所花费的时间（以毫秒为单位）。
- `filter_path=items._id,items._index`: 这个请求仅返回每个 item 的 `_id` 和 `_index` 字段。
- `filter_path=items.*.error`: 这个请求会返回所有包含 `error` 字段的 items。
- `filter_path=hits.hits._source`: 这个请求仅返回搜索结果中的原始文档内容。
- `filter_path=_shards, hits.total`: 这个请求返回关于 `shards` 的信息和命中的总数。
- `filter_path=aggregations.*.value`: 这个请求仅返回每个聚合的值。

请注意，如果你在 `filter_path` 中指定了多个字段，你需要使用逗号将它们分隔开。
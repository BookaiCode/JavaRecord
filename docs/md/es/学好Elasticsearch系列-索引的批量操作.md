[TOC]

Elasticsearch 提供了 _bulk API 来执行批量操作，它允许你在单个 HTTP 请求中进行多个索引/删除/更新/创建操作。这种方法比发送大量的单个请求更有效率。

## 基于mget的批量查询

mget(多文档获取)是Elasticsearch中提供的一个API，用于一次性从同一个索引或者不同索引中检索多个文档。

例子一：

以下是一个Elasticsearch的`mget`（多文档获取）操作示例。在这个示例中，我们将获取索引 `test-index` 中具有特定ID的多个文档。

```Java
GET /test-index/_mget
{
  "ids": ["1", "2"]
}
```

在上述请求中，我们正在获取ID为 "1" 和 "2" 的文档。

例子二：

你也可以在不同的索引中获取文档，只需指定每个文档的 `_index` 和 `_id`：

```Java
GET /_mget
{
  "docs": [
    {
      "_index": "test-index",
      "_id": "1"
    },
    {
      "_index": "another-index",
      "_id": "2"
    }
  ]
} 
```

在这个请求中，我们从 "test-index" 索引获取ID为 "1" 的文档，并从 "another-index" 索引获取ID为 "2" 的文档。

例子三：

在以下的Elasticsearch `mget`（多文档获取）例子中，我们将从两个不同的索引获取文档，并且只返回特定的字段：

```Java
GET /_mget
{
  "docs": [
    {
      "_index": "test-index-1",
      "_id": "1",
      "_source": ["field1", "field2"]
    },
    {
      "_index": "test-index-2",
      "_id": "2",
      "_source": "field3"
    }
  ]
}
```

在这个请求中，我们从 "test-index-1" 索引获取ID为 "1" 的文档，并只返回 "field1" 和 "field2" 字段。同时，我们从 "test-index-2" 索引获取ID为 "2" 的文档，并只返回 "field3" 字段。

源过滤 (`_source`) 可以用来限制返回的字段。你可以提供一个字段的列表，或者一个单独的字段。注意，如果你请求的字段不存在，它将不会出现在响应中。

## 基于bulk的批量增删改

bulk基本格式如下：

```Java
POST /<index>/_bulk 
{"action": {"metadata"}} 
{"data"}
```

bulk api对json的语法有严格的要求，除了delete外，每一个操作都要两个json串（metadata和business data），且每个json串内不能换行，非同一个json串必须换行，否则会报错。

bulk操作中，任意一个操作失败，是不会影响其他的操作的，但是在返回结果里，会告诉你异常日志。

### 增加

```Java
POST /_bulk
{ "create" : { "_index" : "product2", "_id" : "2" } }
{ "field1" : "value1", "field2" : "value2" }
```

在这个请求中，我们创建了一个新的文档，其在 "product2" 索引中的ID为 "2"，并且包含两个字段 "field1" 和 "field2"。

请注意，这个操作都由两行组成：第一行包含操作类型（在这个示例中为 "create"）和元数据；第二行包含要创建或索引的实际文档数据。

### 删除

删除文档，ES对文档的删除是懒删除机制，即标记删除（lazy delete原理）。

```Java
POST /_bulk
{ "delete" : { "_index" : "test-index", "_id" : "1" } }
{ "delete" : { "_index" : "test-index", "_id" : "2" } }
```

在这个请求中，我们从 "test-index" 索引中删除了ID为 "1" 和 "2" 的两个文档。

注意，每个 `delete` 操作仅由一行组成，这一行包含操作类型（在这个示例中为 "delete"）以及元数据。

### 修改

```Java
POST /_bulk
{ "update" : { "_index" : "test-index", "_id" : "1" } }
{ "doc" : { "field1" : "new_value1", "field2" : "new_value2" }}
{ "update" : { "_index" : "test-index", "_id" : "2" } }
{ "doc" : { "field1" : "new_value3", "field2" : "new_value4" }}
```

在这个请求中，我们在 "test-index" 索引中更新了两个文档：

- 我们更新了ID为 "1" 的文档，设置 "field1" 和 "field2" 字段的值为 "new_value1" 和 "new_value2"。
- 我们也更新了ID为 "2" 的文档，设置 "field1" 和 "field2" 字段的值为 "new_value3" 和 "new_value4"。

## filter_path

在Elasticsearch中，`filter_path`参数用于过滤返回的响应内容，可以用于减小 Elasticsearch 返回的数据量。当你指明一个或多个路径时，返回的JSON对象就只会包含这些路径下的键，它接收一个逗号分隔的列表，其中包含了你想要返回的 JSON 对象内的路径。这个参数支持通配符（`*`）匹配和数组元素（`[]`）匹配。列如：

```
POST /_bulk?filter_path=items.*.error
```

上述请求中的 `filter_path=items.*.error` 会让Elasticsearch仅返回 `_bulk` API调用结果中的错误信息。`items.*.error` 这个路径表示，在返回的响应中，匹配到所有存在 `error` 字段的 `items`。

这样做有两个主要好处：

1. 它可以提升Elasticsearch的性能，因为少量的数据意味着更快的序列化和反序列化。
2. 它可帮助你聚焦于感兴趣的部分，不必处理无关的数据。

请注意，`*` 是通配符，代表任何值。

以下是一些其他 `filter_path` 的示例：

1. `filter_path=took`: 这个请求仅返回执行请求所花费的时间（以毫秒为单位）。
2. `filter_path=items._id,items._index`: 这个请求仅返回每个 item 的 `_id` 和 `_index` 字段。
3. `filter_path=items.*.error`: 这个请求会返回所有包含 `error` 字段的 items。
4. `filter_path=hits.hits._source`: 这个请求仅返回搜索结果中的原始文档内容。
5. `filter_path=_shards, hits.total`: 这个请求返回关于 `shards` 的信息和命中的总数。
6. `filter_path=aggregations.*.value`: 这个请求仅返回每个聚合的值。

请注意，如果你在 `filter_path` 中指定了多个字段，你需要使用逗号将它们分隔开。
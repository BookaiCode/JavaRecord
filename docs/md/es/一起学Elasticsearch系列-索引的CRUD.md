本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)

[TOC]

本篇主要是介绍Elasticsearch中索引的基本操作API，即增删改查（CRUD）。

## 创建索引

```JSON
PUT /my_index?pretty
```

`?pretty`是一个可选参数，如果加上，Elasticsearch 将返回格式化（即缩进、换行等使结果更易读）过的 JSON。

输出示例：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "my_index"
}
```

这个输出表示索引已成功创建。`"acknowledged": true` 表示请求已被接受，`"shards_acknowledged": true` 表示所有的分片都已经准备就绪，`"index": "my_index"` 是你刚才创建的索引名称。

## 删除索引

```JSON
DELETE /my_index?pretty
```

假设 `my_index` 索引存在并已成功删除，则输出如下：

```json
{
  "acknowledged" : true
}
```

这个响应表示Elasticsearch已确认删除请求。

**注意：该操作是不可逆的，一旦删除，所有存储在索引中的数据都将被永久移除，因此在执行此操作时务必谨慎**

## 查询数据

请求：

```json
GET /my_index/_search
{
  "query": {
    "match": {
      "field_name": "my_value"
    }
  }
}
```

在此示例中，我们在名为 `my_index` 的索引上进行搜索，查找字段 `field_name` 中值为 `my_value` 的文档。

响应：
Elasticsearch返回的响应包括一系列关于查询的信息，例如查询所花费的时间、是否超时、命中的文档数等。同时，返回的结果也会包括所有匹配的文档。

```json
{
  "took": 30,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "my_index",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "field_name": "my_value"
        }
      }
    ]
  }
}
```

请注意，以上只是基本的示例，实际ES查询可能会复杂得多，包含过滤、聚合、排序等多种操作。

获取所有索引数据的信息

```JSON
GET _cat/indices?v
```

示例输出：

```
health status index                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_task_manager_1   C9SW_Y7cQ8-TJQGArKRcDA   1   0          2            0     31.8kb         31.8kb
yellow open   my_index                 7V75Rtf1QBCslQvWWPOS2A   1   1          0            0       283b           283b
green  open   .apm-agent-configuration en6N1awvRZSLySqh0yjleA   1   0          0            0       283b           283b
green  open   .kibana_1                9-gHntOQTCeM8RqViBAaog   1   0          8            1     19.1kb         19.1kb
```

返回的结果会包含以下列：

- `health`：索引的健康状态。它可以是"green"（一切正常），"yellow"（至少所有主分片都是可用的，但不是所有副本分片都可用）或者"red"（有主分片无法使用）。
- `status`：索引的状态。通常情况下，可能的值是"open"或"close"。
- `index`：索引的名称。
- `uuid`：代表索引的唯一标识符。
- `pri`：主分片的数量。
- `rep`：每个主分片的副本数。
- `docs.count`：存储在索引中的文档数量。
- `docs.deleted`：已删除但尚未完全从存储中移除的文档数量。
- `store.size`：索引当前占用的总物理存储空间。
- `pri.store.size`：主分片占用的物理存储空间。

查询指定文档id

```JSON
GET /my_index/_doc/doc_id
```

返回如下：

```json
{
	"_index": "my_index",
	"_type": "_doc",
	"_id": "1",
	"_version": 3,
	"_seq_no": 2,
	"_primary_term": 2,
	"found": true,
	"_source": {
		"field1": "123",
		"field2": "456"
	}
}
```

这个命令会返回一个包含以下字段的 JSON 响应：

- `_index`：文档所在的索引。
- `_type`：文档的类型。在 7.x 版本中，这通常是 `_doc`。
- `_id`：文档的 ID。
- `_version`： 文档的版本号。每当文档更新时，此数字都会增加。
- `_seq_no`：序列号，每次对文档进行操作时此数字会增加。
- `_primary_term`： 主要期限数，主要用于处理并发控制。
- `found`：如果找到了文档，则此值为 true；否则，为 false。
- `_source`： 文档的原始内容。

如果没有找到与给定 ID 匹配的文档，Elasticsearch 会返回一个状态码为 404 的响应，并且 `found` 字段的值将为 false。

## 添加 & 更新数据

```JSON
PUT /index/_doc/doc_id
{
 JSON数据
}

//例如：PUT /my_index/_doc/1
//{
//  "field1": "123",
//  "field2": "456"
//}
```

PUT既可以用于添加数据，也可以用于更新数据，比如我想更新文档 1 的name字段为：小明，可以这么写：

```JSON
PUT /my_index/_doc/1
{
"name": "小明"
}
```

**注意：PUT既可以用于插入，也可以用于更新，所以PUT的更新是全量更新，而不是部分更新。也就是上面的语句执行之后，文档会被直接替换，只会有name字段，字段值为小明**

如果我们只想部分更新文档中的字段，可以使用POST，示例如下：

```JSON
POST /index/_update/1
{
	"doc": {
		"name": "小明"
	}
}
```

这个命令只会更新文档中的 name 字段为小明。其他字段还是保留原样。

## cat命令

cat命令在ES中会经常使用，下面介绍cat命令中常用的几个命令。

### 参数

cat命令组成形式是：`GET /_cat/indices?format=json&pretty`， `?`之前是命令，之后是参数，多个参数用`&`分隔。

参数有下：

```JSON
//v 显示更加详细的信息
GET /_cat/master?v
//help 显示命令结果字段说明
GET /_cat/master?help
//h 显示命令结果想要展示的字段
GET /_cat/master?h=ip,node
GET /_cat/master?h=i*,node
//format 显示命令结果展示格式,支持格式类型：text json smile yaml cbor
GET /_cat/indices?format=json&pretty
//s 显示命令结果按照指定字段排序
GET _cat/indices?v&s=index:desc,docs.count:desc
```

### 常用命令

**aliases ：显示别名**

```JSON
GET /_cat/aliases
```

获取所有索引别名，如果想获得某个索引的别名可以使用：`GET index/alias`。

**allocation ：显示每个节点的分片数和磁盘使用情况**

```JSON
GET /_cat/allocation
```

**count ：显示整个集群或者索引的文档个数**

```JSON
GET /_cat/count
GET /_cat/count/index
```

**fielddata ：显示每个节点字段所占的堆空间**

```JSON
GET /_cat/fielddata
GET /_cat/fielddata?fields=name,addr
```

**health ：显示集群是否健康**

```JSON
GET /_cat/health
```

**indices ：显示索引的情况**

```JSON
GET /_cat/indices
GET /_cat/indices/index
```

**master： 显示master节点信息**

```JSON
GET /_cat/master
```

**nodes ：显示所有node节点信息**

```JSON
GET /_cat/nodes
```

**recovery ：显示索引恢复情况**

当索引迁移的任何时候都可能会出现恢复情况，例如，快照恢复、复制更改、节点故障或节点启动期间。

```JSON
GET /_cat/recovery
```

**thread_pool ：显示每个节点线程运行情况**

```JSON
GET /_cat/thread_pool
GET /_cat/thread_pool/bulk
GET /_cat/thread_pool/bulk?h=id,name,active,rejected,completed
```

**shards ：显示每个索引各个分片的情况**

展示索引的各个分片，主副分片，文档个数，所属节点，占存储空间大小等信息。

```JSON
GET /_cat/shards
GET /_cat/shards/index
GET _cat/shards?h=index,shard,prirep,state,unassigned.reason
```

分片的状态：`INITIALIZING`初始化；`STARTED`分配完成；`UNASSIGNED`不能分配；可以通过`unassigned.reason`属性查看不能分配的原因。

**segments ：显示每个segment的情况**

包括属于索引，节点，主副，文档数等

```JSON
GET /_cat/segments
GET /_cat/segments/index
```

**templates ：显示每个template的情况**

```JSON
GET /_cat/templates
GET /_cat/templates/mytempla*
```
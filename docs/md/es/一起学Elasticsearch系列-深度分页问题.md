本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)


[TOC]

ES的深度分页问题指的是在大数据集和大页数的情况下，通过持续向后翻页来获取查询结果的一种性能问题。当页码非常高时，ES需要遍历大量文档才能找到正确的分页位置，导致性能和查询速度变慢。

## 深度分页（Deep Paging）

分页是Elasticsearch中最常见的查询场景之一，正常情况下分页代码如下所示：

```json
GET my_index/_search
{
  "from": 0,
  "size": 5
}
```

以下是一个示例响应输出，具体结果会根据实际数据而有所不同：

```json
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 100,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "my_index",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "title" : "Document 1",
          "content" : "This is the content of document 1."
        }
      }
      ......
    ]
  }
}
```

在上述示例中，响应包含了以下信息：

- took：执行搜索所花费的时间（以毫秒为单位）。
- timed_out：指示搜索是否超时。
- _shards：索引的分片信息。
- hits：包含了搜索结果的对象。
  - `total`：匹配到的文档总数。
  - `max_score`：最高得分的文档的分值。
  - `hits`：实际匹配到的文档数组。

但是当我们查询的数据页数特别大， `from + size`大于 `10000`的时候，就会出现问题。

```json
GET my_index/_search
{
  "from": 10000,
  "size": 5
}
```

报错信息如下所示：

> "reason": "Result window is too large, from + size must be less than or equal to: [10000] but was [10005]. See the scroll api for a more efficient way to request large data sets. This limit can be set by changing the [index.max_result_window] index level setting."

报错信息的解释为当前查询的结果超过了 `10000`的最大值，这个错误表示请求中的偏移量（`from`）加上大小（`size`）超过了索引级别参数 `index.max_result_window` 所允许的限制。

默认情况下，该限制为10000，由 `max_result_window` 参数控制。在示例中，请求指定的`from`值为10000加上`size`值为5，总计为10005，超过了默认限制。

## 深度分页的性能问题和危害

首先我们要达成一个共识：

分页查询的时候数据肯定是按照某种顺序排列的，ES中如果不人工指定排序字段，那么最终结果将按照相关度评分排序。

**分布式系统都面临着同一个问题，数据的排序不可能在同一个节点完成**

举个例子：比如我想在一个拥有10万名考生的索引中查询成绩排在10001~10100位的100名考生信息。

这个看似简单的查询实际上并不简单。

假设我们有一个名为"exam_info"的索引，其中存放着10万名考生的考试信息。

由于Elasticsearch的分布式特性和数据分片策略，索引数据在写入时无法预知后续业务查询的具体排序规则，因此数据的排序是随机的。

而且，为了提高数据的准确性，在Elasticsearch中，数据会被均匀地分布在多个分片中。

假设现在有5个分片，并且每个分片中有2万条有效数据。根据需求，我们需要查询成绩排在10001到10100位的一百名考生的信息。为了实现这个目标，首先需要按照成绩进行倒序排列，然后查询按照成绩排序的10001到10100位的学生信息。

在单机数据库中，这个查询逻辑相对简单，只需将10万名学生的成绩排序，然后从前10100条数据中取出第10001~10100条数据，即按照每页100名学生的方式查询第101页的数据。

**然而，在分布式数据库中，情况就不同了，考生的成绩被分散保存在每个分片中，无法保证要查询的这一百名考生的成绩都在同一个分片中**

实际上，结果很可能分布在每个分片中。换句话说，从任意一个分片中取出的前10100名考生的成绩，都不一定是总成绩的前10100名。

**为了解决这个问题，唯一的方法是从每个分片中取出当前分片的前10100名考生的成绩，然后进行汇总（合并排序），再从汇总后的数据中查询前10100名的成绩。只有这样才能确保查询到的成绩是整个索引中的前10100名**

要理解这个过程，可以类比为从保存世界所有国家短跑运动员成绩的索引中查询短跑世界前三名。

每个国家对应一个分片的数据，每个国家会选出成绩最好的前三位运动员参加最后的竞争。然后，从每个国家选出的前三名运动员中再次选出全球前三名。只有经过这两个阶段的筛选和排序，才能得到确切的世界前三名。

现在知道为什么深度分页会导致性能问题了吧。

**每次有序的查询都会在每个分片中执行单独的查询，然后进行数据的二次排序，而这个二次排序的过程是发生在Heap中的，也就是说当你单次查询的数量越大，那么堆内存中汇总的数据也就越多，对内存的压力也就越大**

这里的单次查询的数据量取决于你查询的是第几条数据而不是查询了几条数据，比如你希望查询的是第 `10001~10100`这一百条数据，但是ES必须将前 `10100`条全部取出进行二次查询。

因此，如果查询的数据排序越靠后，就越容易导致OOM（Out Of Memory）情况的发生，频繁的深分页查询会导致频繁的FGC。

ES为了避免用户在不了解其内部原理的情况下而做出错误的操作，设置了一个阈值，即 `max_result_window`，其默认值为 `10000`，通过设定一个合理的阈值，避免初学者分页查询时由于单页数据过大而导致OOM。其作用是为了保护堆内存不被错误操作导致溢出。

 `max_result_window` 的合理大小是需要通过各项指标参数来衡量确定的，比如用户量、数据量、物理内存的大小、分片的数量等等。通过监控数据和分析各项指标从而确定一个最佳值，并非越大越好。

## 深度分页解决方案

### 滚动查询：Scroll Search

Scroll Search是一种用于处理大量数据的分批次查询机制。通过使用滚动搜索，可以在不影响性能的情况下逐批次地获取结果集。

假设我们有一个名为"exam_info"的索引，其中存放着10万名考生的考试信息。我们希望按照成绩进行倒序排序，并获取前100名考生的信息。

示例输入：

```json
GET /exam_info/_search?scroll=5m
{
  "size": 100,
  "sort": [
    { "score": "desc" }
  ]
}
```

参数解释：

- `scroll`：定义滚动搜索的时间间隔。这里设置为5分钟，在5分钟内完成整个滚动搜索操作。
- `size`：每个滚动搜索批次返回的文档数量。这里设置为100，表示每次获取100个考生的信息。
- `sort`：指定按照成绩字段（"score"）进行倒序排序。

示例输出：

```json
{
  "_scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAACsFlRlQjNqSVh0VzIwdXk4UnhOTmdSc2cAAAAAADFLW0xjb3VkT1dHcG9uejZtZURxS3oxMw==",
  "hits": {
    "total": 100000,
    "max_score": null,
    "hits": [
      { "name": "John", "score": 98 },
      { "name": "Alice", "score": 97 },
      { "name": "Bob", "score": 95 },
      ...
    ]
  }
}
```

输出解释：

- `_scroll_id`：滚动搜索的标识符，用于后续获取下一批次结果。
- `hits.total`：符合查询条件的总文档数。这里为10万。
- `hits.hits`：当前批次返回的文档列表，每个文档包含考生的姓名（"name"）和成绩（"score"）。

在获得第一批结果后，可以使用滚动搜索的Scroll API来获取下一批结果，直到获取完整的结果集。

示例输入：

```json
GET /_search/scroll
{
  "scroll": "5m",
  "scroll_id": "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAAACsFlRlQjNqSVh0VzIwdXk4UnhOTmdSc2cAAAAAADFLW0xjb3VkT1dHcG9uejZtZURxS3oxMw=="
}
```

示例输出：

```json
{
  "_scroll_id": "DXF1ZXJ5VGhlbkZldGNoBQAAAAAAAACsFlRlQjNqSVh0VzIwdXk4UnhOTmdSc2cAAAAAADFLW0xjb3VkT1dHcG9uejZtZURxS3oxMw==",
  "hits": {
    "total": 100000,
    "max_score": null,
    "hits": [
      { "name": "Eric", "score": 94 },
      { "name": "Catherine", "score": 93 },
      { "name": "David", "score": 92 },
      ...
    ]
  }
}
```

继续使用Scroll API获取后续批次的结果，直到滚动搜索结束。

相关参数的含义：

- `scroll`：定义滚动搜索的时间间隔。指定一个合适的时间段，确保在这个时间内能够完成整个滚动搜索操作。默认为1分钟，时间单位应越小越好，够当前查询使用即可。

时间单位：

| `d`      | Days         |
| -------- | ------------ |
| `h`      | Hours        |
| `m`      | Minutes      |
| `s`      | Seconds      |
| `ms`     | Milliseconds |
| `micros` | Microseconds |
| `nanos`  | Nanoseconds  |

- `size`：每个滚动搜索批次返回的文档数量。



Scroll Search 无法保存索引状态，原因是滚动搜索是一种临时的、游标式的查询机制，仅用于获取大量数据的分批次结果。它并不会保留索引状态或缓存查询结果。

当执行滚动搜索时，Elasticsearch会创建一个滚动上下文（scroll context），该上下文存储了关于初始查询的一些信息，包括查询条件、排序方式等。然后，每次使用滚动上下文来获取下一批结果时，Elasticsearch都会根据该上下文重新执行查询以返回新的结果。这样可以确保在整个滚动搜索过程中，能够按顺序逐步获取完整的结果集。

然而，滚动搜索并不会保存查询结果或索引的快照。一旦滚动上下文被使用完毕（超过滚动时间间隔或已经遍历完所有结果），它就会被丢弃，并且之前返回的结果将不能再重现。如果需要持久化查询结果或经常使用相同的滚动上下文进行查询，可能需要考虑其他方法，如将结果存储在自定义的数据结构中或使用游标分页等技术。

注意：

- Scroll上下文的存活时间是滚动的，下次执行查询会刷新，也就是说，不需要足够长来处理所有数据，它只需要足够长来处理前一批结果。保持旧段处于活动状态意味着需要更多的磁盘空间和文件句柄。确保您已将节点配置为具有充足的空闲文件句柄。
- 为防止因打开过多Scrolls而导致的问题，ES不允许用户打开超过一定限制的Scrolls。默认情况下，打开Scrolls的最大数量为 500。此限制可以通过 `search.max_open_scroll_context`集群设置进行更新 。

Scroll 超时后，搜索上下文会自动删除。然而，保持Scrolls打开是有代价的，因此一旦不再使用就应明确清除Scroll上下文。

```json
#清除单个
DELETE /_search/scroll
{
  "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAAD4WYm9laVYtZndUQlNsdDcwakFMNjU1QQ=="
}

#清除多个
DELETE /_search/scroll
{
  "scroll_id" : [
    "scroll_id1",
    "scroll_id2"
  ]
}

#清除所有
DELETE /_search/scroll/_all
```

总而言之，滚动搜索是一种方便的分批次查询机制，但不适合长期保存查询结果或索引状态。它主要用于处理大量数据的查询，以提高性能和效率。

### Search After

Search After 是一种基于游标的分页查询机制，用于获取大量数据的连续结果。与滚动搜索不同，Search After适用于持久化保存查询状态，并支持随时获取下一页结果。

假设我们有一个名为"exam_info"的索引，其中存放着10万名考生的考试信息。我们希望按照成绩进行倒序排序，并获取前100名考生的信息。

示例输入：

```json
GET /exam_info/_search
{
  "size": 100,
  "sort": [
    { "score": "desc" }
  ]
}
```

参数解释：

- `size`：每页返回的文档数量。这里设置为100，表示每次获取100个考生的信息。
- `sort`：指定按照成绩字段（"score"）进行倒序排序。

示例输出：

```json
{
  "took": 5,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 100000,
    "max_score": null,
    "hits": [
      { "name": "John", "score": 98 },
      { "name": "Alice", "score": 97 },
      { "name": "Bob", "score": 95 },
      ...
    ]
  }
}
```

输出解释：

- `took`：查询所花费的时间，单位为毫秒。
- `hits.total`：符合查询条件的总文档数。这里为10万。
- `hits.hits`：当前页返回的文档列表，每个文档包含考生的姓名（"name"）和成绩（"score"）。

在获得第一页结果后，可以使用Search After来获取下一页的结果。

示例输入：

```json
GET /exam_info/_search
{
  "size": 100,
  "sort": [
    { "score": "desc" }
  ],
  "search_after": [97]
}
```

参数解释：

- `size`：每页返回的文档数量。与初始请求保持一致。
- `sort`：指定按照成绩字段进行倒序排序。与初始请求保持一致。
- `search_after`：指定上一页最后一条数据的排序值，以此作为游标进行下一页查询。

示例输出：

```json
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 100000,
    "max_score": null,
    "hits": [
      { "name": "Eva", "score": 94 },
      { "name": "Daniel", "score": 93 },
      { "name": "Catherine", "score": 92 },
      ...
    ]
  }
}
```

Search After 和 Scroll Search 的主要区别如下：

- 结果排序：Search After依赖排序字段进行分页，需要指定相应的排序方式。而Scroll Search可以根据查询条件对结果进行排序。
- 时间限制：Search After没有时间限制，可按需获取结果。而Scroll Search需要设置滚动时间间隔，超过该时间将失去滚动上下文。

总结起来，ES的深度分页在处理大规模数据集时是一项非常有用的功能，深度分页查询可能会面临一些性能和可靠性方面的挑战，需要根据具体情况进行权衡和优化。
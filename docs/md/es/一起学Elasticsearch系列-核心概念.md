本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)

[TOC]

**开个新的坑，创作关于Elasticsearch的系列文章**

首先，让我们简单的了解一下Elasticsearch：

Elasticsearch是一个开源的搜索和分析引擎，支持近实时的大数据存储、搜索和分析。它基于Apache Lucene项目，提供全文搜索及能力强大的分布式多用户搜索引擎，同时配备RESTful web接口。它不仅能执行复杂查询，还能高效处理复杂的数据分析。

由于其出色的大数据处理能力，Elasticsearch被广泛应用于需要快速搜索和分析大量结构化和非结构化数据的业务场景。

众多公司都广泛使用Elasticsearch，而我们学习它，是因为它是当前最流行的搜索和数据分析解决方案之一。

这是该系列的首篇文章，本章将介绍Elasticsearch的基本概念和相关术语，以帮助您初步理解并熟悉Elasticsearch。

注：所用的Elasticsearch版本为7.4。

## 节点

ElasticSearch 是一种分布式搜索和分析引擎，它的核心是 Lucene。 ES 具有分布式特性，意味着它可以在很多不同的服务器上运行，这些服务器被称为 "节点"。

- 每个Elasticsearch节点实际上就是一个Java进程，就是一个Elasticsearch的实例。
- 一个节点 ≠一台服务器，意味着一台服务器上可以启动多个Elasticsearch的实例。

集群节点角色可以在配置文件`elasticsearch.yml`中通过`node.roles`配置，如果配置了节点角色，那么该节点将只会执行配置的角色功能。

根据角色的不同，Elasticsearch 的节点可以分为以下几种类型：

### master：候选节点

所谓master节点，就是在主节点down机的时候，可以参与选举，取而代之的节点。

举个例子：

**主节点好比班长，在班长不在的时候（主节点down机了），要选举出一个临时班长（master中选举）**

master节点不仅有选举权还有被选举权，每个master节点主要负责索引创建、索引删除、追踪节点信息和决定分片分配节点等。

配置方法：

```yaml
node.roles: [ master ]
```

### data：数据节点

**数据节点顾名思义就是存放数据的节点，数据节点负责存储文档数据和数据的CRUD操作**

因此该节点是CPU和IO密集型，需要实时监控该节点的资源信息，以免过载。

数据节点又可分为：data_content，data_hot，data_warm，data_code。

- **data_content**：数据内容节点，目录节点负责存储常量数据，且不随着时间的推移，改变数据的温层（hot、warm、cold）。且该节点的查询优先级是高于其它IO操作，所以该节点search和aggregations都会较快一些。
- **data_hot**：热节点，保存热数据，经常会被访问，用于存储最近频繁搜索和修改的时序数据。
- **data_code**：冷节点，保存冷数据，很少会被访问，当数据不再更新，那么可以将该数据移动到冷数据节点，冷数据节点用于存储只读，且访问频率较低的数据。该节点机器性能可以低一点。
- **data_warm**：温节点，介于热节点和冷节点之间，当数据访问频率下降，可以将其移动到温节点，温节点用于存储修改较少，但仍然有查询的数据，查询的频率肯定比热点节点要少。

配置方法：

```yaml
node.roles: [ data ]
```

### Ingest：预处理节点

Ingest 节点是 Elasticsearch 的一种特殊类型的节点，用于预处理文档。预处理可能包括执行各种转换和修改，例如增加新字段、改变已有字段的值、移除字段、更改字段的数据类型等。

在 Elasticsearch 中，此类预处理操作是由 Ingest Pipeline 来完成的。一个 Ingest Pipeline 是一系列的处理器（processor），每一个处理器完成特定的任务。你可以定义多个处理器，然后按照特定的顺序执行。

要配置 Elasticsearch 使其具有 Ingest 能力，需要在 `elasticsearch.yml` 文件中设置如下：

```yaml
node.roles: [ ingest ]
```

以上设置表示该节点将作为 Ingest 节点。

以下是创建 Ingest Pipeline 的简单示例:

```json
PUT _ingest/pipeline/my_pipeline_id
{
  "description" : "my pipeline",
  "processors" : [
    {
      "set" : {
        "field": "new_field",
        "value": "new_value"
      }
    },
    {
      "remove" : {
        "field": "old_field"
      }
    }
  ]
}
```

这个 pipeline 包含两个处理器，第一个处理器在文档中添加了一个新字段，并设置了它的值；第二个处理器删除了一个旧字段。

之后在索引文档时，可以使用这个 pipeline：

```json
PUT /my_index/_doc/my_id?pipeline=my_pipeline_id
{
  "field1" : "value1",
  "old_field" : "value2"
}
```

在这个例子中，被索引的文档将被 Ingest pipeline 提前处理，添加新字段并删除旧字段。

### ml：机器学习节点

Elasticsearch的机器学习（ML）节点用于运行各种机器学习作业，例如异常检测或数据帧分析。这些节点特性在 Elasticsearch 的集群设置中进行配置。

配置 elasticsearch.yml：打开每个节点的 'elasticsearch.yml' 文件，并添加或修改以下设置：

```yaml
node.roles: [ ml ]
xpack.ml.enabled: true
```

这些设置会使节点成为一个机器学习节点。

注意：在生产环境中，你应确保你的机器学习节点有足够的内存和 CPU 来处理你的机器学习工作负载。如果你尝试运行一个过于复杂或者数据量过大的机器学习作业，可能会导致节点崩溃或者过载。

### remote_ cluster_ client：远程候选节点

远程候选节点可以作为远程集群的客户端，其作用在于帮助在本地集群与远程集群之间进行通信。当你希望模拟跨集群搜索或者跨集群复制时，这个节点角色就会派上用场。

配置：

```
node.roles: [ remote_cluster_client ]
```

然后，你需要在`elasticsearch.yml`中设置远程集群的信息：

```yaml
cluster:
  remote:
    my_remote_cluster:
      seeds: 127.0.0.1:9300
```

在此示例中，“my_remote_cluster”是远程集群的别名，而“seeds”是远程集群中的种子节点的地址列表。你可以根据实际情况来更改这些值。

注意：在某些环境中，可能需要额外的网络配置才能确保节点之间的正常通信。

### transform：转换节点

转换节点（Transform）是一种将 Elasticsearch 索引数据进行统计分析并产生新的索引的功能。它可以用来执行复杂的聚合查询，并将结果持久化到新的 Elasticsearch 索引中。这个过程可以定期运行，也可以根据需求随时启动或停止。

以下是创建一个 Transform 的基本步骤：

1. 使用 Kibana 或者 Elasticsearch API 来创建一个 Transform。你需要指定源索引（source index），目标索引（destination index），以及 Transform 的配置。
2. 在 Transform 的配置中，你需要指定聚合查询（aggregations）以及群组字段（group by fields）。这些配置决定了怎样对源索引进行统计分析并生成新的索引。

一个简单的 Transform 配置示例如下：

```json
PUT _transform/my_transform
{
  "source": {
    "index": ["my_source_index"]
  },
  "dest": {
    "index": "my_dest_index"
  },
  "pivot": {
    "group_by": {
      "user_id": {
        "terms": {
          "field": "user_id"
        }
      }
    },
    "aggregations": {
      "total_clicks": {
        "sum": {
          "field": "clicks"
        }
      }
    }
  },
  "frequency": "1m",
  "sync": {
    "time": {
      "field": "timestamp",
      "delay": "60s"
    }
  }
}
```

在上面的示例中，Transform 是从 `my_source_index` 中的数据计算每个 `user_id` 的点击次数总和，并将结果保存到 `my_dest_index` 中。这个 Transform 每分钟运行一次，并且在处理含有 `timestamp` 字段的最新数据时会有 60 秒的延迟。

更多关于 Elasticsearch Transform 的详细信息，建议参考 Elasticsearch 官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/7.4/transform-apis.html

### voting_ only：仅投票节点

在master选举过程中，仅投票节点顾名思义就是只参与主节点选举过程，但不会被选为主节点。

**"voting-only"节点主要用来解决"split brain"（脑裂）的问题**

在某些情况下，可能会发生两个或更多节点都认为自己是主节点的情况，这就是所谓的脑裂。通过增加"voting-only"节点，可以增加主节点选举的“选票”，从而降低脑裂的风险。

要将一个节点配置为"voting-only"节点，需要在该节点的`elasticsearch.yml`配置文件中设置以下属性：

```yaml
node.roles: [voting_only]
```

保存并重启该节点后，它就会成为一个"voting-only"节点。

### coordinating only：协调节点

协调节点主要负责根据集群状态路由分发搜索，不参与索引和搜索操作，不存储数据，只负责将请求路由到适当的节点（例如数据节点或主节点），并根据结果组织响应返回给客户端。

**此外每个节点都自带协调节点功能，即便没有去专门配置，任何Elasticsearch节点默认都能成为协调节点**

## 分片

分片的思想在很多分布式应用和海量数据处理的场景中都非常常见，通常来说，面对海量数据的存储，单个节点显得力不从心。

**通俗解释，分片就是将数据拆分多份，放到不同的服务器节点**

Elasticsearch里的分片分为两种：

- **主分片（Primary Shard）**：这是最初创建索引时所设定的分片。主要用于数据持久化，可以通过配置文件或者API进行设置。
- **副本分片（Replica Shard）**：这是从主分片复制出来的分片，用于提高数据的可用性和查询性能。副本不会与其对应的主分片放在同一节点上，以防止单点故障。

### 主分片

ES可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。构成分布式搜索。

**分片的数量只能在索引创建前指定，并且索引创建后不能更改**

ES的分片数量在索引创建时设定是因为ES将每个索引的数据分布在多个分片上以实现数据的水平扩展。这种分布是基于数据的哈希值进行的，这样可以保证数据的均匀分布。一旦索引被创建并且数据开始写入，这种数据的分布就已经确定下来了。

当客户端发起创建document的时候，ES需要确定这个document放在该index的哪个shard上。这个过程就是**数据路由**。

**路由算法：shard = hash(routing) % number_of_primary_shards**

这里的`routing`指的就是document的id，如果`number_of_primary_shards`在查询的时候取余发生了变化，则无法获取到该数据。

并且如果在索引创建后改变分片的数量，就需要重新计算所有数据的哈希值并且在分片之间迁移数据。这不仅会消耗大量的计算和IO资源，而且在数据迁移过程中还可能会影响查询的准确性。因此，ES设计决定在索引创建时就确定分片的数量，而且创建后不能更改。

然而，虽然原始分片的数量在创建后不能更改，但是你可以通过**reindex**操作将数据复制到一个新的索引中，这个新的索引可以有不同的分片数量。

### 副本分片

副本分片代表索引的副本，ES可以设置多个索引的副本，副本的作用一是提高系统的容错性，当某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高ES的查询效率，ES会自动对搜索请求进行负载均衡。

- 每个主分片和其副本分片不能存在于同一个节点上，所以最低的可用配置是两个节点互为主备。
- 副本分片是不能直接写入数据的，只能通过主分片做数据同步。

以下是如何在创建索引时配置主分片和副本分片的示例：

```json
PUT /my_index
{
    "settings": {
        "number_of_shards": 3,  # 主分片数量
        "number_of_replicas": 2 # 副本分片数量
    }
}
```

这个设置会创建一个名为`my_index`的新索引，它有3个主分片和2个副本。也就是说，总共有9个分片 (3主 * (1原始 + 2副本))。

你也可以在索引创建后修改其副本分片数：

```json
PUT /my_index/_settings
{
    "number_of_replicas": 1
}
```

这将`my_index`索引的副本数从2更改为1。

请注意，虽然你可以在索引创建后更改副本分片的数量，但不能更改主分片的数量。因此，在创建索引时，需要仔细考虑主分片的数量。

## 集群状态

集群健康状态（Cluster Health）描述了集群的总体健康状况，分为 "Green"、"Yellow" 和 "Red"。

- Green：主/副分片都已经分配好且可用，集群处于最健康的状态100%可用。
- Yellow：主分片可用，但是至少有一个副本是未分配的。这种情况下数据也是完整的，但是集群的高可用性会被弱化。
- Red：至少有一个不可用的主分片。此时只是部分数据可以查询，已经影响到了整体的读写，需要重点关注。

## 健康值检查

在Elasticsearch中，你可以使用两种主要的API来检查集群的健康状况：`_cat/health`和`_cluster/health`。虽然它们提供了相似的信息，但是它们的输出格式不同。

`_cat/health`：该API以简洁的表格式返回关于集群健康的信息。它易于阅读和理解。

示例：`GET _cat/health?v`

这将返回如下所示的输出：

```
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1605102382 13:59:42  elasticsearch green           3         3     12  6    0    0        0             0                  -                100.0%
```

返回参数说明：

| 参数                  | 含义                       |
| --------------------- | -------------------------- |
| epoch                 | 自Unix Epoch以来的秒数     |
| timestamp             | 当前时间戳                 |
| cluster               | 集群名称                   |
| status                | 集群状态(绿色、黄色或红色) |
| node.total            | 集群中的节点总数           |
| node.data             | 集群中承载数据的节点数     |
| shards                | 集群中的分片总数           |
| pri                   | 集群中的主要分片数量       |
| relo                  | 正在进行重定位的分片数量   |
| init                  | 初始化的分片数量           |
| unassign              | 未分配的分片数量           |
| pending_tasks         | 等待执行的集群级任务数量   |
| max_task_wait_time    | 等待任务的最长时间         |
| active_shards_percent | 活动分片占总分片的百分比   |

- `_cluster/health`：这个API 以JSON格式返回关于集群健康的详细信息。它提供更丰富的数据，并因此更适合编程访问。

示例：`GET _cluster/health`

这将返回如下所示的输出：

```json
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 12,
  "active_shards" : 12,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 85.7
}
```

返回参数说明：

| 参数                               | 描述                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| `cluster_name`                     | 集群的名称。                                                 |
| `status`                           | 集群的状态，它可能的值有：`green`、`yellow` 或者 `red`。`green` 表示所有主要和副本分片都是活动的。`yellow` 表示所有主要分片是活动的但不是所有副本都是活动的。`red` 表示至少一个主要分片不是活动的。 |
| `timed_out`                        | 如果请求超时，该值为 true。                                  |
| `number_of_nodes`                  | 集群中的节点数。                                             |
| `number_of_data_nodes`             | 集群中执行数据相关操作的节点数。                             |
| `active_primary_shards`            | 当前活动的主分片数。                                         |
| `active_shards`                    | 当前活动的分片数。                                           |
| `relocating_shards`                | 正在重新定位的分片数。                                       |
| `initializing_shards`              | 初始化中的分片数。                                           |
| `unassigned_shards`                | 未分配的分片数。                                             |
| `delayed_unassigned_shards`        | 延迟未分配的分片数。                                         |
| `number_of_pending_tasks`          | 等待执行的集群级别更改的数量。                               |
| `number_of_in_flight_fetch`        | 当前正在进行的获取操作数。                                   |
| `task_max_waiting_in_queue_millis` | 在执行队列中等待的任务的最长时间（以毫秒为单位）。           |
| `active_shards_percent_as_number`  | 活动分片所占的百分比。                                       |

## 索引和文档

ES中索引可以类比为关系型数据库中的Table，在7.0版本之前index由若干个type组成，type实际上是文档的逻辑分类，而文档是ES存储的最小单元。文档（doc）可以类比为关系型数据库中的行，每个文档都有一个文档id。

**7.0及之后弱化了type的概念，7.x版本index只有一个type：_doc**


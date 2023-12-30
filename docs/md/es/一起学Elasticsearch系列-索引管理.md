本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)


[TOC]

在Elasticsearch中，索引是对数据进行组织和存储的基本单元。索引管理涉及创建、配置、更新和删除索引，以及与索引相关的操作，如数据导入、搜索和聚合等。这些关键任务直接影响着系统性能、数据可用性和查询效率。

本文将深入探讨ES索引管理的重要性和最佳实践。我们将介绍索引模板的概念及其用途，了解如何通过索引别名实现无缝切换和版本控制。我们还将探讨滚动索引的概念，它可以帮助应对长期运行的查询和保持数据的时效性。

## 常用索引API

### _cat

通过使用 `_cat API`，可以快速查看和监控集群的状态、索引的健康情况、节点间分片的分配情况等。这些接口提供了轻量级和易于使用的方式来获取集群信息，为管理员和开发人员提供了便利。

| API                           | 描述                                 |
| ----------------------------- | ------------------------------------ |
| `_cat/allocation`             | 获取分片分配信息                     |
| `_cat/count`                  | 获取索引文档计数信息                 |
| `_cat/fielddata`              | 获取字段数据缓存信息                 |
| `_cat/health`                 | 获取集群健康状态信息                 |
| `_cat/indices`                | 获取索引信息                         |
| `_cat/master`                 | 获取主节点信息                       |
| `_cat/nodeattrs`              | 获取节点属性信息                     |
| `_cat/nodes`                  | 获取节点信息                         |
| `_cat/pending_tasks`          | 获取挂起任务信息                     |
| `_cat/plugins`                | 获取插件信息                         |
| `_cat/recovery`               | 获取分片恢复信息                     |
| `_cat/repositories`           | 获取仓库信息                         |
| `_cat/thread_pool`            | 获取线程池信息                       |
| `_cat/shards`                 | 获取分片信息                         |
| `_cat/snapshots`              | 获取快照信息                         |
| `_cat/tasks`                  | 获取任务信息                         |
| `_cat/templates`              | 获取索引模板信息                     |
| `_cat/segments`               | 获取段信息                           |
| `_cat/aliases/{alias}`        | 根据别名获取指定索引的别名信息       |
| `_cat/indices/{index}`        | 根据索引名称获取指定索引的详细信息   |
| `_cat/shards/{index}`         | 根据索引名称获取指定索引的分片信息   |
| `_cat/recovery/{index}`       | 根据索引名称获取指定索引的恢复信息   |
| `_cat/segments/{index}`       | 根据索引名称获取指定索引的段信息     |
| `_cat/tasks/{task_id}`        | 根据任务ID获取指定任务的详细信息     |
| `_cat/snapshots/{repository}` | 根据仓库名称获取指定仓库中的快照信息 |
| `_cat/nodeattrs/{node_id}`    | 根据节点ID获取指定节点的属性信息     |

### _cluster

通过使用 `_cluster API`，可以获得集群的健康状况、资源使用情况，进行集群级别的配置和管理操作。这些接口提供了对集群的细粒度控制和监控能力，帮助您保持集群的稳定性、优化性能，并进行故障排除和调试。

请注意，访问 `_cluster API` 需要具有适当的权限。使用这些接口时，请确保遵循 Elasticsearch 的安全最佳实践，并谨慎处理敏感信息。

| API                                     | 描述                       |
| --------------------------------------- | -------------------------- |
| `_cluster/allocation/explain`           | 解释分片分配相关信息       |
| `_cluster/health`                       | 获取集群健康状态信息       |
| `_cluster/pending_tasks`                | 获取挂起任务信息           |
| `_cluster/reroute`                      | 重新路由分片               |
| `_cluster/state`                        | 获取集群状态信息           |
| `_cluster/stats`                        | 获取集群统计信息           |
| `_cluster/settings`                     | 获取或更改集群级别的设置   |
| `_cluster/recovery`                     | 获取正在进行的分片恢复信息 |
| `_cluster/nodes/hot_threads`            | 获取热线程信息             |
| `_cluster/nodes/info`                   | 获取节点信息               |
| `_cluster/nodes/reload_secure_settings` | 重新加载安全设置           |
| `_cluster/nodes/stats`                  | 获取节点统计信息           |
| `_cluster/pending_tasks`                | 获取待处理的任务列表       |
| `_cluster/remote/info`                  | 获取远程集群连接信息       |

### 判断索引是否存在

```JSON
HEAD <index>
```

### 打开和关闭索引

在生产环境有时要禁止索引做读写操作，此时可以对索引执行关闭。

**打开索引**

```JSON
POST <index>/_open
```

**关闭索引**

```JSON
POST <index>/_close
```

## 索引压缩

索引压缩并不是指压缩索引的大小，而是压缩索引的分片。

例如：有一个名为 `my_index`的索引有6个主分片，压缩为只有2个主分片，称之为索引压缩。

### 前提条件

- 进行压缩的时候，索引必须是只读状态。
- 目标索引的所有主分片必须位于同一节点（因为创建目标索引时会将段从源索引硬链接到目标索引，而硬链接是不支持跨节点的。如果文件系统不支持硬链接，则会将所有segment file都复制到新索引中，复制过程很耗时）。
- 索引的健康状态必须为Green。
- 目标索引不能已存在，避免重名。
- 目标索引的分片数量必须为源索引的约数，比如我源索引分片数为6，那么目标索引的分片数只能为：1，2，3，6。
- 目标节点所在的服务器确保有足够大的磁盘空间。

### 操作步骤

1. **备份数据，以防数据丢失，不做强制要求**

```json
POST _reindex 
{
    "source": {
        "index": "source_index"
    },
    "dest": {
        "index": "target_index"
    }
}
```

注意：索引比较大的情况下，`_reindex` 操作可能耗时会比较久。

2. **设置副本数为0**

```json
"index.number_of_replicas": 0
```

关闭副本数主要是为了只做一份数据的压缩，重复数据没必要同步两次。

3. **设置只读**

设置索引为只读状态，在索引压缩的时候，数据是不可写的。

```json
"index.blocks.write": true
```

4. **迁移数据**

```json
"index.routing.allocation.require._name": "target_node"
```

`index.routing.allocation.require`是Elasticsearch中的索引级别设置，用于指定分配索引分片的要求条件。通过设置该参数，可以控制分配策略，将索引的分片分配到特定的节点或节点标签。

具体使用方式如下：

设置分片的要求条件：

```json
PUT /my_index/_settings
{
  "index.routing.allocation.require.node_type": "hot"
}
```

上述示例将索引`my_index`的分片要求分配到拥有`node_type=hot`标签的节点上。

5. **执行压缩命令**

```json
POST /my_index/_shrink/target_index
{
  "settings": {
    "index.number_of_replicas": 1,
    "index.number_of_shards": 3, 
    //索引压缩算法
    "index.codec": "best_compression" 
  }
}
```

`reindex`和`shrink`是Elasticsearch中用于重新索引和缩减索引的两个不同操作，它们具有以下区别：

Reindex（重新索引）：

- `reindex`操作用于将数据从一个索引复制到另一个索引，并可以在此过程中进行转换、筛选或重塑数据。
- 通过`reindex`操作，可以更改索引的映射、调整分片设置、修改文档内容等。
- `reindex`操作是非破坏性的，原始索引和目标索引同时存在，可以逐步迁移数据。

Shrink（缩减索引）：

- `shrink`操作用于减少索引的分片数量，将一个大的索引缩减为更小的索引。
- 通过`shrink`操作，可以将原始索引的分片合并为较少数量的目标索引分片。
- `shrink`操作通常用于优化索引性能、减少资源占用、提高查询效率等。注意，缩减索引会导致一定的数据迁移开销。

关键区别：

- `reindex`是复制数据并对其进行转换的过程，可以在任何时候执行，并且不需要目标索引事先存在。它允许更灵活地处理数据，并且可以应用各种转换逻辑。
- `shrink`是减少索引分片数量的操作，只能在满足特定条件的情况下执行。它主要用于优化索引性能和资源利用。

总结来说，`reindex`用于数据的复制和转换，而`shrink`用于缩减索引的分片数量以提高性能。具体选择哪个操作取决于需求和目标。

6. **恢复索引**

```json
PUT target_index/_settings
{
  //允许分配到任意节点
  "index.routing.allocation.require._name": null,
  "index.blocks.write": false
}
```

索引压缩完成，此时可以逐步将业务从源索引切流到目标索引上，可以使用alias来完成切流过程。

## 索引别名

索引别名是个非常重要并且非常实用的功能

### 别名作用

**官方描述**

索引别名是用于引用一个或多个现有索引的辅助名称，大多数 Elasticsearch API 接受索引别名来代替索引。

Elasticsearch 的所有 API 都会自动将别名转换为实际的索引名称。一个别名也可以映射到多个索引，当指定它时，别名会自动扩展为别名索引。别名也可以与搜索时自动应用的过滤器和路由值相关联。别名不能与索引同名。

**保护索引**

索引相对于调用者是隐藏的。

### 使用场景

索引别名在Elasticsearch中具有以下用途和使用场景：

- 通过使用别名，可以为不同版本的索引创建不同的别名，并在切换版本时轻松进行索引升级和回滚。
- 可以先创建一个新的索引并将别名指向新索引，然后平滑地将读写操作从旧索引切换到新索引。
- 通过别名，可以将多个索引组合成一个逻辑集合，并对集合进行查询或操作。这样做可以方便地处理大量数据，实现数据分片和分布式搜索。
- 使用别名可以为不同的用户、应用程序或租户创建独立的别名，以实现数据的隔离和多租户支持。
- 别名还可以用于按照特定规则将请求路由到不同的索引，以实现负载均衡或按时间范围进行数据分片。

这些只是一些使用场景的例子，实际上别名功能非常灵活，可以根据具体需求进行定制。索引别名为用户提供了更大的灵活性和可管理性，使得在进行索引维护、升级或数据操作时更加方便和安全。

### 使用

**语法**

```json
POST /_aliases
```

下面是一个使用`_aliases` API 创建和删除别名的示例，以及相应的输入和输出：

**创建别名**

- 输入：

  ```json
  POST /_aliases
  {
    "actions": [
      { "add": { "index": "my_index", "alias": "my_alias" } }
    ]
  }
  ```

- 输出：

  ```json
  {
    "acknowledged": true
  }
  ```

  解释：通过以上输入，将索引`my_index`与别名`my_alias`进行关联。输出中的`acknowledged`字段为`true`表示操作已成功。

**删除别名**

- 输入：

  ```json
  POST /_aliases
  {
    "actions": [
      { "remove": { "index": "my_index", "alias": "my_alias" } }
    ]
  }
  ```

- 输出：

  ```json
  {
    "acknowledged": true
  }
  ```

  解释：通过以上输入，解除索引`my_index`与别名`my_alias`的关联。输出中的`acknowledged`字段为`true`表示操作已成功。


注意：

-  一个索引可以绑定多个别名，一个别名也可以绑定多个索引。
-  别名不能和索引名相同。

## 索引模版

索引模板（Index Template）是在Elasticsearch中用于自动创建和配置索引的一种机制。它允许你定义索引的设置、映射和别名等，并在新索引满足特定条件时自动应用这些配置。

索引模板的主要目的是为了简化索引管理和维护工作，并确保新创建的索引具有一致的结构和配置。

索引模板在企业生产实践中常配合滚动索引（Rollover Index）、索引的生命周期管理（ILM：Index lifecycle management）、数据流一起使用。

以下是索引模板的核心概念和用法：

- 索引名称模式（index_patterns）：
  - 索引模板通过`index_patterns`参数定义一个或多个与索引名称匹配的模式。
  - 模式通常包含一个固定的前缀和一个通配符，如`my_index-*`，其中`*`表示通配符部分。
  - 当新索引的名称与模式匹配时，该模板将被应用到新索引上。
- 设置和配置（settings）：
  - 索引模板可以定义新索引的各种设置和配置，例如分片数量、副本数量、刷新间隔等。
  - 这些设置将自动应用到新索引上，确保新索引的行为与模板定义的一致。
- 映射（mappings）：
  - 通过索引模板，您可以定义新索引的字段映射、属性和数据类型等。
  - 索引模板中的映射配置将自动应用到新索引上，确保新索引的结构与模板定义的一致。
- 别名（aliases）：
  - 索引模板可以定义一个或多个别名，用于管理和访问新创建的索引。
  - 别名可以被用作查询、索引操作等，而不需要直接指定特定的索引名称。

通过使用索引模板，可以根据业务需求和索引管理的最佳实践来定义一套统一的索引创建和配置规则。当创建新的满足模式匹配条件的索引时，Elasticsearch会自动应用该模板，并创建带有预定义设置、映射和别名的索引。

下面是一个示例请求，用于创建名为`my_index_template`的索引模板：

```json
PUT _index_template/my_index_template
{
  "index_patterns": ["my_index-*"],
  "template": {
    "settings": {
      "number_of_shards": 5,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "field1": {
          "type": "text"
        },
        "field2": {
          "type": "keyword"
        }
      }
    },
    "aliases": {
      "my_alias": {}
    }
  }
}
```

在上述示例中：

- `_index_template/my_index_template`表示要创建的索引模板的名称。
- `index_patterns`参数设置了匹配的索引名称模式，这里使用了通配符`*`来适配任意后缀。
- `template`字段指定了要应用于新索引的设置、映射和别名等配置。
- `settings`定义了新索引的分片数为5，副本数为1。
- `mappings`指定了新索引的字段映射配置，其中`field1`为文本类型，`field2`为关键字类型。
- `aliases`定义了一个别名`my_alias`，用于访问该索引。

通过发送以上请求，您可以创建一个名为`my_index_template`的索引模板。当新创建的索引名称满足`my_index-*`的模式时，该模板将自动应用到新索引上，并使新索引具有预定义的设置、映射和别名。

## 滚动索引

滚动索引（Rollover Index）是Elasticsearch中用于管理索引自动切换和维护的一种机制。它允许在索引达到预定义条件时，自动创建新的索引并将写入流量切换到新索引上，以实现索引的平滑滚动和分片维护。

使用滚动索引的主要场景是处理大容量和高吞吐量的数据，并确保索引的性能和可伸缩性。

下面是滚动索引（Rollover Index）的基本工作流程：

1. 创建索引模板（Index Template）：
   - 首先，你需要创建一个索引模板，其中包含指定条件以触发滚动操作的设置。这些条件可以是基于时间、文档数量、索引大小等来定义。
2. 创建初始索引（Initial Index）：
   - 使用索引模板创建初始索引，该索引将用于接收初始的数据写入流量。
3. 监控索引状态：
   - 持续监控当前索引的状态和指标，如文档数量、索引大小等。
   - 当索引达到预定义的条件时，即满足了滚动的触发条件，就会开始进行滚动操作。
4. 触发滚动操作：
   - 当索引满足触发条件时，Elasticsearch将自动创建新的索引，同时将写入流量切换到新索引上。
   - 新索引可以具有更高的分片数、新的配置参数和优化设置，以确保整个系统的性能和可用性。
5. 后续维护和操作：
   - 一旦滚动操作完成，你可以对旧索引进行必要的维护操作，如关闭、删除或备份。
   - 您还可以使用滚动名称（Alias）来管理所有滚动的索引，以便于查询和操作。

通过使用滚动索引（Rollover Index），可以实现自动化的索引管理和平滑的索引切换，同时为数据处理和存储提供了更好的可伸缩性和性能。它特别适用于需要处理大量数据并保证系统稳定性的场景，如日志记录、时间序列数据等。

### 触发条件

滚动索引（Rollover Index）的触发条件是通过索引模板中的一些参数来定义的。以下是常用的触发条件及其参数：

- 文档数量触发条件：
  - `max_docs`：指定索引中的文档数量上限，达到该值时触发滚动操作。
- 索引大小触发条件：
  - `max_size`：指定索引的大小上限，达到该值时触发滚动操作。
  - `max_size_bytes`：与`max_size`类似，但使用字节数表示索引大小上限。
- 时间触发条件：
  - `max_age`：指定索引的最大存储时间，超过该时间时触发滚动操作。
  - `max_age_seconds`：与`max_age`类似，但以秒为单位表示存储时间上限。
- 自定义触发条件：
  - `conditions`：允许您自定义其他触发条件，例如基于字段的特定规则或复杂逻辑判断。

这些触发条件参数可以在索引模板中进行配置，以根据具体需求定义何时触发滚动操作。可以同时使用多个触发条件，也可以选择仅使用其中部分来触发滚动操作。

例如，以下是一个示例索引模板的定义，其中包含了文档数量和时间两个触发条件：

```json
PUT _template/my_template
{
  "index_patterns": ["my_index*"],
  "settings": {
    "number_of_shards": 5
  },
  "mappings": {
    "_source": {
      "enabled": true
    }
  },
  "aliases": {
    "my_alias": {}
  },
  "rollover": {
    "max_docs": 100000,
    "max_age": "7d"
  }
}
```

以上示例中，当索引满足文档数量达到10万或存储时间超过7天的条件之一时，将触发滚动操作。

或者也可以直接调用  `_rollover`  API：

```json
# index_alias 必须是别名 而不能是索引的本名
POST /index_alias/_rollover
{
  "conditions": {
    "max_age":   "7d",
    "max_docs":  2,
    "max_size": "5gb"
  }
}
```
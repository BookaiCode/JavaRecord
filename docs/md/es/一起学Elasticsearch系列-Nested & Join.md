本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)


[TOC]

ES的 Nested 类型用于处理在一个文档中嵌套复杂的结构数据，而 Join 类型用于建立父子文档之间的关联关系。

## 嵌套类型：Nested

Elasticsearch没有内部对象的概念，因此，ES在存储复杂类型的时候会把对象的复杂层次结果扁平化为一个键值对列表。

**比如**：

```json
PUT my-index/_doc/1
{
  "group" : "fans",
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
```

上面的文档被创建之后，user数组中的每个json对象会以下面的形式存储

```json
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
```

`user.first`和 `user.last`字段被扁平化为多值字段，`first`和 `last`之间的关联丢失。

解决方法可以使用Nested类型，Nested属于object类型的一种，是Elasticsearch中用于复杂类型对象数组的索引操作，嵌套类型（Nested）允许在一个文档内部嵌套另一个文档，这使得可以在同一个文档中表示复杂的层次结构数据。

下面是关于如何定义和使用嵌套类型的示例：

定义映射（Mapping）：

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "comments": {
        "type": "nested",
        "properties": {
          "user": { "type": "keyword" },
          "message": { "type": "text" }
        }
      }
    }
  }
}
```

在上述示例中，我们创建了一个名为 "my_index" 的索引，并定义了一个 "comments" 字段作为嵌套类型。嵌套类型包含两个属性： "user" 和 "message"。

输入数据（Indexing）：

```json
POST /my_index/_doc
{
  "name": "Product A",
  "comments": [
    {
      "user": "User 1",
      "message": "Great product!"
    },
    {
      "user": "User 2",
      "message": "Needs improvement."
    }
  ]
}
```

在上述示例中，我们向索引 "my_index" 中插入了一个文档，其中 "comments" 字段包含了两个嵌套文档。

查询数据（Querying）：

```json
GET /my_index/_search
{
  "query": {
    "nested": {
      "path": "comments",
      "query": {
        "bool": {
          "must": [
            { "match": { "comments.user": "User 1" } },
            { "match": { "comments.message": "Great product!" } }
          ]
        }
      }
    }
  }
}
```

在上述示例中，我们使用嵌套查询（nested query）来搜索包含特定评论的文档。我们指定了路径为 "comments"，并在 `must` 子句中添加了匹配条件。

输出结果：

```json
{
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "hits": [
      {
        "_source": {
          "name": "Product A",
          "comments": [
            {
              "user": "User 1",
              "message": "Great product!"
            }
          ]
        }
      }
    ]
  }
}
```

在上述示例中，我们得到了一个匹配的文档，其中 "comments" 字段只包含了符合查询条件的嵌套文档。

### 参数

- `path`（必需）：指定嵌套字段的路径。它告诉 Elasticsearch 在哪个字段上应用嵌套查询。

- `score_mode`（可选）：指定如何计算嵌套文档的评分。

  avg （默认）：使用所有匹配的子对象的平均相关性得分。

  max：使用所有匹配的子对象中的最高相关性得分。

  min：使用所有匹配的子对象中最低的相关性得分。

  none：不要使用匹配的子对象的相关性分数。该查询为父文档分配得分为0。

  sum：将所有匹配的子对象的相关性得分相加。

- `inner_hits`（可选）：允许获取与嵌套文档匹配的内部结果。使用此参数可以检索与查询匹配的特定嵌套文档，并返回有关它们的信息。

- `ignore_unmapped`（可选）：如果设置为 `true`，则忽略没有嵌套字段映射的文档，并将其视为无匹配。默认情况下，设为 `false`。

- `nested`（可选）：表示查询是否应该应用于嵌套字段的上下文。默认情况下，设为 `true`。如果设置为 `false`，则将查询视为普通的非嵌套查询。

- `score_mode`（可选）：指定如何计算嵌套文档的评分。可选的值包括 `"none"`、`"avg"`、`"max"`、`"sum"` 和 `"min"`。默认情况下，使用 `"avg"`。

## 父子级关系：Join

连接数据类型是一个特殊字段，它在同一索引的文档中创建父/子关系。关系部分在文档中定义了一组可能的关系，每个关系是一个父名和一个子名。

父/子关系可以定义如下：

```json
PUT <index_name>
{
  "mappings": {
    "properties": {
      "<join_field_name>": { 
        "type": "join",
        "relations": {
          "<parent_name>": "<child_name>" 
        }
      }
    }
  }
}
```

常见的一个示例是创建一个索引来存储博客的数据。每个博客可以有多个评论，我们可以使用Join类型来建立博客和评论之间的父子关系。

首先，我们定义一个包含两个类型的索引：`blogs`和`comments`。`blogs`类型表示博客，而`comments`类型表示评论。我们将为`blogs`类型定义一个Join字段，用于与`comments`类型建立关联。

以下是一个简化的示例：

创建索引并定义映射：

```json
PUT my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "join_field": {
        "type": "join",
        "relations": {
          "blogs": "comments" 
        }
      }
    }
  }
}
```

添加博客文档：

```json
PUT my_index/_doc/1
{
  "title": "Elasticsearch Join 示例",
  "join_field": "blogs"
}
```

添加评论文档，并关联到博客：

```json
PUT my_index/_doc/2?routing=1
{
  "title": "很棒的博客",
  "join_field": {
    "name": "comments",
    "parent": "1"
  }
}
```

查询博客及其关联的评论：

```json
GET my_index/_search
{
  "query": {
    "has_child": {
      "type": "comments",
      "query": {
        "match_all": {}
      }
    }
  }
}
```

以上示例展示了如何使用Join类型在Elasticsearch中建立父子关系，并进行查询操作。实际使用时，可能需要根据自己的数据结构和查询需求进行适当的调整。

使用场景

**Join唯一合适应用场景是：当索引数据包含一对多的关系，并且其中一个实体的数量远远超过另一个的时候。比如：老师有 一万个学生**

`join`类型不能像关系数据库中的表链接那样去用，不论是 `has_child`或者是 `has_parent`查询都会对索引的查询性能有严重的负面影响。并且会触发 global ordinals。

Global Ordinals是一种用于优化字段的查询性能的技术。在使用Join类型时，如果启用了Global Ordinals特性，它将为Join字段创建全局有序的编号，以支持快速的父子文档查询。

当你执行具有Join字段的查询时，ES会使用Global Ordinals来识别匹配的父文档，并快速定位到对应的子文档。这样可以避免对所有文档进行扫描和过滤的开销，提高查询的效率。

需要注意的是，启用Global Ordinals可能会增加索引的内存使用量和一些额外的计算开销。因此，在决定是否启用Global Ordinals时，需要权衡查询性能和资源消耗之间的平衡。

**注意**

- 在索引父子级关系数据的时候必须传入routing参数，即指定把数据存入哪个分片，因为父文档和子文档必须在同一个分片上，因此，在获取、删除或更新子文档时需要提供相同的路由值。
- 每个索引只允许有一个 `join`类型的字段映射。
- 一个元素可以有多个子元素但只有一个父元素。
- 可以向现有连接字段添加新关系。
- 也可以向现有元素添加子元素，但前提是该元素已经是父元素。

### 参数

当使用Elasticsearch的Join类型进行查询时，以下是一些常用的参数和选项：

- `has_parent`和`has_child`：这两个查询参数用于在父子文档之间执行查询。您可以指定要匹配的父文档或子文档的类型以及具体的查询条件。
- `parent_id`：用于指定要查询的子文档的父文档ID。通过指定`parent_id`参数，您可以快速检索与特定父文档相关联的所有子文档。
- `inner_hits`：内部命中参数允许您在查询结果中获取与父文档或子文档匹配的内部命中结果。您可以使用`inner_hits`来检索与查询条件匹配的子文档或匹配的父文档及其关联的子文档。
- `ignore_unmapped`：当设置为true时，如果查询字段不存在映射或没有任何匹配的文档时，将忽略该查询并返回空结果。
- `max_children`：可用于限制每个父文档返回的子文档数量。

这些只是一些常见的参数和选项，根据你的实际需求，还可以使用其他参数来进一步细化查询。请参考Elasticsearch官方文档以获取更详细的参数和用法信息。
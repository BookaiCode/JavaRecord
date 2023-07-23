[TOC]

这章主要是介绍Elasticsearch中索引的基本操作API，即增删改查（CRUD）。

## 创建索引

```JSON
PUT /index?pretty
```

?pretty可加可不加，主要就是对输出进行格式化，更加好看点。

## 删除索引

```JSON
DELETE /index?pretty
```

## 查询数据

查询当前索引的信息

```JSON
GET /index/_search
//_search：查询 index 索引下的所有信息。
```

输出示例如下：

```JSON
{
  //消耗时间
  "took": 11,
  "timed_out": false,
  "_shards" : {
  "total": 1,
  "successful" : 1,
  "skipped": 0,
  "failed": 0
},
"hits": {
  "total": {
    "value": 0,
    "relation": "eq"
  },
  "max_ score": null,
  "hits": []
}
}
```

获取所有索引数据的信息

```JSON
GET _cat/indices?v
```

查询指定文档id

```JSON
GET /index/_doc/doc_id
```

## 添加 & 更新数据

```JSON
PUT /index/_doc/doc_id
{
 JSON数据
}

//例如：PUT /index/_doc/1
//{
//  "field1": "value1",
//  "field2": 123
//}
```

PUT也可以用于更新数据，比如我有一个文档有两个字段：name和age。我想更新name为：小明，可以这么写：

```JSON
PUT /index/_doc/1
{
"name": "小明"
}
```

**需要注意的是PUT既可以用于插入，也可以用于更新，所以PUT的更新是全量更新，而不是部分更新。也就是上面的语句执行之后，文档会被直接替换，只会有name字段，字段值为小明**。

如果我们想要部分更新的话，可以使用POST，示例如下：

```JSON
POST /index/_doc/id/_update
{
  "doc": {
    "name": "小明"
  }
}
```

把PUT换位POST，并把更新的字段包进doc里，就能实现更新部分字段。除了上面那种写法外，还可以使用下面这种写法，更推荐使用下面这种写法：

```JSON
POST /index/_update/1
{
"doc": {
"name": "小明"
}
}
```

## cat命令

cat命令在es中会经常使用，下面介绍cat命令中常用的几个命令。

### 公共参数

cat命令组成形式是：`GET /_cat/indices?format=json&pretty`， `?`之前是命令，之后是参数，多个参数用`&`分隔。公共参数有下：

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

## 常用命令

### aliases 显示别名

```JSON
GET /_cat/aliases
```

GET /_cat/aliases是获取所有别名，如果想获得某个索引的别名可以使用：`GET index/alias`。

### allocation 显示每个节点的分片数和磁盘使用情况

```JSON
GET /_cat/allocation
```

### count 显示整个集群或者索引的文档个数

```JSON
GET /_cat/count
GET /_cat/count/index
```

### fielddata 显示每个节点字段所占的堆空间

```JSON
GET /_cat/fielddata
GET /_cat/fielddata?fields=name,addr
```

### health 显示集群是否健康

```JSON
GET /_cat/health
```

### indices 显示索引的情况

```JSON
GET /_cat/indices
GET /_cat/indices/index
```

### master 显示master节点信息

```JSON
GET /_cat/master
```

### nodes 显示所有node节点信息

```JSON
GET /_cat/nodes
```

### recovery 显示索引恢复情况

当索引迁移的任何时候都可能会出现恢复情况，例如，快照恢复、复制更改、节点故障或节点启动期间。

```JSON
GET /_cat/recovery
```

### thread_pool 显示每个节点线程运行情况。

```JSON
GET /_cat/thread_pool
GET /_cat/thread_pool/bulk
GET /_cat/thread_pool/bulk?h=id,name,active,rejected,completed
```

### shards 显示每个索引各个分片的情况

展示索引的各个分片，主副分片，文档个数，所属节点，占存储空间大小

```JSON
GET /_cat/shards
GET /_cat/shards/index
GET _cat/shards?h=index,shard,prirep,state,unassigned.reason
```

分片的状态：`INITIALIZING`初始化；`STARTED`分配完成；`UNASSIGNED`不能分配；可以通过`unassigned.reason`属性查看不能分配的原因。

### segments 显示每个segment的情况

包括属于索引，节点，主副，文档数等

```JSON
GET /_cat/segments
GET /_cat/segments/index
```

### templates 显示每个template的情况

```JSON
GET /_cat/templates
GET /_cat/templates/mytempla*
```
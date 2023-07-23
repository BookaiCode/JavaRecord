[TOC]

这篇讲解Elasticsearch中非常重要的一个概念Mapping，Mapping是索引必不可少的组成部分。

## Mapping 的基本概念

**Mapping 也称之为映射，定义了 ES 的索引结构、字段类型、分词器等属性，是索引必不可少的组成部分**。

ES 中的 mapping 有点类似与关系型数据库中“表结构”的概念，在 MySQL 中，表结构里包含了字段名称，字段的类型还有索引信息等。在 Mapping 里也包含了一些属性，比如字段名称、类型、字段使用的分词器、是否评分、是否创建索引等属性。

### 查看索引 Mapping

```JSON
//查看索引完整的mapping
GET /index/_mappings
//查看索引指定字段的mapping
GET /index/_mappings/field/<field_name>
```

## 字段数据类型

映射的数据类型也就是 ES 索引支持的数据类型，其概念和 MySQL 中的字段类型相似，但是具体的类型和 MySQL 中有所区别，最主要的区别就在于 ES 中支持可分词的数据类型，如：Text 类型，可分词类型是用以支持全文检索的，这也是 ES 生态最核心的功能。

### 数字类型

- **long**：64 位有符号整形。
- **integer**：32 位有符号整形。
- **short**：16 位有符号整形。
- **byte**：8位有符号整形。
- **double**：双精度 64位浮点类型。
- **float**：单精度 64位浮点类型。
- **half_float**：半精度 64位浮点类型。
- **scaled_float**：缩放类型浮点数，按固定 double 比例因子缩放。
- **unsigned_long**：无符号 64 位整数。

### 基本数据类型

- **binary**：Base64 字符串二进制值。
- **boolean**：布尔类型，接收 ture 和 false 两个值。
- **alias**：字段别名。

### Keywords 类型

- **keyword**：适用于索引结构化的字段，可以用于过滤、排序、聚合。keyword类型的字段只能通过精确值搜索到。如 Id、姓名这类字段应使用 keyword。
- **constant_keyword**：始终包含相同值的关键字字段。
- **wildcard**：可针对类似 grep 的场景。

### Dates（时间类型）

- **date**：JSON 没有日期数据类型，因此 Elasticsearch 中的日期可以是以下三种：
  - 包含格式化日期的字符串：例如 "2015-01-01"、 "2015/01/01 12:10:30"。
  - 时间戳：表示自"1970年 1 月 1 日"以来的毫秒数/秒数。
  - date_nanos：此数据类型是对 date 类型的补充。但是有一个重要区别。date 类型存储最高精度为毫秒，而date_nanos 类型存储日期最高精度是纳秒，但是高精度意味着可存储的日期范围小，即：从大约 1970 到 2262。

### 对象类型

- **object**：非基本数据类型之外，默认的 json 对象为 object 类型。
- **flattened**：单映射对象类型，其值为 json 对象。
- **nested** ：嵌套类型。
- **join**：父子级关系类型。

### 空间数据类型

- **geo_point**：纬度和经度点。
- **geo_shape**：复杂的形状，例如多边形。
- **point**：任意笛卡尔点。
- **shape**：任意笛卡尔几何。

### 文档排名类型

- **dense_vector**：记录浮点值的密集向量。
- **rank_feature**：记录数字特征以提高查询时的命中率。
- **rank_features**：记录数字特征以提高查询时的命中率。

### 文本搜索类型

- **text**：文本类型。
- **annotated-text：**包含特殊文本标记，用于标识命名实体。
- **completion** ：用于自动补全，即搜索推荐。
- **search_as_you_type：** 类似文本的字段，经过优化为提供按类型完成的查询提供现成支持。
- **token_count**：文本中的标记计数。

## 两种映射类型

### 自动映射：Dynamic Field Mapping

| **field type** | **dynamic**                        |
| -------------- | ---------------------------------- |
| true/false     | boolean                            |
| 小数           | float                              |
| 数字           | long                               |
| object         | object                             |
| 数组           | 取决于数组中的第一个非空元素的类型 |
| 日期格式字符串 | date                               |
| 数字类型字符串 | float/long                         |
| 其他字符串     | text + keyword                     |

除了上述字段类型之外，其他类型都必须显式映射，也就是必须手工指定，因为其他类型ES无法自动识别。

### 显式映射 Expllcit Field Mapping

例如：

```JSON
PUT test_mapping
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      },
      "name": {
        "type": "text",
        "fields": {
          "name2": {
            "type": "keyword",
            "ignore_ above": 256
          }
        }
      },
      "age": "byte"
    }
  }
}
```

## 映射参数

- **index**：是否对创建对当前字段创建倒排索引，默认 true，如果不创建索引，该字段不会通过索引被搜索到，但是仍然会在 source 元数据中展示。
- **analyzer**：指定分析器（character filter、tokenizer、Token filters）。
- **boost**：对当前字段相关度的评分权重，默认1。
- **coerce**：是否允许强制类型转换，为 true的话 “1”能被转为 1， false则转不了。
- **copy_to**：该参数允许将多个字段的值复制到组字段中，然后可以将其作为单个字段进行查询。
- **doc_values**：为了提升排序和聚合效率，默认true，如果确定不需要对字段进行排序或聚合，也不需要通过脚本访问字段值，则可以禁用doc值以节省磁盘空间（不支持text和annotated_text）。
- **dynamic**：控制是否可以动态添加新字段
  - **true** 新检测到的字段将添加到映射中（默认）。
  - **false** 新检测到的字段将被忽略。这些字段将不会被索引，因此将无法搜索，但仍会出现在_source返回的匹配项中。这些字段不会添加到映射中，必须显式添加新字段。
  - **strict** 如果检测到新字段，则会引发异常并拒绝文档。必须将新字段显式添加到映。
- **eager_global_ordinals**：用于聚合的字段上，优化聚合性能，但不适用于 Frozen indices。
  - **Frozen indices**（冻结索引）：有些索引使用率很高，会被保存在内存中，有些使用率特别低，宁愿在使用的时候重新创建，在使用完毕后丢弃数据，Frozen indices 的数据命中频率小，不适用于高搜索负载，数据不会被保存在内存中，堆空间占用比普通索引少得多，Frozen indices是只读的，请求可能是秒级或者分钟级。
- **enable**：是否创建倒排索引，可以对字段操作，也可以对索引操作，如果不创建索引，仍然可以检索并在_source元数据中展示，谨慎使用，该状态无法修改。**enable的作用和index类似，区别就是enable可以对全局进行设置**。例如：

```JSON
PUT my_index
{
  "mappings": {
    "enabled": false
  }
}
```

- **fielddata**：查询时内存数据结构，在首次用当前字段聚合、排序或者在脚本中使用时，需要字段为fielddata数据结构，并且创建倒排索引保存到堆中。
- **fields**：给field创建多字段，用于不同目的（全文检索或者聚合分析排序）。
- **format**：格式化。例如：

```JSON
"date": {
  "type":  "date",
  "format": "yyyy-MM-dd"
}
```

- **ignore_above**：超过长度将被忽略。
- **ignore_malformed**：忽略类型错误。
- **index_options**：控制将哪些信息添加到反向索引中以进行搜索和突出显示。仅用于text字段。
- **Index_phrases**：提升 exact_value 查询速度，但是要消耗更多磁盘空间。
- **Index_prefixes**：前缀搜索。
  - **min_chars**：前缀最小长度> 0，默认 2（包含）
  - **max_chars**：前缀最大长度< 20，默认 5（包含）
- **meta**：附加元数据。
- **normalizer**：normalizer 参数用于解析前（索引或者查询时）的标准化配置。
- **norms**：是否禁用评分（在 filter 和聚合字段上应该禁用）。
- **null_value**：为 null 值设置默认值。
- **position_increment_gap**：参考：https://blog.csdn.net/wlei0618/article/details/128189190
- **properties**：除了mapping还可用于object的属性设置。
- **search_analyzer**：设置单独的查询时分析器，**如果定义了analyzer而没有定义search_analyzer，则search_analyzer的值默认会和analyzer保持一致，如果两个都没有定义，则默认是："standard"。analyzer针对的是元数据，而search_analyzer针对的是传入的搜索词**。
- **similarity**：为字段设置相关度算法，和评分有关。支持BM25、classic（TF-IDF）、boolean。
- **store**：设置字段是否仅查询。
- **term_vector**：运维参数。

## Text 和 Keyword 类型

### Text 类型

#### 概述

当一个字段是要被全文搜索的，比如 Email 内容、产品描述，这些字段应该使用 text 类型。设置 text 类型以后，字段内容会被分析，在生成倒排索引以前，字符串会被分析器分成一个一个词项。text类型的字段不用于排序，很少用于聚合。

#### 注意事项

- 适用于全文检索：如 match 查询。
- 文本字段会被分词。
- 默认情况下，会创建倒排索引。
- 自动映射器会为 Text 类型创建 Keyword 字段。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScMvaqlT5NT0m2n3F5E1sljQH2OKwFiaibIolpkHJa0iazFkAmmpN3hVkpibWa6ahlO4oE8XdEn32o5gLA/640?wx_fmt=png)

### Keyword 类型

#### 概述

Keyword 类型适用于不分词的字段，如姓名、Id、数字等。如果数字类型不用于范围查找，用 Keyword 的性能要高于数值类型。

#### 语法和语义

如当使用 keyword 类型查询时，其字段值会被作为一个整体，并保留字段值的原始属性。

```JSON
GET index/_search
{
  "query": {
    "match": {
      "title.keyword": "测试文本值"
    }
  }
}
```

#### 注意事项

- Keyword 不会对文本分词，会保留字段的原有属性，包括大小写等。
- Keyword 仅仅是字段类型，而不会对搜索词产生任何影响。
- Keyword 一般用于需要精确查找的字段，或者聚合排序字段。
- Keyword 通常和 Term 搜索一起用。
- Keyword 字段的 ignore_above 参数代表其截断长度，默认 256，**如果超出长度，字段值会被忽略，而不是截断，忽略指的是会忽略这个字段的索引，搜索不到，但数据还是存在的**。

## 映射模板

### 简介

之前讲过的映射类型或者字段参数，都是为确定的某个字段而声明的，如果希望对符合某类要求的特定字段制定映射，就需要用到映射模板：Dynamic templates。映射模板有时候也被称作：自动映射模板、动态模板等。

**之前设置mapping的时候，我们明确知道字段名字，但是当我们不确定字段名字的时候该怎么设置mapping？映射模板就是用来解决这种场景的**。

### 用法

#### 基本语法

```JSON
"dynamic_templates": [
  {
    "my_template_name": { 
      ... match conditions ... 
      "mapping": { ... } 
    }
  },
  ...
]
```

#### Conditions参数

- **match_mapping_type** ：主要用于对数据类型的匹配。
- **match 和 unmatch**：用于对字段名称的匹配。

### 案例

```JSON
PUT test_dynamic_template

{
  "mappings": {
    "dynamic_templates": [
      {
        "integers": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "longs_as_strings": {
          "match_mapping_type": "string",
          "match": "num_*",
          "unmatch": "*_text",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}
```

以上代码会产生以下效果：

- 所有 long 类型字段会默认映射为 integer。
- 所有文本字段，如果是以 num_ 开头，并且不以 _text 结尾，会自动映射为 keyword 类型。
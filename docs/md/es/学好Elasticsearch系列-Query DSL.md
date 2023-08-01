[TOC]

DSL是Domain Specific Language的缩写，指的是为特定问题领域设计的计算机语言。这种语言专注于某特定领域的问题解决，因而比通用编程语言更有效率。

在Elasticsearch（ES）中，DSL指的是Elasticsearch Query DSL，一种以JSON形式表示的查询语言。通过这种语言，用户可以构建复杂的查询、排序和过滤数据等操作。这些查询可以是全文搜索、分面/聚合搜索，也可以是结构化的搜索。

## 查询上下文

使用query关键字进行检索，倾向于相关度搜索，故需要计算评分。搜索是Elasticsearch最关键和重要的部分。

在查询上下文中，一个查询语句表示一个文档和查询语句的匹配程度。无论文档匹配与否，查询语句总能计算出一个相关性分数在`_score`字段上。

## 相关度评分：_score

相关度评分用于对搜索结果排序，评分越高则认为其结果和搜索的预期值相关度越高，即越符合搜索预期值，默认情况下评分越高，则结果越靠前。在7.x之前相关度评分默认使用TF/IDF算法计算而来，7.x之后默认为BM25。

## 源数据：_source

source字段包含索引时原始的JSON文档内容，字段本身不建立索引（因此无法进行搜索），但是会被存储，所以当执行获取请求是可以返回source字段。

虽然很方便，但是source字段的确会对索引产生存储开销，因此可以禁用source字段，达到节省存储开销的目的。可以通过以下接口进行关闭。

```JSON
PUT my_index
{
  "mappings": {
    "_source": {
      "enabled": false
    }
  }
}
```

但是需要注意的是这么做会带来一些弊端，_source禁用会导致如下功能无法使用：

- 不支持update、update_by_query和reindex API。
- 不支持高亮。
- 不支持reindex、更改mapping分析器和版本升级。

**总结：在禁用source之前，应该仔细考虑是否需要进行此操作。如果只是希望降低存储的开销，可以压缩索引比禁用source更好。**

## 数据源过滤器

例如，假设你的应用只需要获取部分字段（如"name"和"price"），而其他字段（如"desc"和"tags"）不经常使用或者数据量较大，导致传输和处理这些额外的数据会增加网络开销和处理时间。在这种情况下，通过设置includes和excludes可以有效地减少每次请求返回的数据量，提高效率。例如：

```JSON
PUT product
{
  "mappings": {
    "_source": {
      "includes": ["name", "price"],
      "excludes": ["desc", "tags"]
    }
  }
}
```

**Including：**结果中返回哪些field。

**Excluding：**结果中不要返回哪些field，不返回的field不代表不能通过该字段进行检索，因为元数据不存在不代表索引不存在，Excluding优先级比Including更高。

> 需要注意的是，尽管这些设置会影响搜索结果中_source字段的内容，但并不会改变实际存储在Elasticsearch中的数据。也就是说，"desc"和"tags"字段仍然会被索引和存储，只是在获取源数据时不会被返回。

在mapping中定义这种方式不推荐，因为mapping不可变。我们可以在查询过程中使用_source指定返回的字段，如下：

```JSON
GET product/_search
{
  "_source": {
    "includes": ["owner.*", "name"],
    "excludes": ["name", "desc", "price"]
  },
  "query": {
    "match_all": {}
  }
}
```

Elasticsearch的`_source`字段在查询时支持使用通配符（wildcards）来包含或排除特定字段。使得能够更灵活地操纵返回的数据。

关于规则，可以参考以下几点:

- *：匹配任意字符序列，包括空序列。
- ?：匹配任意单个字符。
- [abc]： 匹配方括号内列出的任意单个字符。例如，[abc]将匹配"a", "b", 或 "c"。

请注意，通配符表达式可能会导致查询性能下降，特别是在大型索引中，因此应谨慎使用。

## 全文检索

全文检索是Elasticsearch的核心功能之一，它可以高效地在大量文本数据中寻找特定关键词。

在Elasticsearch中，全文检索主要依靠两个步骤："分析"（Analysis）和"查询"（Search）。

1. 分析: 当你向Elasticsearch索引一个文档时，会进行"分析"处理，将原始文本数据转换成称为"tokens"或"terms"的小片段。这个过程可能包括如下操作：
   - 切分文本（Tokenization）
   - 将所有字符转换为小写（Lowercasing）
   - 删除常见但无重要含义的单词（Stopwords）
   - 提取词根（Stemming）
2. 查询: 当执行全文搜索时，查询字符串也会经过类似的分析过程，然后再与已经分析过的索引进行比对，找出匹配的结果并返回。

Elasticsearch提供了许多种全文搜索的查询类型，例如：

- Match Query: 最基本的全文搜索查询。
- Match Phrase Query: 用于查找包含特定短语的文档。
- Multi-Match Query: 类似Match Query，但可以在多个字段上进行搜索。
- Query String Query: 提供了丰富的搜索语法，可以执行复杂的、灵活的全文搜索。

### match：匹配包含某个term的子句

```JSON
GET product/_search
{
  "query": {
    "match": {
      "name": "xiaomi nfc phone"
    }
  }
}
```

上面的搜索语句，只要文档的"name"字段包含"xiaomi"、"nfc"或者"phone"中的任何一个词，就会被视为匹配。

### match_all：匹配所有结果的子句

`match_all` 是 Elasticsearch 中的一个查询类型，它匹配所有文档，不需要任何参数。

```JSON
GET product/_search
{
  "query": {
    "match_all": {}
  }
}
```

上面的语句等价于：

```JSON
GET /product/_search
```

这个查询将会返回索引中的所有文档。这通常用于在没有特定搜索条件时获取所有的文档，或者与其他查询结合使用（如过滤器）。

需要注意，由于 `match_all` 查询可能返回大量的数据，所以一般在使用时都会与分页（pagination）功能结合起来，这样可以控制返回结果的数量，避免一次性加载过多数据导致的性能问题。例如，你可以使用 `from` 和 `size` 参数来限制返回结果：

```JSON
GET /_search
{
  "query": {
    "match_all": {}
  },
  "from": 10,
  "size": 10
}
```

### multi_match：多字段条件

`multi_match` 查询是 Elasticsearch 中用来在多个字段上执行全文查询的功能。它接受一个查询字符串和一组需要在其中执行查询的字段列表。例如：

```JSON
GET /_search
{
  "query": {
    "multi_match" : {
      "query":  "xiaomi nfc phone", 
      "fields": [ "name", "description" ] 
      }
  }
}` 
```

这个查询会在 "name" 和 "description" 两个字段中查找包含 "xiaomi nfc phone" 的文档。

`multi_match` 还支持多种类型的匹配模式，如：`best_fields`, `most_fields`, `cross_fields`, `phrase`, `phrase_prefix`等。这些类型的行为略有不同，可以按照实际需求进行选择。

例如，“best_fields” 类型会从指定的字段中挑选分数最高的匹配结果计算最终得分，而“most_fields” 类型则会在每个字段中都寻找匹配项并将其分数累加起来。

需要注意的是，当使用 `multi_match` 查询时，如果字段不同，其权重可能也会不同。你可以通过在字段名后面添加尖括号（^）和权重值来调整特定字段的权重。例如，`"fields": [ "name^3", "description" ]`表示在"name"字段中的匹配结果权重是"description"字段的三倍。

### match_phrase：短语查询

`match_phrase` 是 Elasticsearch 中的一种全文查询类型，它用于精确匹配包含指定短语的文档。match_phrase 查询需要字段值中的单词顺序与查询字符串中的单词顺序完全一致。

例如：

```JSON
GET /_search
{
  "query": {
    "match_phrase": {
      "message": "this is a test"
    }
  }
}
```

这个查询将会找到"message"字段中包含完整短语"this is a test"的所有文档。

此外，`match_phrase` 查询还有一个 `slop` 参数，可以定义词组中的词语可能存在的位置偏移量。例如，如果将 `slop` 设置为 1，则查询 "this is a test" 也可匹配 "this is test a"，因为 "a" 和 "test" 只需移动一个位置即可匹配。

## Query String

Query String Query是Elasticsearch中的一种查询方式，它允许你使用特定的搜索语法来进行复杂的、灵活的查询。

Query String Query是基于Lucene Query Parser解析器的，因此支持丰富的搜索语法，包括但不限于：

- 基本文本查询: "quick brown fox"
- 逻辑操作符 (AND, OR, NOT): "quick AND brown"
- 范围查询: "age:[18 TO 30]"
- 通配符查询: "qu?ck br*wn"
- 分组: "(quick OR brown) AND fox"
- 字段指定查询: "title:quick"

下面是几个例子：

### 查询所有

```JSON
GET /product/_search
```

### 分页

```JSON
GET /product/_search?from=0&size=2&sort=price:asc
```

### 精准匹配 exact value

```JSON
GET /product/_search?q=date:2021-06-01
```

### _all搜索 相当于在所有有索引的字段中检索

all搜索与精准匹配就是带不带字段参数的区别，如果把index索引禁用，则all搜索不会去该字段上查询。

```JSON
GET /product/_search?q=2021-06-01
```

## 精准查询-Term query

精确查询用于查找包含指定精确值的文档，而不是执行全文搜索。

### term：匹配和搜索词项完全相等的结果

举个例子：

```JSON
GET /_search
{
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

这个查询会找到"user"字段精确匹配"kimchy"的所有文档。

需要注意的是，`term` 查询对大小写敏感，并且不会进行分词处理。也就是说，如果你在使用 `term` 查询时输入了一个完整的句子，它将尝试查找与这个完整句子精确匹配的文档，而不是把句子拆分成单词进行匹配。

#### term和match_phrase的区别

`term` 查询和 `match_phrase` 查询是 Elasticsearch 提供的两种查询方式，它们都用于查找文档，但主要的区别在于如何解析查询字符串以及匹配的精确度。

1. `term` 查询：这种查询对待查询字符串为一个完整的单位，不进行分词处理，并且大小写敏感。它可以在文本、数值或布尔类型字段上使用，通常用于精确匹配某个字段的确切值。
2. `match_phrase` 查询：这种查询把查询字符串当作一种短语来匹配。查询字符串会被分词器拆分成单独的词项，然后按照词项在查询字符串中的顺序去匹配文档。只有当文档中的词项顺序与查询字符串中的顺序完全一致时才能匹配成功，match_phrase 查询通常对大小写不敏感，除非你的字段映射或索引设置更改了这个行为。

简单来说，`term` 查询更多的是做精确的、字面的匹配，而 `match_phrase` 则是做短语匹配，在搜索结果的精确度上，`term` 查询比 `match_phrase` 更高。

### terms：匹配和搜索词项列表中任意项匹配的结果

`terms` 查询用于匹配指定字段中包含一个或多个值的文档。这是一个精确匹配查询，不会像全文查询那样对查询字符串进行分析。

假设你有一个 "user" 的字段，并且你想找到该字段值为 "John" 或者 "Jane" 的所有文档，你可以使用 `terms` 查询：

```JSON
GET /_search
{
  "query": {
    "terms" : {
      "user" : ["John", "Jane"],
      "boost" : 1.0
    }
  }
}
```

上面的查询将返回所有"user" 字段等于 "John" 或者 "Jane" 的文档。

其中`boost` 参数用于增加或减少特定查询的相对权重。它将改变查询结果的相关性分数（_score），以影响最终结果的排名。

例如，在上述 `terms` 查询中，`boost` 参数被设置为 1.0。这意味着如果字段 "user" 的值包含 "John" 或 "Jane"，那么其相关性分数（_score）就会乘以 1.0。因此，这个设置实际上并没有改变任何东西，因为乘以 1 不会改变原始分数。但是，如果你将 `boost` 参数设置为大于 1 的数，那么匹配的文档的 _score 将会提高，反之则会降低。

### range：范围查找

range 查询允许你查找位于特定范围内的值。这对于日期、数字或其他可排序类型的字段非常有用。

下面的语句会查询出age字段大于等于10，小于等于20的文档。

例子1：假设你有一些表示博客文章的文档，每个文档都有一个发表日期，并且你想找出在特定日期范围内发布的所有文章，你可以使用 `range` 查询来实现这一目标

```JSON
GET /_search
{
  "query": {
    "range" : {
      "date" : {
        "gte" : "2020-01-01",
        "lte" : "2020-12-31",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
```

在上面的查询中，`range` 查询被用来查找字段 "date" 的值在 "2020-01-01" 和 "2020-12-31"（包含）之间的所有文档。

`range` 查询支持以下运算符：

- `gt`：大于 (greater than)
- `gte`：大于等于 (greater than or equal to)
- `lt`：小于 (less than)
- `lte`：小于等于 (less than or equal to)

例子2：下面的语句会查询出date字段1天前的文档，其中now表示当前时间。

```JSON
GET product/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "now-1d/d",
        "lt": "now/d"
      }
    }
  }
}
```

例子3：下面的语句会查询出date字段小于当前时间，大于2021-04-15T08:00:00的文档。time_zone表示时区，意思就是原文档中的数据会被+8小时再去搜索，例如原文档有条数据是：2021-04-15。则该数据能被查询出来。

```JSON
GET product/_search
{
  "query": {
    "range": {
      "date": {
        "time_zone": "+08:00",
        "gte": "2021-04-15T08:00:00",
        "lt": "now"
      }
    }
  }
}
```

## 过滤器-Filter

过滤器（Filter）是一种特殊类型的查询，它不关心评分 (_score)，只关心是否匹配。基于这个原因，过滤器比标准的全文查询更快并且能被缓存。

一个典型的使用场景是布尔查询 (`bool`), 它有两个重要的部分：`must` 和 `filter`。`must` 部分用于全文搜索，`filter` 部分用于过滤结果。看一个例子：

```JSON
GET /_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "quick" }}
      ],
      "filter": [
        { "term":  { "published": true }}
      ]
    }
  }
}
```

在这个查询中，`bool` 查询包含了一个 `must` 子句和一个 `filter` 子句。`must` 子句会执行全文搜索并对结果进行评分。在这个例子中，它会找出所有标题包含"quick"的文章。

`filter` 子句则会在 `must` 子句的基础上进一步过滤结果。在这个例子中，它会筛选出那些已经发布的文章。这个过滤操作不会影响到评分，因为它只关心是否匹配。

总的来说，过滤器非常适合用于分类、范围查询或者确认某个字段是否存在等场景。过滤器的效率高并且可以被缓存，所以在大型数据集上性能表现良好。

### Filter缓存机制

在 Elasticsearch 中，过滤查询结果的缓存机制是非常重要的一个性能优化手段。由于过滤器（filter）只关心是否匹配，而不关心评分 (_score)，因此它们的结果可以被缓存以提高性能。

每次 filter 查询执行时，Elasticsearch 都会生成一个名为 "bitset" 的数据结构，其中每个文档都对应一个位（0 或 1），表示这个文档是否与 filter 匹配。这个 bitset 就是被存储在缓存中的部分。

如果相同的 filter 查询再次执行，Elasticsearch 可以直接从缓存中获取这个 bitset，而不需要再次遍历所有的文档来找出哪些文档符合这个 filter。这大大提高了查询速度，并减少了 CPU 使用。

这种缓存策略特别适合那些重复查询的场景，例如用户界面的过滤器和类似的功能，因为他们通常会产生很多相同的 filter 查询。

然而，值得注意的是，虽然这种缓存可以显著改善查询性能，但也会占用内存空间。如果你有很多唯一的过滤条件，那么过滤器缓存可能会变得很大，从而导致内存问题。这就需要你对使用的过滤器进行适当的管理和限制。

另外，Elasticsearch 默认情况下会自动选择哪些过滤器进行缓存，考虑到查询频率和成本等因素。你也可以手动配置某个特定的 filter 是否需要进行缓存。

## 组合查询-Bool query

组合查询可以组合多个查询条件，bool查询也是采用more_matches_is_better的机制，因此满足must和should子句的文档将会合并起来计算分值。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOyKiabe9ph3M760B405ANeTQKKL6ArXcTAUI1g5KV80ib0zNicy5L2usIJuPtDAGvsVa5ibs5Sch5GZw/640?wx_fmt=png)

boot和minumum_should_match是参数，其他四个都是查询子句。

- **must**：必须满足子句（查询）必须出现在匹配的文档中，并将有助于得分。
- **filter**：过滤器不计算相关度分数。
- **should**：满足 or子句（查询）应出现在匹配的文档中。
- **must_not**：必须不满足，不计算相关度分数 ，not子句（查询）不得出现在匹配的文档中。子句在过滤器上下文中执行，这意味着计分被忽略，并且子句被视为用于缓存。

例子1：下面的语句表示：包含"xiaomi"或"phone" 并且包含"shouji"的文档例子：

```JSON
GET product/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "name": "xiaomi phone"
                    }
                },
                {
                    "match_phrase": {
                        "desc": "shouji"
                    }
                }
            ]
        }
    }
}
```

### should与must或filter一起使用

当 `should` 子句与 `must` 或 `filter` 子句一起使用时，这时候需要注意了！只要满足了 `must` 或 `filter` 的条件，`should` 子句就不再是必须的。换句话说，如果存在一个或者多个 `must` 或 `filter` 子句，那么 `should` 子句的条件会被视为可选。

然而，如果 `should` 子句与 `must_not` 子句单独使用（也就是没有 `must` 或 `filter`），则至少需要满足一个 `should` 子句的条件。

这里有一个例子来说明：

```JSON
GET /_search
{
  "query": {
    "bool": {
      "must": [
        { "term": { "user": "kimchy" }}
      ],
      "filter": [
        { "term":  { "tag": "tech" }}
      ],
      "should": [
        { "term": { "tag": "wow" }},
        { "term": { "tag": "elasticsearch" }}
      ]
    }
  }
}
```

在这个查询中，`must` 和 `filter` 子句的条件是必须满足的，而 `should` 子句的条件则是可选的。如果匹配的文档同时满足 `should` 子句的条件，那么它们的得分将会更高。

那如果我们一起使用的时候想让should满足该怎么办？这时候`minimum_should_match` 参数就派上用场了。

### minimum_should_match

minimum_should_match参数定义了在 `should` 子句中至少需要满足多少条件。

例如，如果你有5个 `should` 子句并且设置了 `"minimum_should_match": 3`，那么任何匹配至少三个 `should` 子句的文档都会被返回。

这个参数可以接收绝对数值（如 `2`）、百分比（如 `30%`）、和组合（如 `3<90%` 表示至少匹配3个或者90%，取其中较大的那个）等不同类型的值。

注意：如果 `bool` 查询中只有 `should` 子句（没有 `must` 或 `filter`），那么默认情况下至少需要匹配一个 `should` 条件，也就是`minimum_should_match`默认值是1，除非 `minimum_should_match` 明确设定为其他值。如果包含 `must` 或 `filter`的情况下`minimum_should_match`默认值 0。

所以我们可以在包含`must` 或 `filter`的情况下，设置`minimum_should_match`值来满足`should`子句中的条件。
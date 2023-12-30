本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)


[TOC]

DSL是Domain Specific Language的缩写，指的是为特定问题领域设计的计算机语言。这种语言专注于某特定领域的问题解决，因而比通用编程语言更有效率。

在Elasticsearch中，DSL指的是Elasticsearch Query DSL，是一种以JSON形式表示的查询语言。通过这种语言，用户可以构建复杂的查询、排序和过滤数据等操作。这些查询可以是全文搜索、聚合搜索，也可以是结构化的搜索。

## 查询上下文

搜索是Elasticsearch中最关键和重要的部分，使用`query`关键字进行检索，更倾向于相关度搜索，故需要计算评分。

在查询上下文中，一个查询语句表示一个文档和查询语句的匹配程度。无论文档匹配与否，查询语句总能计算出一个相关性分数在`_score`字段上。

### 相关度评分：score

相关度评分用于对搜索结果排序，评分越高则认为其结果和搜索的预期值相关度越高，即越符合搜索预期值，默认情况下评分越高，则结果越靠前。在7.x之前相关度评分默认使用TF/IDF算法计算而来，7.x之后默认为BM25。

score是根据各种因素计算出来的，包括：

- **Term Frequency（词频）**：一个词在文档中出现的次数越多，score就越高。
- **Inverse Document Frequency（逆文档频率**）：一个词在所有文档中出现的次数越少，score就越高。
- **Field Length Norm（字段长度规范**）：字段的长度越短，score就越高。

这三个因素共同决定了score的值。然而，你也可以通过设置自定义评分或者禁用评分来影响score的计算。

#### TF/IDF & BM25

TF/IDF是一种在信息检索和文本挖掘中广泛使用的统计方法，用于评估一个词语对于一个文件集或一个语料库中的一个文件的重要程度。名称中的TF表示“术语频率”，IDF表示“逆向文件频率”。

- **TF (Term Frequency)** ：这是衡量词在文档中出现的频率。通常来说，一个词在文档中出现的次数越多，其重要性就可能越大。但这并不总是正确的，比如在很多英文文档中，“the”、“and”等词出现的频率非常高，但我们并不能因此认为它们就非常重要。因此，需要结合 IDF 来使用。
- **IDF (Inverse Document Frequency)** ：这是衡量词是否常见的度量。如果某个词在许多文档中都出现，那么它可能并不具有区分性，对于搜索和分类的帮助就不大。例如，每篇英文文章中都会出现的“the”对于区分文章内容就没有什么帮助。所以，如果一个词在所有文档中出现得越多，那么其 IDF 值就会越小，相反，如果一个词很少在文档中出现，那么其 IDF 值就会较大。

TF-IDF 会将这两个因子结合起来，为每个词产生一个权重。具有较高 TF-IDF 分数的词被认为在文档中更重要。通过这种方式，ES 能够提供相关性排序，使得包含用户查询词汇的最相关文档排在搜索结果的前面。

BM25是一种更先进的排名函数，也是基于TF/IDF的一种改进型方法。它引入了两个新概念:

- **文档长度归一化**：长文档可能会有更多的关键词，但这并不意味着它与查询更相关。BM25通过调整文档长度来解决这个问题。
- **饱和度**：在TF/IDF中，词项的出现频率越高，其重要性就越大。然而在实践中，一旦一个词在文档中出现过，再次出现时增加的相关性可能会降低。BM25通过设置一个饱和点来解决这个问题，超过这个点，词的权重增加就会变得不那么敏感。

总结而言，BM25是TF/IDF的改进版，通过文档长度归一化和频率饱和度控制来优化搜索结果。

### 源数据：source

`_source`字段包含索引时原始的JSON文档内容，字段本身不建立索引（因此无法进行搜索），但是会被存储，所以当执行获取请求是可以返回`_source`字段。

虽然很方便，但是`_source`字段的确会对索引产生存储开销，你可以通过关闭`_source`字段来节省空间，但这通常不建议，因为有了原始数据，我们可以对数据进行重新索引，并且在获取数据时也更加灵活。

如果你禁用了`_source`字段，那么会有以下几个影响：

- **无法获取原始数据**：当你查询某个文档时，你将无法获取到原始的`_source`字段内容，因为它没有被存储在Elasticsearch中。
- **更新和重新索引的问题**：如果你想更新文档或者执行重新索引操作，可能会遇到问题，因为这两种操作都需要原始的`_source`字段。
- **脚本字段和某些Aggregations可能受到影响**：如果你正在使用脚本字段或者依赖`_source`字段的Aggregations，那么禁用`_source`可能导致这些特性出问题。

下面是一些使用`_source`字段的例子：



1. 在索引文档时启用/禁用`_source`：

```json
PUT my_index
{
  "mappings": {
    "_source": {
      "enabled": false
    },
    "properties": {
      "field1": { "type": "text" }
    }
  }
}
```

在这个例子中，新创建的`my_index`索引将不会存储`_source`字段。



2. 获取文档的`_source`字段：

```json
GET /my_index/_doc/1
```

返回的结果中会包含`_source`字段。



3. 在获取文档时只获取`_source`字段中特定的字段：

```json
GET /my_index/_doc/1?_source=field1,field2
```

在这个例子中，返回的`_source`字段只包含`field1`和`field2`。



注意：`_source`字段并不用于搜索，禁用`_source`字段不会影响你的搜索结果。

## 源数据过滤

假设你的应用只需要获取部分字段（如"name"和"price"），而其他字段（如"desc"和"tags"）不经常使用或者数据量较大，导致传输和处理这些额外的数据会增加网络开销和处理时间。在这种情况下，通过设置`includes`和`excludes`可以有效地减少每次请求返回的数据量，提高效率。

例如：

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

**Including**：结果中返回哪些field。

**Excluding**：结果中不要返回哪些field，Excluding优先级比Including更高。

> 需要注意的是，尽管这些设置会影响搜索结果中_source字段的内容，但并不会改变实际存储在Elasticsearch中的数据。也就是说，"desc"和"tags"字段仍然会被索引和存储，只是在获取源数据时不会被返回。

上述这种在mapping中定义的方式不推荐，因为mapping不可变。我们可以在查询过程中指定返回的字段，如下：

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

- **分析**： 当你向Elasticsearch插入一个文档时，会进行"分析"处理，将原始文本数据转换成称为"tokens"或"terms"的小片段。这个过程可能包括如下操作：
  - 切分文本（Tokenization）
  - 将所有字符转换为小写（Lowercasing）
  - 删除常见但无重要含义的单词（Stopwords）
  - 提取词根（Stemming）
- **查询**：当执行全文搜索时，查询字符串也会经过类似的分析过程，然后再与已经分析过的数据进行比对，找出匹配的结果并返回。

Elasticsearch提供了许多种全文搜索的查询类型，例如：

- **Match Query**：最基本的全文搜索查询。
- **Match Phrase Query**：用于查找包含特定短语的文档。
- **Multi-Match Query**：类似Match Query，但可以在多个字段上进行搜索。
- **Query String Query**：提供了丰富的搜索语法，可以执行复杂的、灵活的全文搜索。

### match：匹配包含某个term的子句

`match` 查询是 Elasticsearch 中的一种全文查询方式，它包括标准分析和词项搜索。尽管它可以应用于精确字段，但其主要用途是进行全文搜索。当与全文字段一起使用时，match 查询可以解析查询字符串，并执行短语查询或者构建一个布尔查询，这意味着它会考虑字段中的每个单词。

下面有一个简单的 `match` 查询示例：

```json
GET /_search
{
  "query": {
    "match": {
      "message": "this is a test"
    }
  }
}
```

在这个示例中，Elasticsearch 会在 "message" 字段中搜索包含 "this"、"is"、"a" 和 "test" 的文档。

请注意，`match` 查询不仅仅会匹配完全相同的短语，它还可以处理更复杂的情况，如多个单词（它会匹配任何一个）、误拼、同义词等，这主要取决于你所使用的分析器和搜索设置。

`match` 查询还有一些其他参数，例如：

- **operator**：定义多个搜索词之间的关系，默认为 `or`。如果设为 `and`，则返回的文档必须包含所有搜索词。
- **minimum_should_match**：控制返回的文档应至少匹配的搜索词的数量或比例。
- **fuzziness**：允许模糊匹配，可以找到那些拼写错误或接近的词汇。

### match_all：匹配所有结果的子句

`match_all`是Elasticsearch中的一个查询类型，用于获取索引中的所有文档。

这是一个`match_all`查询的基本示例：

```json
{
  "query": {
    "match_all": {}
  }
}
```

在上述示例中，我们可以看到查询对象中存在一个"match_all"字段，其值是一个空对象。这表示我们希望匹配所有文档。

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

Elasticsearch的 `match_all` 查询是最简单的查询，它不需要任何参数，但如果你想为它添加权重，可以使用 `boost` 参数。例如：

```json
GET /_search
{
    "query": {
        "match_all": { "boost" : 1.2 }
    }
}
```

在上面的查询中，`boost` 参数被设置为1.2，给匹配到的所有文档增加了额外的相关性得分提升。

### multi_match：多字段条件

`multi_match` 可以用来在多个字段上进行全文搜索。它接受一个查询字符串和一组需要在其中执行查询的字段列表。

例如：

```json
{
  "query": {
    "multi_match" : {
      "query":    "这是测试", 
      "fields": [ "field1", "field2" ] 
    }
  }
}
```

在此示例中，查询字符串"这是测试"将在字段"field1"和"field2"中搜索。

`multi_match`查询也支持使用通配符(*)来匹配多个字段：

```json
{
  "query": {
    "multi_match" : {
      "query":    "这是测试", 
      "fields": [ "*_name" ] 
    }
  }
}
```

在这个例子中，会在所有以"_name"结尾的字段中进行搜索。

此外，`multi_match` 查询还支持许多参数，包括：

- **type**：设置查询类型，可选值包括：`best_fields`, `most_fields`, `cross_fields`, `phrase`, `phrase_prefix` 等。

例如，“best_fields” 类型会从指定的字段中挑选分数最高的匹配结果计算最终得分，而“most_fields” 类型则会在每个字段中都寻找匹配项并将其分数累加起来。

- **tie_breaker**：当一个词在多个字段中找到时，用于决定最终得分的参数。
- **minimum_should_match**：用于控制应匹配的最小子句数。
- **operator**：主要有两个操作符 `OR` 和 `AND`，默认为 `OR`。

需要注意的是，当使用 `multi_match` 查询时，如果字段不同，其权重可能也会不同。你可以通过在字段名后面添加尖括号（^）和权重值来调整特定字段的权重。例如，`"fields": [ "name^3", "description" ]`表示在"name"字段中的匹配结果权重是"description"字段的三倍。

### match_phrase：短语查询

`match_phrase` 用于精确匹配包含指定短语的文档。match_phrase 查询需要字段值中的单词顺序与查询字符串中的单词顺序完全一致。

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

```json
GET /_search
{
  "query": {
    "match_phrase": {
        "query": "this is a test",
        "slop": 2
      }
  }
}
```

请注意，`match_phrase` 查询需要整个短语完全匹配，而不仅仅是查询中的所有单词都存在。如果你只是希望所有单词都存在，而不关心它们的顺序或精确出现方式，那么你应该使用 `match` 查询。

## Term Query

精确查询用于查找包含指定精确值的文档，而不是执行全文搜索。

### term：匹配和搜索词项完全相等的结果

`term` 查询主要用于查询某个字段完全匹配给定值的文档。这对精确匹配非常有效，例如数字、布尔值或者字符串。

用法示例：

```JSON
GET /_search
{
    "query": {
        "term" : { "user" : "Kimchy" } 
    }
}
```

在这个例子中，我们正在搜索"user"字段中完全匹配"Kimchy"的文档。

需要注意的是，`term` 查询对于分析过的字段（例如，文本字段）可能不会像你预期的那样工作，因为它会搜索精确的词汇项，而不是单词。如果你想要对文本字段进行全文搜素，应该使用 `match` 查询。

另外一个需要注意的点就是 `term` 查询对大小写敏感，所以 "Kimchy" 和 "kimchy" 是两个不同的词条。

#### term和match_phrase的区别

`term` 查询和 `match_phrase` 查询是 Elasticsearch 提供的两种查询方式，它们都用于查找文档，但主要的区别在于如何解析查询字符串以及匹配的精确度。

- **term**：这个查询做的是精确匹配。当你使用`term`查询时，Elasticsearch会查找完全等于你指定的词汇的文档。例如，如果你搜索`term` "apple"，那么只有包含完全为"apple"的文档会被匹配到，而包含"apples"或"APPLE"的文档则不会被匹配到。因此，`term`查询对大小写敏感，且不会进行任何形式的分析（如停用词移除、词干提取等）。
- **match_phrase**：这个查询是用来匹配一系列词汇或者短语的。`match_phrase`查询会保证你查询的词汇必须以你提供的顺序完全匹配。比如，如果你使用`match_phrase`查询 "quick brown fox"，那么只有包含这个完整短语的文档才会被匹配到，单独包含"quick"、"brown"或者"fox"的文档则不会被匹配到。此外，与`term`查询不同，`match_phrase`查询会进行文本分析，这意味着它会考虑词汇的大小写、复数形式等。

总结来说，`term`查询更适合精确匹配，而`match_phrase`查询更适合短语匹配。但是，`match_phrase`并不能100%保证精确匹配，因为它会处理和考虑文本的各种变体（比如，大小写、单复数形式等）。

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

## Range：范围查找

Range查询允许我们查找某个范围内的值。假设我们有一个商品表，其中有商品价格字段，我们可以用range查询来查找价格在一定范围内的商品。

以下是一个基础的范围查询的例子：

```json
GET /products/_search
{
  "query": {
    "range" : {
        "price" : {
            "gte" : 10,
            "lte" : 20,
            "boost" : 2.0
        }
    }
  }
}
```

在这个例子中，我们正在查询价格大于或等于（gte）10且小于或等于（lte）20的所有商品。"boost"参数表示增加该查询的重要性。

Range查询支持以下参数：

- **gte**：大于或等于。
- **lte**：小于或等于。
- **gt**：大于。
- **lt**：小于。
- **boost**：增加查询的重要性。

此外，对于日期类型的字段，你还可以使用如下方式进行范围查询：

```json
{
  "query": {
    "range" : {
        "timestamp" : {
            "gte" : "now-1d/d",
            "lt" :  "now/d"
        }
    }
  }
}
```

在上述查询中，我们正在查找过去24小时内的数据。"now-1d/d"表示从现在算起的一天前，而"now/d"表示当前时间。

## Filter

过滤器（Filter）是用于筛选数据的一种工具。过滤器和查询（query）相似，但有几个重要的区别：

- **过滤不关心文档的相关度得分（relevance score）**：查询会为每个匹配的文档计算一个相关度得分，以决定返回结果的排序。相比之下，过滤器只关心文档是否匹配 - 没有“部分匹配”，只有“匹配”或“不匹配”。
- **过滤器可以被缓存**：由于过滤器不需要计算得分，因此它们的结果可以被缓存起来用于之后的搜索请求，这可以大大提高性能。

常见的过滤器类型包括：`term`、`terms`、`range`、`bool`、`match_all` 等。例如，范围过滤器 `range` 可以用于查找数字或日期字段在指定范围内的文档；布尔过滤器 `bool` 则允许你组合多个过滤器，并定义它们如何互相交互。

使用过滤器时，通常会把它们放在 `bool` 查询的 `filter` 子句中。例如：

```json
{
  "query": {
    "bool": {
      "filter": [
        { "term":  { "status": "active" }},
        { "range": { "age": { "gte": 30, "lte": 40 }}}
      ]
    }
  }
}
```

这个查询会返回所有“状态为 active 并且年龄在 30 到 40 之间”的文档，而不会考虑它们的相关度得分。

### Filter缓存机制

在 Elasticsearch 中，过滤查询结果的缓存机制是非常重要的一个性能优化手段。由于过滤器（filter）只关心是否匹配，而不关心评分 (_score)，因此它们的结果可以被缓存以提高性能。

每次 filter 查询执行时，Elasticsearch 都会生成一个名为 "bitset" 的数据结构，其中每个文档都对应一个位（0 或 1），表示这个文档是否与 filter 匹配。这个 bitset 就是被存储在缓存中的部分。

如果相同的 filter 查询再次执行，Elasticsearch 可以直接从缓存中获取这个 bitset，而不需要再次遍历所有的文档来找出哪些文档符合这个 filter。这大大提高了查询速度，并减少了 CPU 使用。

这种缓存策略特别适合那些重复查询的场景，例如用户界面的过滤器和类似的功能，因为他们通常会产生很多相同的 filter 查询。

然而，值得注意的是，虽然这种缓存可以显著改善查询性能，但也会占用内存空间。如果你有很多唯一的过滤条件，那么过滤器缓存可能会变得很大，从而导致内存问题。这就需要你对使用的过滤器进行适当的管理和限制。

Filter缓存功能会遵循以下原则：

- **同一Filter的多次应用**：如果在后续查询中有多次使用相同的Filter，则ES会把第一次查询的结果储存在缓存中，后续的查询将直接从缓存中获取结果，而不再做任何磁盘I/O或者其他计算。
- **根据需求清理缓存**：ES会根据内存使用情况自动清理缓存，当然你也可以手动清空缓存。但这并不意味着我们无限制地依赖Filter缓存，大量的缓存可能导致更重的GC压力。
- **不缓存复杂查询**：一些查询条件较复杂的过滤器可能不会被缓存，比如script filter、geo filter等。这是因为这些过滤器本身的构建和维护成本可能就超过了查询的计算成本。

ES的Filter缓存机制可以大大提高查询效率，但如果不慎用，比如缓存过多或者不适合缓存的查询，可能会对性能产生负面影响。因此，在设计和优化ES查询时，应当充分考虑Filter的使用和缓存策略。

## Bool Query

Bool Query（组合查询）可以组合多个查询条件，bool查询也是采用more_matches_is_better的机制，因此满足must和should子句的文档将会合并起来计算分值。

![](https://mmbiz.qpic.cn/mmbiz_png/jC8rtGdWScOyKiabe9ph3M760B405ANeTQKKL6ArXcTAUI1g5KV80ib0zNicy5L2usIJuPtDAGvsVa5ibs5Sch5GZw/640)

`boost`和`minumum_should_match`是参数，其他四个都是查询子句。

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

当 `should` 子句与 `must` 或 `filter` 子句一起使用时，这时候需要注意了。

只要满足了 `must` 或 `filter` 的条件，`should` 子句就不再是必须的。换句话说，如果存在一个或者多个 `must` 或 `filter` 子句，那么 `should` 子句的条件会被视为可选。

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

`minimum_should_match`参数定义了在 `should` 子句中至少需要满足多少条件。

例如，如果你有5个 `should` 子句并且设置了 `"minimum_should_match": 3`，那么任何匹配至少三个 `should` 子句的文档都会被返回。

这个参数可以接收绝对数值（如 `2`）、百分比（如 `30%`）、和组合（如 `3<90%` 表示至少匹配3个或者90%，取其中较大的那个）等不同类型的值。

注意：如果 `bool` 查询中只有 `should` 子句（没有 `must` 或 `filter`），那么默认情况下至少需要匹配一个 `should` 条件，也就是`minimum_should_match`默认值是1，除非 `minimum_should_match` 明确设定为其他值。如果包含 `must` 或 `filter`的情况下`minimum_should_match`默认值 0。

所以我们可以在包含`must` 或 `filter`的情况下，设置`minimum_should_match`值来满足`should`子句中的条件。
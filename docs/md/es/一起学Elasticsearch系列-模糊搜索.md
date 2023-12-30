本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)


[TOC]

在 Elasticsearch 中，模糊搜索是一种近似匹配的搜索方式。它允许找到与搜索词项相似但不完全相等的文档。

## 前缀匹配：prefix

前缀匹配通过指定一个前缀值，搜索并匹配索引中指定字段的文档，找出那些以该前缀开头的结果。

在 Elasticsearch 中，可以使用 `prefix` 查询来执行前缀搜索。其基本语法如下：

```json
{
  "query": {
    "prefix": {
      "field_name": {
        "value": "prefix_value"
      }
    }
  }
}
```

其中，`field_name` 是要进行前缀搜索的字段名，`prefix_value` 是要匹配的前缀值。

**注意**：前缀搜索匹配的是term，而不是field，换句话说前缀搜索匹配的是分析之后的词项，并且不计算相关度评分。

**优点：**

- 快速：前缀搜索使用倒排索引加速匹配过程，具有较高的查询性能。
- 灵活：可以基于不同的字段进行前缀搜索，适用于各种数据模型。

**缺点：**

- 前缀无法通配：前缀搜索只能匹配以指定前缀开始的文档，无法进行通配符匹配。
- 高内存消耗：如果前缀值过长或前缀匹配的文档数量过多，将占用较大的内存资源，并且前缀搜索是没有缓存的。

### index_prefixes

`index_prefixes`参数允许对词条前缀进行索引，以加速前缀搜索。它接受以下可选设置:

- **min_chars**：索引的最小前缀长度（包含），必须大于0，默认值为2。
- **max_chars**：索引的最大前缀长度（包含），必须小于20，默认值为5。

index_prefixe可以理解为在索引上又建了层索引，会为词项再创建倒排索引，会加快前缀搜索的时间，但是会浪费大量空间，本质还是空间换时间。

## 通配符匹配：wildcard

通配符匹配允许使用通配符来匹配文档中的字段值，是一种基于模式匹配的搜索方法，它使用通配符字符来匹配文档中的字段值。

通配符字符包括 `*` 和 `?`，其中 `*` 表示匹配任意数量（包括零个）的字符，而 `?` 则表示匹配一个字符。

在通配符搜索中，可以在搜索词中使用通配符字符，将其替换为要匹配的任意字符或字符序列。通配符搜索可以应用于具有文本类型的字段。

**注意**：通配符搜索和前缀搜索一样，匹配的都是分析之后的词项。

**请求示例：** 以下是一个使用通配符搜索的示例请求：

```
GET /my_index/_search
{
  "query": {
    "wildcard": {
      "title.keyword": {
        "value": "elast*"
      }
    }
  }
}
```

在上述示例中，我们对名为 `my_index` 的索引执行了一个通配符搜索。我们指定了要搜索的字段为 `title.keyword`，并使用 `elast*` 作为通配符搜索词。这将匹配 `title.keyword` 字段中以 `elast` 开头的任意字符序列。

## 正则表达式匹配：regexp

正则表达式匹配（regexp）是一种基于正则表达式模式进行匹配的搜索方法，它允许使用正则表达式来匹配文档中的字段值。

**用途：** 正则表达式匹配在以下情况下非常有用：

- 高级模式匹配：当需要更复杂的模式匹配时，正则表达式匹配提供了更多的灵活性和功能。
- 模糊搜索：通过使用通配符和限定符，可以进行更精确的模糊匹配。

**优缺点：**

- 优点：
  - 强大的模式匹配：正则表达式匹配提供了强大且灵活的模式匹配功能，可以满足各种复杂的搜索需求。
  - 可定制性：通过使用正则表达式，您可以根据具体需求编写自定义的匹配规则。
- 缺点：
  - 性能：正则表达式匹配的性能较低，尤其是在大型索引上进行正则表达式匹配可能会导致搜索延迟和资源消耗增加。
  - 学习成本高：使用正则表达式需要一定的学习和理解，对于不熟悉正则表达式的人来说可能会有一定的难度。

**语法**：

```json
GET <index>/_search
{
  "query": {
    "regexp": {
      "<field>": {
        "value": "<regex>",
        "flags": "ALL",
      }
    }
  }
}
```

**请求示例：** 以下是一个使用正则表达式匹配的示例请求：

```
GET /my_index/_search
{
  "query": {
    "regexp": {
      "title.keyword": {
        "value": "elast.*",
        "flags": "ALL"
      }
    }
  }
}
```

在上述示例中，我们对名为 `my_index` 的索引执行了一个正则表达式匹配。我们指定要搜索的字段为 `title.keyword`，并使用 `elast.*` 作为正则表达式匹配模式。这将匹配 `title.keyword` 字段中以 `elast` 开头的字符序列，并且后面可以是任意字符。

**注意**：regexp查询的性能可以根据提供的正则表达式而有所不同。为了提高性能，应避免使用通配符模式，如 `.`  或 `.?+` 未经前缀或后缀。

### flags

正则表达式匹配的 `flags` 参数用于指定正则表达式的匹配选项。它可以修改正则表达式的行为以进行更灵活和精确的匹配。

**语法：** 在正则表达式匹配的查询中，`flags` 参数是一个字符串，它可以包含多个选项，并用逗号分隔。每个选项都由一个字母表示。

以下是常用的 `flags` 参数选项及其说明：

- `ALL`：启用所有选项，相当于同时启用了 `ANYSTRING`, `COMPLEMENT`, `EMPTY`, `INTERSECTION`, `INTERVAL`, `NONE`, `NOTEMPTY`, 和 `NOTNONE`。
- `ANYSTRING`：允许使用 `.` 来匹配任意字符，默认情况下 `.` 不匹配换行符。
- `COMPLEMENT`：求反操作，匹配除指定模式外的所有内容。
- `EMPTY`：匹配空字符串。
- `INTERSECTION`：允许使用 `&&` 运算符来定义交集。
- `INTERVAL`：允许使用 `{}` 来定义重复数量的区间。
- `NONE`：禁用所有选项，相当于不设置 `flags` 参数。
- `NOTEMPTY`：匹配非空字符串。
- `NOTNONE`：匹配任何内容，包括空字符串。

flags参数用到的场景比较少，做下了解即可。


## 模糊匹配：fuzzy

模糊查询（Fuzzy Query）是 Elasticsearch 中一种近似匹配的搜索方式，用于查找与搜索词项相似但不完全相等的文档。基于编辑距离（Levenshtein 距离）计算两个词项之间的差异。

它通过允许最多的差异量来匹配文档，以处理输入错误、拼写错误或轻微变体的情况。

**用途**：纠正拼写错误，模糊查询可用于纠正用户可能犯的拼写错误，可以提供宽松匹配，使搜索结果更加全面。

- 混淆字符 (**b**ox → fox) 
- 缺少字符 (**b**lack → lack)
- 多出字符 (sic → sic**k**)
- 颠倒次序 (a**c**t → **c**at)

请求示例：

```json
GET /my_index/_search
{
  "query": {
    "fuzzy": {
      "title": {
        "value": "quick",
        "fuzziness": "2"
      }
    }
  }
}
```

`fuzziness`是编辑距离，即：**编辑成正确字符所需要挪动的字符的数量**

### 参数

- **value**：必须，关键词。
- **fuzziness**：编辑距离，范围是（0，1，2），并非越大越好，过大召回率高但结果不准确，默认是：AUTO，即自动从0~2取值。
  - 两段文本之间的Damerau-Levenshtein距离是使一个字符串与另一个字符串匹配所需的插入、删除、替换和调换的数量。
  - 距离公式：Levenshtein是lucene的概念，ES做了改进，使用的是基于Levenshtein的Damerau-Levenshtein，比如：axe=>aex。 Levenshtein会算作2个距离，而Damerau-Levenshtein只会算成1个距离。
- **transpositions**：可选，布尔值，指示编辑是否包括两个相邻字符的变位（ab→ba），默认为true，使用的是Damerau-Levenshtein，如果为false，就会使用Levenshtein去计算。

## 短语前缀：match_phrase_prefix

先来了解下match_phrase，match_phrase检索有如下特点：

- match_phrase会分词。
- 被检索字段必须包含match_phrase中的所有词项并且顺序必须是相同的。
- 默认被检索字段包含的match_phrase中的词项之间不能有其他词项。

`match_phrase_prefix`与`match_phrase`相同,但是它多了一个特性，就是它允许在文本的最后一个词项(term)上的前缀匹配。

如果是一个单词，比如a，它会匹配文档字段所有以a开头的文档，如果是一个短语，比如 "this is ma" ，他会先在倒排索引中做以ma做前缀搜索，然后在匹配到的doc中以 "this is" 做match_phrase查询。

`match_phrase_prefix` 查询是一种结合了短语匹配和前缀匹配的查询方式。它用于在某个字段中匹配包含指定短语前缀的文档。

具体来说，`match_phrase_prefix` 查询会将查询字符串分成两部分：前缀部分和后缀部分。然后它会先对前缀部分进行短语匹配，找到以该短语开头的文档片段；接下来，针对符合前缀匹配的文档片段，再对后缀部分进行前缀匹配，从而进一步筛选出最终匹配的文档。

以下是 `match_phrase_prefix` 查询的示例：

```
GET /my_index/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": {
        "query": "quick brown f",
        "max_expansions": 10
      }
    }
  }
}
```

解释：

- 在上述示例中，我们执行了一个 `match_phrase_prefix` 查询。
- 查询字段为 `title`，我们要求匹配的短语是 "quick brown f"。
- `max_expansions` 参数用于控制扩展的前缀项数量（默认为 50）。这里我们设置为 10，表示最多扩展 10 个前缀项进行匹配。

`match_phrase_prefix` 查询适用于需要同时支持短语匹配和前缀匹配的场景。例如，当用户输入一个搜索短语的前缀时，可以使用该查询来获取相关的文档结果。

### 参数

- **analyzer**：指定何种分析器来对该短语进行分词处理。
- **max_expansions**：限制匹配的最大词项，有点类似SQL中的limit，默认值是50。
- **boost**：用于设置该查询的权重。
- **slop**：允许短语间的词项(term)间隔，slop 参数告诉 match_phrase 查询词条相隔多远时仍然能将文档视为匹配，相隔多远意思就是说为了让查询和文档匹配你需要移动词条多少次，默认是0。

## ngram & edge ngram

ngram 和 edge ngram 是两种用于分析和索引文本的字符级别的分词器。

- **ngram**：ngram 分词器将输入的文本按照指定的长度切割成一系列连续的字符片段。例如，对于字符串 "Hello"，使用 2-gram（双字符）分词器会生成 ["He", "el", "ll", "lo"]。
- **edge ngram**：edge ngram 分词器是 ngram 分词器的一种特殊形式，它只会产生从单词开头开始的 ngram 片段。例如，对于字符串 "Hello"，使用 2-gram（双字符）edge ngram 分词器会生成 ["He", "el"]。 edge ngram作用类似fuzzy，但是性能要比fuzzy好，当然也更占用磁盘空间，原因是因为edge ngram对更细粒度的token创建了索引。

参数：

- **min_gram**：创建索引所拆分字符的最小阈值。
- **max_gram**：创建索引所拆分字符的最大阈值。

以下是一个示例来说明如何在 Elasticsearch 中使用 ngram 和 edge ngram 分词器：

```json
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_ngram_analyzer": {
          "tokenizer": "my_ngram_tokenizer"
        },
        "my_edge_ngram_analyzer": {
          "tokenizer": "my_edge_ngram_tokenizer"
        }
      },
      "tokenizer": {
        "my_ngram_tokenizer": {
          "type": "ngram",
          "min_gram": 2,
          "max_gram": 4
        },
        "my_edge_ngram_tokenizer": {
          "type": "edge_ngram",
          "min_gram": 2,
          "max_gram": 10
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_ngram_analyzer"
      },
      "keyword": {
        "type": "text",
        "analyzer": "my_edge_ngram_analyzer"
      }
    }
  }
}
```

在上述示例中，我们创建了一个名为 `my_index` 的索引，定义了两个不同的分词器和对应的字段映射：

- `my_ngram_analyzer` 使用了 ngram 分词器，适用于处理 `title` 字段。
- `my_edge_ngram_analyzer` 使用了 edge ngram 分词器，适用于处理 `keyword` 字段。

通过在查询时指定相应的分析器，可以使用这些分词器来进行文本搜索、前缀搜索等操作。

注意：ngram 作为 tokenizer 的时候会把空格也包含在内，而作为 token filter 时，空格不会作为处理字符。
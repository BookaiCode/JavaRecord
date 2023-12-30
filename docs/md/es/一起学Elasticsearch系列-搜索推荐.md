本文已收录至Github，推荐阅读 👉 [Java随想录](https://github.com/ZhengShuHai/JavaRecord)

微信公众号：[Java随想录](https://mmbiz.qpic.cn/mmbiz_jpg/jC8rtGdWScMuzzTENRgicfnr91C5Bg9QNgMZrxFGlGXnTlXIGAKfKAibKRGJ2QrWoVBXhxpibTQxptf8MsPTyHvSg/0?wx_fmt=jpeg)


[TOC]

我们在进行搜索的时候，一般都会要求具有“搜索推荐”或者叫“搜索补全”的功能，即在用户输入搜索的过程中，进行自动补全或者纠错，以此来提高搜索文档的匹配精准度，进而提升用户的搜索体验，这就是Suggest。

ES针对不同的应用场景，把Suggester主要分为以下四种：

`Tern Suggester`，`Phrase Suggester`，`Completion Suggester`，`Context Suggester`

## Term Suggester

意如其名，Term Suggester针对单独term的搜索推荐，不考虑搜索短语中多个term的关系。

请求示例模版：

```json
POST <index>/_search
{ 
  "suggest": {
    "<suggest_name>": {
      "text": "<search_content>",
      "term": {
        "suggest_mode": "<suggest_mode>",
        "field": "<field_name>"
      }
    }
  }
}
```

以下是一个具体的示例，演示如何使用 Term Suggester 进行搜索建议：

```json
POST my_index/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "bown",
      "term": {
        "suggest_mode": "popular",
        "field": "title"
      }
    }
  }
}
```

在这个示例中，我们发送了一个建议请求，要求根据用户输入的文本 `"bown"` 提供搜索建议。建议器将在 `title` 字段中查找匹配项，并提供最受欢迎的建议结果。

### Options

- **text**：用户搜索的文本。
- **field**：要从哪个字段选取推荐数据。
- **analyzer**：使用哪种分词器。
- **size**：每个建议返回的最大结果数。
- **sort**：如何按照提示词项排序，参数值只可以是以下两个枚举：
  - score：分数>词频>词项本身。
  - frequency：词频>分数>词项本身。
- **suggest_mode**：搜索推荐的推荐模式，参数值亦是枚举：
  - missing：默认值，当用户输入的文本在索引中找不到匹配项时，仍然提供建议。如果用户输入的文本在索引中没有匹配项，但有与之相关的建议结果，则这些建议结果将被返回作为搜索建议。这种模式适用于确保即使没有完全匹配的结果，用户仍能获得相关的建议。
  - popular：根据最受欢迎或最频繁出现的词项来生成建议结果。对于给定的用户输入，Term Suggester 将返回那些在索引中最常出现的词项作为建议结果。这种模式适用于提供与最流行或最常见搜索关键词相关的建议。
  - always：始终提供建议，即使已经存在完全匹配的结果。无论用户输入的文本是否与索引中的某个词项完全匹配，Term Suggester 都会提供一组建议结果。这种模式适用于用户输入的文本可能只是部分匹配的情况，以便提供更多的补全或纠错建议。
- **max_edits**：可以具有最大偏移距离候选建议以便被认为是建议。只能是1到2之间的值。任何其他值都将导致引发错误的请求错误。默认为2。
- **prefix_length**：前缀匹配的时候，必须满足的最少字符。
- **min_word_length**：最少包含的单词数量，通过设置 `min_word_length` 参数，可以过滤掉那些长度不足的词项，从而得到更具有意义和相关性的建议结果。
- **min_doc_freq**：最少的文档频率，通过设置 `min_doc_freq` 参数，可以过滤掉那些在文档中出现频率较低的词项，从而得到更具有代表性和相关性的建议结果。
- **max_term_freq**：最大的词频，通过设置 `max_term_freq` 参数，可以控制建议结果中词项的重复出现程度，以避免过多重复的词项。

## Phrase Suggester

Phrase Suggester 是 Elasticsearch 中用于短语级别建议的功能。它可以根据用户输入的文本生成相关的短语建议，帮助用户补全或纠正输入。

Term Suggester可以对单个term进行建议或者纠错，但是不会考虑多个term之间的关系，Phrase Suggester在Term Suggester的基础上，会去考虑多个term之间的关系，比如是否同时出现在一个索引原文中，相邻程度以及词频等等。

以下是一个使用 Phrase Suggester 的请求示例模板：

```json
POST <index>/_search
{
  "suggest": {
    "<suggest_name>": {
      "text": "<search_content>",
      "phrase": {
        "field": "<field_name>",
        "gram_size": <gram_size>,
        "direct_generator": [
          {
            "field": "<field_name>",
            "suggest_mode": "<suggest_mode>"
          }
        ]
      }
    }
  }
}
```

以下是一个具体的示例，演示如何使用 Phrase Suggester 进行短语建议：

```json
POST my_index/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "quik brwn",
      "phrase": {
        "field": "title",
        "gram_size": 2,
        "direct_generator": [
          {
            "field": "title",
            "suggest_mode": "popular"
          }
        ]
      }
    }
  }
}
```

在这个示例中，我们发送了一个建议请求，要求根据用户输入的文本 `"quik brwn"` 提供短语建议。Phrase Suggester 将在 `title` 字段中查找与短语相关的建议结果。

生成短语时，使用的 gram 大小为 2，表示使用两个连续的词项进行组合。而直接生成器（direct_generator）将根据最受欢迎或最频繁出现的词项生成建议结果。

### Options

- **real_word_error_likelihood**：默认值为 0.95，即告诉 Elasticsearch 索引中有5% 的术语拼写错误。该参数指定了词语在索引中被认为是拼写错误的概率。较低的值将使得更多在索引中出现的词语被视为拼写错误，即使它们实际上是正确的。
- **max_errors**：最大容忍错误百分比。默认值为 1，表示最多允许 1% 的错误。当建议短语与输入短语匹配时，如果超过该百分比的术语被认为是错误的，则该建议会被排除。
- **confidence**：默认值为 1.0，取值范围为 [0, 1]。该参数控制建议结果的置信度阈值。只有得分高于此阈值的建议才会返回。较高的值意味着只有得分接近或高于输入短语的建议才会显示。
- **collate**：该参数用于修剪建议结果，仅保留那些与给定查询匹配的建议。它接受一个匹配查询作为参数，并且只有当建议的文本与该查询匹配时，才会返回该建议。还可以在查询参数的 "params" 对象中添加更多字段。当参数 "prune" 设置为 true 时，响应中会增加一个 "collate_match" 字段，指示建议结果中是否存在匹配所有更正关键词的匹配项。
- **direct_generator**：该参数控制候选生成器的行为。Phrase Suggester 使用候选生成器生成给定文本中每个项的可能建议项列表。目前，只有一种候选生成器可用，即 direct_generator。它以文本中的每个项单独调用 Term Suggester 来生成候选项，并将生成器的输出与建议结果进行打分。

## Completion Suggester

Completion Suggester 是一种用于实现自动补全功能的建议器。它基于预定义的文本片段，为用户提供与输入文本匹配的建议。

Completion Suggester 支持三种查询：前缀查询（prefix），模糊查询（fuzzy），正则表达式查询（regex)。

Completion Suggester也是最常使用的Suggester。

主要针对的应用场景就是"Auto Completion"。 此场景下用户每输入一个字符的时候，就需要即时发送一次查询请求到后端查找匹配项，在用户输入速度较高的情况下对后端响应速度要求比较苛刻。

因此实现上它和前面两个Suggester采用了不同的数据结构。

**索引并非通过倒排来完成，而是将analyze过的数据编码成FST和索引一起存放，对于一个open状态的索引，FST会被ES整个装载到内存里的，进行前缀查找速度极快。但是FST只能用于前缀查找，这也是Completion Suggester的局限所在**

使用Completion Suggester需要注意以下两点：

- 内存代价太大，性能高是通过大量的内存换来的。
- 只能前缀搜索，假如输入的不是前缀，召回率可能很低。

Completion Suggester 需要对字段进行特定的映射来支持自动补全功能。以下是为使用 Completion Suggester 所需的映射配置：

1. **type**：将字段类型设置为 "completion"。
2. **analyzer**：为字段指定一个适当的分析器。建议使用 "simple" 分析器，因为它会保留完整的输入字符串作为术语的后缀，并用于生成建议。
3. **search_analyzer**：对搜索查询应用的分析器。通常，与索引时使用的相同的分析器一起使用。

以下是一个示例映射配置：

```json
{
  "mappings": {
    "properties": {
      "suggestion_field": {
        "type": "completion",
        "analyzer": "simple",
        "search_analyzer": "simple"
      }
    }
  }
}
```

请注意，Completion Suggester 只能在专门为自动补全而设计的字段上使用。它不适用于常规的文本字段。

以下是一个使用 Completion Suggester 的请求示例模板：

```json
POST <index>/_search
{
  "suggest": {
    "<suggest_name>": {
      "prefix": "<search_content>",
      "completion": {
        "field": "<field_name>"
      }
    }
  }
}
```

以下是一个具体的示例，演示如何使用 Completion Suggester 进行自动完成建议：

```json
POST my_index/_search
{
  "suggest": {
    "my-suggestion": {
      "prefix": "th",
      "completion": {
        "field": "title_suggest"
      }
    }
  }
}
```

在这个示例中，我们发送了一个建议请求，要求根据用户输入的前缀 `"th"` 提供自动完成建议。Completion Suggester 将在 `title_suggest` 字段中查找与前缀匹配的建议结果。

## Context Suggester

Context Suggester允许在生成建议时考虑额外的上下文信息。与 Completion Suggester 不同，Context Suggester 可以根据特定的上下文条件来过滤和排序建议结果。

Context Suggester是建立在Completion Suggester基础之上的，可以看成是Completion Suggester的一种补充。

Context Suggester 支持两种类型的上下文：

- **Category Context**：允许为建议结果定义一个或多个分类标签，并使用这些标签进行过滤。这样可以确保生成的建议结果与特定的类别相关联。例如，如果您正在构建一个电子商务应用程序，可以使用 Category Context 将建议限制为特定的产品类别，如衣物、鞋类等。
- **Geo Location Context**：允许您基于地理位置信息进行建议。您可以提供经纬度坐标，并根据这些坐标过滤建议结果。这对于需要基于用户当前位置生成建议的应用程序非常有用，比如附近的商铺或景点推荐。

Context Suggester 中，有几个重要的参数可以用来指定上下文条件和设置建议行为。下面是一些常用的参数：

- **name**：上下文名称，用于标识特定的上下文条件。
- **type**：上下文类型，可以是 `"category"` 或 `"geo"`，分别表示分类标签上下文和地理位置上下文。
- **path**：对于嵌套对象，用于指定包含上下文条件的字段路径。

**请求示例：**

```json
POST /my-index/_search
{
  "suggest": {
    "my-suggestion": {
      "prefix": "Pro",
      "completion": {
        "field": "suggestions",
        "context": {
          "category": {
            "path": "category.sub_category"
          }
        }
      }
    }
  }
}
```

在上述示例中，我们向索引 `my-index` 发送了一个搜索请求，并使用了 Context Suggester。

- `field` 参数设置为 `"suggestions"`，表示要从该字段中获取建议。
- `context.path` 参数设置为 `"category.sub_category"`，表示要从文档的 `category.sub_category` 字段中提取上下文信息。

这样，Context Suggester 将根据搜索的前缀和上下文信息生成相应的建议结果。

- **context**：上下文值，根据上下文类型和值的数据类型进行指定。可以是文本、数字、布尔值等。
- **boost**：可选参数，用于调整上下文的重要性。默认情况下，所有上下文都具有相同的权重。
- **precision**：仅适用于 Geo Location Context，用于指定经纬度坐标的精度。
- **neighbors**：仅适用于 Geo Location Context，用于指定返回结果时附近的邻居数量。

通过这些参数，可以配置 Context Suggester 来满足特定的需求。例如，可以定义多个不同的上下文条件，并为每个上下文条件指定不同的权重，以影响建议结果的排序顺序。还可以使用 path 参数来处理嵌套对象中的上下文条件。

当使用 Context Suggester 时，可以通过以下请求示例向 Elasticsearch 插入文档：

```
POST /my-index/_doc/1
{
  "title": "Product 1",
  "suggestions": [
    {
      "input": "Product 1",
      "weight": 10,
      "contexts": {
        "category": ["electronics"],
        "location": ["New York"]
      }
    },
    {
      "input": "Phone",
      "weight": 5,
      "contexts": {
        "category": ["electronics", "communication"],
        "location": ["Seattle"]
      }
    }
  ]
}
```

这个请求用于向名为 "my-index" 的索引插入一篇文档。该文档的ID是 "1"，包含了一个 "title" 字段和一个 "suggestions" 字段。

"suggestions" 字段是一个数组，其中包含了两个建议项。每个建议项都有一个 "input" 属性表示建议的文本，一个可选的 "weight" 属性表示权重值，以及一个 "contexts" 对象表示建议的上下文信息。

具体解释如下：

- "title"： "Product 1" 表示这篇文档的标题是 "Product 1"。
- "suggestions"：[...] 是一个包含两个建议项的数组。
- 第一个建议项:
  - "input"："Product 1" 表示第一个建议项的文本是 "Product 1"。
  - "weight"：10 表示给予这个建议项的权重是 10。
  - "contexts"：{...} 表示这个建议项的上下文信息。
    - "category"：["electronics"] 表示这个建议项属于 "electronics" 类别。
    - "location"：["New York"] 表示这个建议项的位置是 "New York"。
- 第二个建议项:
  - "input"："Phone" 表示第二个建议项的文本是 "Phone"。
  - "weight"：5 表示给予这个建议项的权重是 5。
  - "contexts"：{...} 表示这个建议项的上下文信息。
    - "category"：["electronics", "communication"] 表示这个建议项属于 "electronics" 和 "communication" 类别。
    - "location"：["Seattle"] 表示这个建议项的位置是 "Seattle"。

接下来，让我给出一个关于如何发送请求并获取响应的示例：

**请求：**

```json
POST /my-index/_search
{
  "suggest": {
    "my-suggestion": {
      "prefix": "Pro",
      "completion": {
        "field": "suggestions",
        "context": {
          "category": "electronics",
          "location": "New York"
        }
      }
    }
  }
}
```

在上述示例中，我们发送了一个搜索请求，并指定了一个自定义的建议器名称 `"my-suggestion"`。我们设置了前缀为 `"Pro"`，并在 `completion` 参数中指定了要使用的字段名和上下文信息。

**响应：**

```json
{
  "suggest": {
    "my-suggestion": [
      {
        "text": "Pro",
        "offset": 0,
        "length": 3,
        "options": [
          {
            "text": "Product 1",
            "_index": "my-index",
            "_type": "_doc",
            "_id": "1",
            "_score": 10,
            "_source": {
              "title": "Product 1"
            },
            "contexts": {
              "category": ["electronics"],
              "location": ["New York"]
            }
          }
        ]
      }
    ]
  }
}
```

在响应结果中，将看到根据输入前缀 `"Pro"` 检索到的一个建议项。该建议项具有文本、偏移量、长度等属性，并包含相关的元数据，如源文档的信息和上下文信息。
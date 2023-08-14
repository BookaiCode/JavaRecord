## 规范化：normalization

在Elasticsearch中，"normalization" 是指将文本数据转化为一种标准形式的步骤。这种处理主要发生在索引时，包括以下操作：

1. **Lowercasing**：将所有字符转换为小写。这是最常见的标准化形式，因为搜索常常是不区分大小写的。
2. **Removing diacritical marks**：移除重音符号或其他变音记号。例如，将 "résumé" 转换为 "resume"。
3. **Converting characters to their ASCII equivalent**：将非ASCII字符转换为等效的ASCII字符。例如，将 "ë" 转换为 "e"。

这些转换有助于提高搜索的准确性，因为用户可能以各种不同的方式输入同一个词语。通过将索引和搜索查询都转换为相同的形式，可以更好地匹配相关结果。

**说白了normalization就是将不通用的词汇变成通用的词汇。文档规范化，提高召回率**。

举个例子：

假设我们希望在 Elasticsearch 中创建一个新的索引，该索引包含一个自定义分析器，该分析器将文本字段转换为小写并移除变音符号。你可以使用以下的请求：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": { "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "asciifolding"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

在这个例子中，我们首先使用 `PUT` 请求创建了一个名为 "my_index" 的新索引。然后，在 `settings` 对象中定义了一个名为 "my_custom_analyzer" 分析器。

这个分析器包括三部分：

- `"type": "custom"`: 这表示我们正在创建一个自定义分析器。
- `"tokenizer": "standard"`: 这设置了标准分词器，它按空格和标点符号将文本拆分为单词。
- `"filter": ["lowercase", "asciifolding"]`: 这是一个过滤器链，将所有文本转为小写 (lowercasing) 并移除所有的变音符号（如 accented characters）。

最后，在 `mappings` 对象中，我们指定 "my_field" 字段要使用这个自定义分析器。

现在，当我们索引包含像 "Résumé" 这样的文本时，它会被标准化为"resume"，这样无论用户输入 "resume" 还是 "résumé" 或者 "RESUME", 都能匹配到正确的结果。

```JSON
POST /my_index/_doc
{
  "my_field": "Méditerranéen RÉSUMÉ"
}
```

Elasticsearch 在索引这个文档时会将 "Méditerranéen RÉSUMÉ" 转换成 "mediterraneen resume"。这样，无论搜索查询是 "Méditerranéen", "méditerranéen", "MEDITERRANÉEN", "Resume", "résumé" 或 "RESUME"，都能找到这个文档。

## 字符过滤器：character filter

Character filters就是在分词之前过滤掉一些无用的字符， 是 Elasticsearch 中的一种文本处理组件，它可以在分词前先对原始文本进行处理。这包括删除HTML标签、转换符号等。

下面是一些常用的 character filter：

1. **HTML Strip Character Filter**：从输入中去除HTML元素，只保留文本内容。例如，`<p>Hello World</p>` 被处理为 `"Hello World"`。
2. **Mapping Character Filter**：通过一个预定义的映射关系，将指定的字符或字符串替换为其他字符或字符串。例如，你可以定义一个规则将 "&" 替换为 "and"。
3. **Pattern Replace Character Filter**：使用正则表达式匹配和替换字符。

### HTML Strip Character Filter

HTML Strip Character Filter 是 Elasticsearch 中的一个 character filter，其功能是从输入的文本中去除 HTML 元素。这对于处理包含 HTML 标签的文本十分有用。

下面的例子展示了如何创建一个使用 HTML Strip Character Filter 的索引：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_html_analyzer": { 
          "tokenizer": "standard",
          "char_filter": ["html_strip"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "my_field": {
        "type": "text",
        "analyzer": "my_html_analyzer"
      }
    }
  }
  }
```

在这个例子中，我们首先创建了一个新的索引 "my_index"。然后，在分析器配置中，我们创建了一个名为 "my_html_analyzer" 的分析器，并在此分析器中使用了名为 "html_strip" 的内置 character filter。这将会移除 "my_field" 字段中任何的 HTML 标记，只保留纯文本内容。

如果你现在往该索引插入一个包含HTML标签的文档，例如：

```JSON
POST /my_index/_doc
{
  "my_field": "<p>Hello World</p>"
}
```

Elasticsearch 将会把 "<p>Hello World</p>" 看作是 "Hello World" 进行索引。这样无论搜索者输入 "Hello World" 还是 "<p>Hello World</p>"，都可以找到这个文档。

### Mapping Character Filter

在Elasticsearch中，Mapping Character Filter允许通过创建自定义字符映射来预处理文本。这意味着在进行索引或搜索时，可以将特定的字符或字符序列替换为其他字符。

例如，如果你正在处理法语文本并希望统一所有形式的“è”，你可能会创建一个映射，将“è”映射为“e”。或者，如果你正在处理包含特定公司名称的文本，并希望将所有变体都映射到一个常见形式，可以使用此过滤器。

具体的配置如下：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": ["my_char_filter"]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": ["&=> and ", "è => e"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

在这个例子中，我们创建了一个新的char_filter，命名为my_char_filter。然后在分析器my_analyzer中引用了这个字符过滤器。最后，我们定义了两个映射：“&”映射为“and ”，以及“è”映射为“e”。

总的来说，Mapping Character Filter提供了一种灵活的方式，让你能够根据需求修改和控制如何处理文本数据。

当你配置了索引并创建了特定的字符映射规则后，你可以往该索引中插入文档。以下是一个例子：

```JSON
PUT /my_index/_doc/1
{
  "text": "M&M's are delicious!"
}
```

在这个例子中，我们向 `my_index` 索引中的 `text` 字段添加了一条记录："M&M's are delicious!"。因为我们之前在 `my_analyzer` 中定义了一个映射规则，它会自动把 "&" 替换成 "and"。

所以，在Elasticsearch中，无论用户搜索 "M and M's are delicious!" 还是原始的 "M&M's are delicious!"，都能找到这条记录。同时，如果你检索这个文档，例如 `GET /my_index/_doc/1`，返回的结果中 `text` 字段仍为原始输入： "M&M's are delicious!"，因为 character filter 只对搜索和索引过程生效，不会改变实际存储的文档内容。

Mapping Character Filter还有一个使用场景，比如平时我们网上发送弹幕或者游戏中公屏聊天的时候，要是说脏话，脏话内容会被替代为：*。

### Pattern Replace Character Filter

Pattern Replace Character Filter 是 Elasticsearch 中一个强大的工具，它允许使用正则表达式来匹配文本，并将匹配的内容替换为指定的字符串。这对于处理有复杂需求的文本非常有用。

例如，假设你需要在索引或搜索时删除所有的数字，可以使用 Pattern Replace Character Filter，并设置一个匹配所有数字的正则表达式 `[0-9]`，然后将其替换为空字符串或其他所需的字符。

以下是如何配置并使用 Pattern Replace Character Filter 的示例：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": ["my_pattern_replace_char_filter"]
        }
      },
      "char_filter": {
        "my_pattern_replace_char_filter": {
          "type": "pattern_replace",
          "pattern": "[0-9]",
          "replacement": ""
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

在这个例子中，我们定义了一个名为 `my_pattern_replace_char_filter` 的字符过滤器，该过滤器将所有数字（匹配正则表达式 `[0-9]`）替换为一个空字符串（""）。然后，在我们的分析器 `my_analyzer` 中使用了这个字符过滤器。最后，在映射中我们指定了字段 "text" 使用这个分析器。因此，当你向 "text" 字段存储含有数字的文本时，所有的数字会被移除。

例如：

当你配置好索引并设定了特定的字符过滤规则后，你可以向这个索引插入文档。以下是一个插入文档的例子:

```JSON
PUT /my_index/_doc/1
{
  "text": "I have 10 apples."
}
```

在这个例子中，我们向`my_index`索引的`text`字段添加了一条记录："I have 10 apples."。因为我们之前在 `my_analyzer` 中定义了一个正则表达式替换规则，它会自动把数字（"[0-9]"）替换为空字符串。

所以，在Elasticsearch中，无论用户搜索 "I have apples." 还是原始的 "I have 10 apples."，都能找到这条记录。同时，如果你检索这个文档，例如 `GET /my_index/_doc/1`，返回的结果中 `text` 字段仍为原始输入： "I have 10 apples."，因为 character filter 只对搜索和索引过程生效，不会改变实际存储的文档内容。

Pattern Replace Character Filter有一个常用的场景就是手机号脱敏，比如：假设我们希望将电话号码中的前 7 位数字替换为星号 "*" 来进行脱敏处理。可以使用 Pattern Replace Character Filter 进行配置，如下：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "keyword",
          "char_filter": ["my_pattern_replace_char_filter"]
        }
      },
      "char_filter": {
        "my_pattern_replace_char_filter": {
          "type": "pattern_replace",
          "pattern": "(\d{3})(\d{4})(\d{4})",
          "replacement": "$1****$3"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
} 
```

在这个示例中，我们创建的 `my_pattern_replace_char_filter` 将匹配任意连续10位数字的电话号码，并将其中的第 4 至第 7 位替换为四个星号 "*"。

然后插入一条包含手机号的文档：

```JSON
PUT /my_index/_doc/1
{
  "text": "My phone number is 12345678901."
}
```

Elasticsearch 在索引这个文档时，会按照我们设置的规则将手机号码脱敏为 "123***\*8901\*\*"，所以无论用户搜索 "\*\*My phone number is 12345678901.\*\*" 还是 "\*\*My phone number is 123\****8901." 都能找到这条记录。

## 令牌过滤器（token filter）

在 Elasticsearch 中，Token Filter 负责处理 Analyzer 的 Tokenizer 输出的单词或者 tokens。这些处理操作包括：转换为小写、删除停用词、添加同义词等。

### 大小写和停用词

以下是一个例子，我们创建一个自定义分析器来演示如何使用 `lowercase` 和 `stop` token filter：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": ["lowercase", "english_stop"]
        }
      },
      "filter": {
        "english_stop": {
          "type": "stop",
          //这里的 _english_ 是一个预设的停用词列表，
          //它包含了一些常用的英语停用词，如 "and", "is", "the" 等。
          "stopwords": "_english_"
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

在这个例子中，我们创建了一个名为 `my_analyzer` 的自定义分析器，它首先使用 `standard` 分词器将文本分割成 tokens，然后使用 `lowercase` 将所有 tokens 转换为小写形式，并使用 `english_stop` 过滤器移除英文停用词。

现在插入一条记录来测试：

```JSON
PUT /my_index/_doc/1
{
  "text": "The Quick BROWN Fox Jumps Over THE Lazy Dog"
}
```

上述例子中的文本 "The Quick BROWN Fox Jumps Over THE Lazy Dog"，运用我们自定义的 my_analyzer 分析器后，停用词（如 "The", "Over"）将被剔除，并且所有的单词都会被转化为小写。所以这句话在进行索引和搜索时，实际上会被处理成：["quick", "brown", "fox", "jumps", "lazy", "dog"]。

### 同义词

`synonym` token filter 可以帮助我们处理同义词。它可以将某个词或短语映射到其它的同义词。

例如，假设你有一个电子商务网站，并且你想让搜索 "cellphone" 的用户也能看到所有包含 "mobile", "smartphone" 的商品。你可以使用 `synonym` token filter 来实现这一目标。

以下是一个使用 `synonym` token filter 的例子：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          // "synonyms": ["赵,钱,孙,李=>吴", "周=>王"]
          //也可以像上面这种写法
          //当字段中出现"赵"、"钱"、"孙"或"李"时，会被替换成"吴"进行索引；
          //当字段中出现"周"时，会被替换成"王"进行索引。
          "synonyms": [
            "cellphone, mobile, smartphone"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }, 
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "my_synonyms"
      }
    }
  }
}
```

在这个设置中，我们创建了一个名为 `my_synonym_filter` 的同义词过滤器，并定义了 "cellphone", "mobile", "smartphone" 是互为同义词。然后我们在 `my_synonyms` 分析器中使用了该过滤器。

所以现在，无论你是输入 "cellphone", "mobile", 还是 "smartphone" 搜索，Elasticsearch 都会将其视为相同的查询。

我们可以使用`synonyms_path` 指定同义词规则路径，这个文件中列出了所有你定义的同义词，每行都是一组同义词，各词之间用逗号分隔。

使用 `synonyms_path` 参数的主要优点是，你可以在不重启 Elasticsearch 或重新索引数据的情况下，通过更新这个文件来动态地改变同义词规则。

假设你有一个名为 `synonyms.txt` 的文件，内容如下：

```JSON
cellphone, mobile, smartphone tv, telly, television 
```

然后你可以这样配置你的 index：

```JSON
PUT /my_index
{
  "settings": {
    "index" : {
      "analysis" : {
        "analyzer" : {
          "my_analyzer" : {
            "tokenizer" : "standard",
            "filter" : ["lowercase", "my_synonym_filter"]
          }
        },
        "filter" : {
          "my_synonym_filter" : {
            "type" : "synonym",
            "synonyms_path" : "analysis/synonyms.txt"
          }
        }
      }
    } 
    }, 
    "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}
```

在这个设置中，我们创建了一个自定义分析器 `my_analyzer` ，并使用了一个自定义的同义词过滤器 `my_synonym_filter`。过滤器中的 `synonyms_path` 参数指向了存放同义词的 `synonyms.txt` 文件。

注意：`synonyms_path` 是相对于 `config` 目录的路径。例如，如果你的 `config` 目录在 `/etc/elasticsearch/`，那么 `synonyms.txt` 文件应该放在 `/etc/elasticsearch/analysis/synonyms.txt`。

## 分词器（tokenizer）

在 Elasticsearch 中，分词器是用于将文本字段分解成独立的关键词（或称为 token）的组件。这是全文搜索中的一个重要过程。Elasticsearch 提供了多种内建的 tokenizer。

以下是一些常用的 tokenizer：

1. **Standard Tokenizer**：它根据空白字符和大部分标点符号将文本划分为单词。这是默认的 tokenizer。
2. **Whitespace Tokenizer**：仅根据空白字符（包括空格，tab，换行等）进行切分。
3. **Language Tokenizers**：基于特定语言的规则来进行分词，如 `english`、`french` 等。
4. **Keyword Tokenizer**：它接收任何文本并作为一个整体输出，没有进行任何分词。
5. **Pattern Tokenizer**：使用正则表达式来进行分词，可以自定义规则。

你可以根据不同的数据和查询需求，选择适当的 tokenizer。另外，也可以通过定义 custom analyzer 来混合使用 tokenizer 和 filter（比如 lowercase filter，stop words filter 等）以达到更复杂的分词需求。

### 自定义分词器：custom analyzer

在 Elasticsearch 中，你可以创建自定义分词器（Custom Analyzer）。一个自定义分词器由一个 tokenizer 和零个或多个 token filters 组成。tokenizer 负责将输入文本划分为一系列 token，然后 token filters 对这些 token 进行处理，比如转换成小写、删除停用词等。

以下是一个自定义分析器的例子：

```JSON
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_stopwords": {
          "type": "stop",
          "stopwords": ["the", "and"]
        }
      },
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_stopwords"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}
```

在这个配置中，我们首先定义了一个名为 `my_stopwords` 的停用词过滤器，包含两个停用词 "the" 和 "and"。然后我们创建了一个名为 `my_custom_analyzer` 的自定义分析器，其中使用了 `standard` tokenizer 以及 `lowercase` filter 和 `my_stopwords` filter。

因此，在为字段 `text` 索引文本时，Elasticsearch 会首先使用 `standard` tokenizer 将文本切分为 tokens，然后将这些 tokens 转换为小写，并移除其中的 "the" 和 "and"。对于搜索查询也同样适用此规则。

## 中文分词器：ik分词

elasticsearch 默认的内置分词器对中文的分词效果可能并不理想，因为它们主要是针对英文等拉丁语系的文本设计的。如果要在中文文本上获得更好的分词效果，我们可以考虑使用中文专用的分词器。

IK 分词器是一个开源的中文分词器插件，特别为 Elasticsearch 设计和优化。它在中文文本的分词处理上表现出色，能够根据中文语言习惯进行精细的分词。

### 安装和部署

- ik下载地址：https://github.com/medcl/elasticsearch-analysis-ik
- 创建插件文件夹 cd your-es-root/plugins/ && mkdir ik
- 将插件解压缩到文件夹 your-es-root/plugins/ik
- 重新启动es

### ik文件描述

- IKAnalyzer.cfg.xml：IK分词配置文件。
- main.dic：主词库。
- stopword.dic：英文停用词，不会建立在倒排索引中。
- quantifier.dic：特殊词库：计量单位等。
- suffix.dic：特殊词库：行政单位。
- surname.dic：特殊词库：百家姓。
- preposition：特殊词库：语气词。

### ik提供的两种analyzer

1. ik_max_word会将文本做最细粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,中华人民,中华,华人,人民共和国,人民,人,民,共和国,共和,和,国国,国歌”，会穷尽各种可能的组合，适合 Term Query。
2. ik_smart: 会做最粗粒度的拆分，比如会将“中华人民共和国国歌”拆分为“中华人民共和国,国歌”，适合 Phrase 查询。

### ik自定义词库

要使用 IK 分词器的自定义词库，需要对 IK 插件的配置文件进行修改。步骤如下：

1. 找到你 Elasticsearch 安装目录下的 `plugins` 文件夹，然后打开 `ik` 目录。
2. 在 `ik` 目录中，你会找到名为 `config` 的文件夹，这就是 IK 配置的位置。
3. 在 `config` 文件夹中新建一个文本文件，比如叫做 `my_dict.dic`，然后在这个文件中加入你自己的词汇，每行一个词。
4. 接着打开 `IKAnalyzer.cfg.xml` 配置文件，在 `<properties>` 标签内添加一行 `<property name="ext_dict">my_dict.dic</property>`，告诉 IK 分词器你的自定义词库在哪里。
5. 保存修改并重启 Elasticsearch，这时就可以使用自定义的词库了。

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "[http://java.sun.com/dtd/properties.dtd](http://java.sun.com/dtd/properties.dtd)">
<properties>
  <comment>IK Analyzer 扩展配置</comment>
  <property name="ext_dict">my_dict.dic</property>
  <!--用户可以在这里配置自己的扩展字典 -->
  <property name="ext_stopwords"></property>
  <!--用户可以在这里配置自己的扩展停止词字典-->
</properties>
```

上述配置告诉 IK 分词器使用 `my_dict.dic` 作为扩展字典，但没有设置扩展的停用词字典。

注意这种方式只支持静态词库，一旦词库文件更改，则需要重启 Elasticsearch 才能加载新的词条。

### 热更新

要修改词库，必须重启ES才能生效，有时我们会频繁更新词库，比较麻烦，更致命的是，es肯定是分布式的，可能有数百个节点，我们不能每次都一个一个节点上面去修改。基于这种场景，我们可以使用热更新功能。

实现热更新有2种办法：基于远程词库和基于数据库。

#### 基于远程词库

IK 分词器支持从远程 URL 下载扩展字典，这就可以用来实现词库的热更新。

在 `IKAnalyzer.cfg.xml` 配置文件中，你可以设置 `ext_dict` 和 `ext_stopwords` 属性为一个指向你的在线词库文件的 URL：

```XML
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "[http://java.sun.com/dtd/properties.dtd](http://java.sun.com/dtd/properties.dtd)">
<properties>
  <comment>IK Analyzer 扩展配置</comment>
  <property name="ext_dict">[http://myserver.com/my_dict.dic](http://myserver.com/my_dict.dic)</property>
  <!--用户可以在这里配置远程扩展字典 -->
  <property name="ext_stopwords">[http://myserver.com/my_stopwords.dic](http://myserver.com/my_stopwords.dic)</property>
  <!--用户可以在这里配置远程扩展停止词字典-->
</properties>
```

此设置告诉 IK 分词器从指定的 URL 下载词库。它会周期性地（默认每 60 秒）检查这些 URL，如果发现有更新，就重新下载并加载新的词库。

根据官方文档，该请求需要满足下列2点：

1. 该 http 请求需要返回两个头部(header)，一个是 `Last-Modified`，一个是 `ETag`，这两者都是字符串类型，只要有一个发生变化，该插件就会去抓取新的分词进而更新词库。
2. 该 http 请求返回的内容格式是一行一个分词，换行符用 `\n` 即可。

满足上面两点要求就可以实现热更新分词了，不需要重启 ES 实例。

可以将需要自动更新的热词放在一个 UTF-8 编码的 .txt 文件里，放在 nginx 或其他简易 http server 下，当 .txt 文件修改时，http server 会在客户端请求该文件时自动返回相应的 Last-Modified 和 ETag。可以另外做一个工具来从业务系统提取相关词汇，并更新这个 .txt 文件。

基于远程词库这种方式比较简单上手，但是也存在一些缺点：

缺点：

1. 词库的管理不方便，要操作直接操作磁盘文件，检索页很麻烦。
2. 文件的读写没有专门的优化性能不好。
3. 多一层接口调用和网络传输。

#### 基于数据库

另外一种方式是基于数据库，这种方式使用比较多，需要修改ik插件源码。

基本思路是将词库维护在数据库（MySQL，Oracle等）,修改ik源码去数据库加载词库，然后将源码重新打包引入到我们的elasticsearch中。

大概操作步骤如下：

1. **获取 IK 项目源码**：首先从 GitHub 或其他地方获取 IK 分词器插件的源码。
2. **设置数据库连接**：在代码中设置好你的数据库连接参数，如数据库地址、用户名、密码等。
3. **编写读取数据库词库的函数**：编写一个可以从数据库读取词库数据并转换为 IK 分词器可以使用的格式（比如 ArrayList<String>）的函数。
4. **修改字典加载部分的代码**：找到 IK 源码中负责加载扩展字典的部分，原本这部分代码是将文件内容加载到内存中，现在改为调用你刚才编写的函数，从数据库中加载词库数据。
5. **添加定时任务**：添加一个定时任务，每隔一段时间重新执行一次上述加载操作，以实现词库的热更新。
6. **编译和安装**：完成上述修改后，按照 IK 插件的构建说明，使用 Maven 或其他工具将其编译成插件，然后安装到 Elasticsearch 中。
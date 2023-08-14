[TOC]

Elasticsearch的 Scripting 是一种允许你使用脚本来评估自定义表达式的功能。通过它，你可以实现更复杂的查询、数据处理以及柔性调整索引结构等。

Elasticsearch支持多种脚本语言。在 ES 中，脚本语言主要是 Painless，这是 Elasticsearch 自家开发的一种安全、高效并且易于学习的语言。除了 Painless，Elasticsearch 也支持其他几种脚本语言，如 Lucene 的表达式语言，但 Painless 是推荐和默认的选项。

以下是一些常见的使用脚本的场景：

1. **计算字段(Field Calculations)**：你可以使用脚本在查询时动态地改变或添加字段的值。
2. **脚本查询(Script Queries)**：在查询中使用脚本进行复杂的条件判断。
3. **脚本聚合(Script Aggregations)**：使用脚本进行更复杂的聚合计算。

使用脚本时需要注意的是，由于涉及到运行时的计算，过度或者不恰当的使用脚本可能会对性能造成影响。另外，由于脚本具有执行任意代码的能力，因此需要确保脚本的使用在一个安全的环境中，并且只运行信任的脚本。

## 概念

Scripting是Elasticsearch支持的一种专门用于复杂场景下支持自定义编程的强大的脚本功能，ES支持多种脚本语言，如painless，其语法类似于Java，也有注释、关键字、类型、变量、函数等，其就要相对于其他脚本高出几倍的性能，并且安全可靠，可以用于内联和存储脚本。

### 支持的语言

- **groovy**：ES 1.4.x-5.0的默认脚本语言。
- **painless**：JavaEE使用java语言开发，.Net使用C#/F#语言开发，Flutter使用Dart语言开发，同样，ES 5.0+版本后的Scripting使用的语言默认就是painless，painless是一种专门用于Elasticsearch的简单语言，用于内联和存储脚本，是ES 5.0+的默认脚本语言。
- expression：每个文档的开销较低，表达式的作用更多，可以非常快速地执行，甚至比编写native脚本还要快，支持javascript语法的子集。缺点：只能访问数字，布尔值，日期和geo_point字段，存储的字段不可用。
- mustache：提供模板参数化查询。

### Painless特点

优点：

- 语法简单，学习成本低
- 灵活度高，可编程能力强
- 性能相较于其他脚本语言更高，安全性好

缺点：

- 独立语言，虽然易学但仍需单独学习
- 相较于DSL性能低
- 不适用于复杂的业务场景

### 简单例子

以下是一个在 Elasticsearch 查询中使用脚本的简单例子。这个例子将会搜索名为 "my_index" 的索引，寻找字段 "price" 和 "tax" 之和大于 100 的文档。

```Java
GET /my_index/_search
{
  "query": {
    "script": {
      "script": {
        "source": "doc['price'].value + doc['tax'].value > params.value",
        "params": {
          "value": 100
        }
      }
    }
  }
}
```

在上面的查询中：

- 使用了 `script` 查询，它允许在执行查询时运行 Painless 脚本。
- `doc['price'].value + doc['tax'].value > params.value` 是脚本的主体部分，它计算了每个文档的 "price" 和 "tax" 字段的值之和，然后检查这个和是否大于参数 "value"。
- `"params": { "value": 100 }` 定义了传递给脚本的参数，在这个脚本中，参数 "value" 被设置为 100。

这个查询将返回所有 "price" 和 "tax" 之和大于 100 的文档。

## Scripting的CRUD

以下是一些使用 Painless 脚本进行的 Elasticsearch CRUD 操作实例：

### insert（新增）

```Java
POST product/_update/6 {
  "script": {
    "lang": "painless",
    "source": "ctx._source.tags.add('无线充电')"
  }
}
```

这个 Elasticsearch 请求是在尝试更新 "product" 索引中 ID 为 6 的文档，具体来说，它要将新的标签 '无线充电' 添加到 "tags" 字段。

- `POST product/_update/6` 是 HTTP 请求的一部分，告诉 Elasticsearch 要在 "product" 索引中更新 ID 为 6 的文档。
- `"script"` 部分定义了一个 Painless 脚本。脚本内容（`"source": "ctx._source.tags.add('无线充电')"`）表示对当前文档的 "tags" 字段添加一个新的元素 '无线充电'。

因此，整个请求的意思是，在 "product" 索引中，找到 ID 为 6 的文档，并在其 "tags" 字段中添加一个新的元素 '无线充电'。

### update（更新）

```Java
POST product/_update/2 {
  "script": "ctx._source.price-=1"
}
```

这个 Elasticsearch 请求表示在 "product" 索引中对 ID 为 2 的文档进行更新操作，具体来说，是将其 "price" 字段的值减少 1。

- `POST product/_update/2` 是 HTTP 请求的一部分，它告诉 Elasticsearch 在 "product" 索引中更新 ID 为 2 的文档。
- `"script": "ctx._source.price-=1"` 是请求体，其中的脚本用于执行实际的更新操作。在这个例子中，脚本将当前文档（由 `_source` 指定）的 "price" 字段减去 1。

### delete（删除）

```Java
POST product/_update/10 {
  "script": {
    "lang": "painless",
    "source": "ctx.op='delete'"
  }
}
```

这个 Elasticsearch 请求是用来删除指定的文档。

- `POST product/_update/10` 是 HTTP 请求的一部分，它告诉 Elasticsearch 要在 "product" 索引中更新 ID 为 10 的文档。
- `"script"` 部分中的 `"lang": "painless"` 指定了脚本的语言为 Painless。
- `"source": "ctx.op='delete'"` 是脚本的主体内容。这里，`ctx.op` 是一个特殊变量，表示待执行的操作。当它被设置为 'delete' 时，指示 Elasticsearch 删除当前操作中的文档。

### upsert（更新插入）

```Java
POST product/_update/15 {
  "script": {
    "lang": "painless",
    "source": "ctx._source.price += 100"
  },
  "upsert": {
    "name": "小米手机10",
    "desc": "充电贼快掉电更快",
    "price": 1999
  }
}
```

这个 Elasticsearch 请求的含义是在 "product" 索引中更新 ID 为 15 的文档，如果这个文档不存在，则插入一个新的文档。

- `POST product/_update/15` 是 HTTP 请求的主要部分，指明了要在 "product" 索引中更新 ID 为 15 的文档。
- 在 `"script"` 部分，声明了用 Painless 语言编写的脚本（由 `"lang": "painless"` 指定）。脚本内容（由 `"source": "ctx._source.price += 100"` 定义）表示增加当前文档的 "price" 字段值 100。
- `"upsert"` 部分定义了当 ID 为 15 的文档不存在时需要插入的新文档的内容。

整个请求的意思是在 "product" 索引中查找 ID 为 15 的文档并使其 "price" 字段增加 100。如果该文档不存在，则会插入一个新的文档，其 "name"、"desc" 和 "price" 字段的值分别为 "小米手机10"、"充电贼快掉电更快" 和 1999。

### search（查询）

```Java
GET product/_search {
  "script_fields": {
    "my_price": {
      "script": {
        "lang": "expression",
        "source": "doc['price'].value* 0.9"
      }
    }
  }
}
```

这个 Elasticsearch 请求是在尝试搜索 "product" 索引中的文档，并且它使用脚本字段 ("script_fields") 来返回计算结果而不是原始数据。

- `GET product/_search` 是 HTTP 请求的一部分，告诉 Elasticsearch 在 "product" 索引中进行搜索。
- `"script_fields"` 部分定义了一个名为 "my_price" 的脚本字段。这个脚本字段的内容由一个表达式语言脚本 (`"lang": "expression"`) 中的 `"source": "doc['price'].value * 0.9"` 进行计算。这个表达式会将每个文档的 "price" 字段乘以 0.9。

整个请求的意思是，在 "product" 索引中搜索全部文档，并计算每个文档的 "price" 字段值的 90%，然后将结果作为 "my_price" 字段返回。

## 参数化脚本

Elasticsearch 会把编译过的脚本储存在缓存中，以提高重复执行同一脚本的性能。当你再次运行相同的脚本时，Elasticsearch 可以直接从缓存中获取已编译的脚本，而不需要再次编译。但是频繁编译脚本会到来性能问题。可以使用参数化脚本动态传参，解决脚本编译的性能问题。

**参数化脚本在 Elasticsearch 中，是指在编写脚本时使用占位符，并在执行脚本时为这些占位符提供实际值。参数化脚本可以增加脚本的灵活性，并能防止脚本注入攻击**。

在脚本中，你可以通过 `params` 对象访问到传递的参数。

例子一：

```Java
POST /product/_update/1
{
  "script": {
    "lang": "painless",
    "source": """
      if (ctx._source.tags == null) {
        ctx._source.tags = new ArrayList(params.tag_list)
      } else {
        for (tag in params.tag_list) {
          if (!ctx._source.tags.contains(tag)) {
            ctx._source.tags.add(tag);
          }
        }
      }
    """,
    "params": {
      "tag_list": ["无线充电", "大屏幕", "防水"]
    }
  }
}
```

在这个请求中，我们想要更新 "product" 索引中 ID 为 1 的文档，并添加一些新的标签。我们使用了一个 Painless 脚本，该脚本检查文档是否已有 "tags" 字段，如果没有，则创建一个包含参数列表中所有标签的新列表。如果已有 "tags" 字段，则只添加不在现有列表中的新标签。

这里的 `params.tag_list` 就是一个参数占位符。在执行脚本时，由 `"params": {"tag_list": ["无线充电", "大屏幕", "防水"]}` 提供其值。这样的话，你就可以通过改变 `tag_list` 参数来修改你想要添加的标签，而无需每次都修改脚本本身。

例子2：

```Java
GET product / _search {
  "script_fields": {
    "price": {
      "script": {
        "lang": "painless",
        "source": "doc['price'].value"
      }
    },
    "discount_price": {
      "script": {
        "lang": "painless",
        "source": "[doc['price'].value* params.discount_8,doc['price'].value* params.discount_7,doc['price'].value* params.discount_6,doc['price'].value* params.discount_5]",
        "params": {
          "discount_8": 0.8,
          "discount_7": 0.7,
          "discount_6": 0.6,
          "discount_5": 0.5
        }
      }
    }
  }
}
```

- `GET product/_search` 是 HTTP 请求的一部分，告诉 Elasticsearch 在 "product" 索引中进行搜索。

- ```
  "script_fields"
  ```

   部分定义了两个脚本字段："price" 和 "discount_price"。

  - "price" 脚本字段返回每个文档的原始 "price" 字段值；
  - "discount_price" 脚本字段返回一个由四个元素组成的数组。数组中的每个元素都是 "price" 字段值与不同折扣率的乘积。这个脚本字段使用了参数化脚本，参数包括四个不同的折扣率："discount_8", "discount_7", "discount_6" 和 "discount_5"。

因此，整个请求的意思是，在 "product" 索引中搜索所有的文档，并为每个文档计算原始价格和不同折扣率下的价格，然后将这些计算结果作为 "price" 和 "discount_price" 字段返回。

## 脚本模版

在 Elasticsearch 中，脚本模板就是将脚本的源代码作为字符串存储，在运行时使用参数替换占位符以创建实际的脚本。脚本模板使得你可以重用相同的脚本逻辑，并通过提供不同的参数值来改变其行为。

这种方式与参数化脚本略有不同，参数化脚本只在已经定义的脚本中替换参数。而脚本模板则更加灵活，可以在整个脚本中替换参数，甚至可以改变脚本的结构。

脚本模板的一个主要应用场景是搜索请求。你可能希望根据用户的输入来调整查询的某部分，但又不希望每次都重写整个查询。在这种情况下，你可以创建一个脚本模板，并在其中使用占位符来代表可变的部分。然后，你只需要提供必要的参数就可以执行查询，而无需每次都手动修改查询的源码。

这种做法可以简化代码，增强代码的可读性和可维护性，并且降低了因为拼接字符串导致的错误风险。

下面是一个例子：

```Java
POST _scripts / calculate_discount {
  "script": {
    "lang": "painless",
    "source": "doc.price.value * params.discount"
  }
}
```

这个 Elasticsearch 请求创建了一个名为 `calculate_discount` 的脚本模板。该模板包含一个简单的脚本，用于计算一个文档字段（假设为 "price"）的折扣价。"price" 字段值与参数 `params.discount` 相乘，得到折扣后的价格。

这个模板可以在许多不同的地方使用，例如在搜索请求中作为脚本字段或者在更新请求中。只需要提供不同的 `discount` 参数就可以得到不同的折扣价，而无需每次都修改整个脚本源码。

以下是如何在搜索请求中使用这个模板的示例：

```Java
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "discounted_price": {
      "script": {
        "id": "calculate_discount",
        "params": {
          "discount": 0.9
        }
      }
    }
  }
}
```

在这个搜索请求中，我们使用了 `calculate_discount` 模板，并提供了一个折扣参数 `0.9`。这个请求会返回所有 "products" 索引中的文档，并且每个文档都会包含一个新的字段 "discounted_price"，它的值是原始 "price" 字段值的 90%。

## 函数式编程

Elasticsearch 的脚本语言 Painless 支持函数式编程。函数式编程是一种编程范式，它让你能够编写出更加简洁清晰的代码。函数可以作为参数传递给其他函数，也可以从其他函数中返回。

> Painless 是 Elasticsearch 的默认脚本语言，它的语法是基于 Java 语言的，但并不是完全等同于 Java。它旨在提供 Java 类似的语法，同时增加安全性和性能。这意味着如果你熟悉 Java，那么你很快就能上手 Painless。总的来说，虽然 Painless 的语法大部分与 Java 相似，但它们还是有一些重要的区别。具体可以查阅 Elasticsearch 官方文档。

以下是一个使用 Painless 进行函数式编程的示例：

```Java
POST _scripts/calculate_total
{
  "script": {
    "lang": "painless",
    "source": """
      double calculateTotal(List items, Map prices) {
        return items.stream().mapToDouble(item -> prices.get(item)).sum();
      }
      calculateTotal(params.items, params.prices)
    """,
    "params": {
      "items": ["item1", "item2"],
      "prices": {"item1": 10.0, "item2": 20.0}
    }
  }
}
```

在这个示例中，我们首先定义了一个函数 `calculateTotal`，它接受一个商品列表和一个价格映射，然后计算并返回所有商品的总价。注意这里使用了 Java 8 的 Stream API 和 Lambda 表达式来进行函数式编程。然后我们调用 `calculateTotal` 函数，并提供参数 `params.items` 和 `params.prices`。

此外，Painless 还支持许多其他函数式编程特性，如高阶函数、纯函数、不可变数据等。所有这些特性都使得你可以编写出更加简洁、有表现力的脚本。

## 正则表达式

早先某些版本正则表达式默认情况下处于禁用模式，因为它绕过了painless的针对长时间运行和占用内存脚本的保护机制。

如果你需要启用这个功能，可以在 Elasticsearch 的配置文件(`elasticsearch.yml`)里加入以下行：

```
script.painless.regex.enabled: true
```

然后重启 Elasticsearch 使得更改生效。但请注意，因为正则表达式操作可能会导致长时间运行和大量占用内存，所以只有在完全了解风险并且确实需要使用正则表达式的情况下，才应该启用这个功能。

以下是一个使用正则表达式的例子，它查询所有字段 "message" 包含至少一个数字的文档：

```Java
GET /_search
{
  "query": {
    "bool": {
      "filter": {
        "script": {
          "script": {
            "source": "doc['message'].value =~ /\d+/",
            "lang": "painless"
          }
        }
      }
    }
  }
}
```

在这个请求中，我们使用了 Painless 中的正则表达式操作符 `=~` 来判断 "message" 字段的值是否匹配正则表达式 `/\d+/`，该正则表达式表示一或多个数字。

注意正则表达式需要两个反斜杠进行转义，因为 JSON 语法本身也需要对反斜杠进行转义。如果没有 JSON 语法的转义需求，在 Painless 中写正则表达式时只需要一个反斜杠即可。

## 聚合中使用script

```Java
GET product / _search {
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "price": {
            "lte": 1000
          }
        }
      }
    }
  },
  "aggs": {
    "tag_agg": {
      "sum": {
        "script": {
          "lang": "painless",
          "source": """
          int total = 0; 
            for(int i = 0; i <doc['tags.keyword'].length; i++){ 
            total++; 
          } 
            return total;
          """
        }
      }
    }
  }
}
```

这个Elasticsearch查询的目标是：

1. 通过`constant_score`查询，找到价格（'price'字段）小于或等于1000的所有产品。
2. 对查询结果进行聚合，用名为"tag_agg"的求和操作，计算每个产品的'tags.keyword'字段的长度（即，每个产品有多少个标签）。这个聚合操作使用了Painless脚本语言。

这样执行之后，你将得到价格小于或等于1000的所有产品，以及每个产品的标签数量。

## doc 和params

### doc和params的用法

使用 `doc['field'].value` 访问简单字段值：

假设你有一个字段叫做 "age" ，你想通过脚本检查年龄是否大于30。在Painless脚本中，你可以像这样编写：

```Java
"script": {
  "lang": "painless",
  "source": "return doc['age'].value > 30;"
}
```

使用 `params['_source']['field']` 访问复杂字段值：

假设你有一个名为 "person" 的复杂字段，它内部含有 "name" 子字段。如果你想获取该名字的长度，可以这样编写脚本：

```Java
"script": {
  "lang": "painless",
  "source": "return params['_source']['person']['name'].length();"
}
```

请注意，以上示例都基于特定的数据模型和需求，实际使用可能需要根据你的实际数据结构进行修改。

### doc 和params 的区别

假设我们有如下的Elasticsearch文档结构：

```Java
{
  "product": {
    "name": "An Amazing Product",
    "details": {
      "manufacturer": "Great Company",
      "price": 100
    }
  }
}
```

这是一个嵌套对象，其中 "product" 是一个对象，而 "details" 是 "product" 的子对象。

如果你想在脚本中获取 "price" 值，你可能会尝试下面的代码：

```Java
"script": {
  "lang": "painless",
  "source": "return doc['product.details.price'].value;"
}
```

然而，这将不会成功，因为 `doc['field'].value` 无法直接访问复杂类型（如 object 或 nested）字段。在这种情况下，你需要使用 `params['_source']['field']`：

```Java
"script": {
   "lang": "painless",
   "source": "return params['_source']['product']['details']['price'];"
}
```

1. `doc['field'].value` 是从Lucene索引中读取字段值，这种方式速度快，效率高。然而，它把数据加载到内存中，可能会增加内存使用。此外，它只能用于简单类型字段，无法处理复杂类型（如object或nested）。
2. `params['_source']['field']` 是从原始的 `_source` 字段获取数据。这种方式可以访问所有类型的字段，包括复杂类型。但是，这要求加载和解析整个原始JSON文档，因此执行效率较低。

所以，如果你的字段是简单类型，并且你关心查询的性能，那么优先使用 `doc['field'].value`。如果你需要处理复杂类型字段或者未索引的字段，那么可以使用 `params['_source']['field']`。
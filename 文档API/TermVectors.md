## Term Vectors

返回文档特定字段的terms信息和统计信息。文档可以存储在索引中或由用户人为提供。term vector是默认[实时](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime)的，而不是接近实时的。这可以通过设置`realtime`参数为`false`来更改状态。

```
curl -XGET 'localhost:9200/twitter/tweet/1/_termvectors?pretty'
```

或者，您可以使用url中的参数指定要检索信息的字段

```
curl -XGET 'localhost:9200/twitter/tweet/1/_termvectors?fields=message&pretty'
```

或者通过在请求正文中添加请求的字段（请参见下面的示例）。字段也可以使用通配符以与[多重匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)相似的方式指定

![警告](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/warning.png)

请注意，`/_termvector`在2.0中不推荐使用，并替换为`/_termvectors`。

### 返回值

可以要求三种类型的价值：*term information，*term statistics* 和*field statistics*。默认情况下，所有字段的所有术语信息和字段统计信息都会返回，但是没有任何term statistics。

#### Term information

- 在该字段的term frequency（总是返回）
- term 位置（`positions`：true）
- 开始和结束偏移量（`offsets`：true）
- term payload（`payloads`：true），被base64编码

如果所请求的信息没有存储在索引中，则将在可能的情况下即时计算。另外，对于甚至不存在于索引中但是由用户提供的文档，可以计算term vector。

![警告](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/warning.png)

开始和结束偏移假定使用UTF-16编码。如果要使用这些偏移量来获取产生该token的原始文本，则应确保使用UTF-16编码的字符串,子字符串编码也应该是UTF-16。

#### Term statistics

设置`term_statistics`为`true`（默认是`false`）将返回

- 总term频率（所有文档中term的出现频率）
- 文档频率（包含当前term的文档数量）

默认情况下，这些值不会被返回，因为term统计可能会有严重的性能影响。

#### 字段统计

设置`field_statistics`为`false`（默认是`true`）将省略：

- document数量（多少个document包含这个字段）
- 文档频率总和（本字段所有term的文档频率总和）
- 总term频率之和（该字段中每个term的总频率之和）

#### Terms 过滤

使用参数`filter`，返回的term也可以根据其tf-idf分数进行过滤。这可能是有用的，以便找出一个文档的一个很好的特征向量。此功能与“ [更像此查询”](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html)的[第二阶段](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html#mlt-query-term-selection)类似。见[例5](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html#docs-termvectors-terms-filtering) 供使用。

支持以下子参数：

| `max_num_terms`   | 每个字段必须返回的最大条数。默认为`25`。     |
| ----------------- | -------------------------- |
| `min_term_freq`   | 在源文档中忽略小于这个频率的单词。默认为`1`。   |
| `max_term_freq`   | 在源文件中忽略超过这个频率的单词。默认为无界。    |
| `min_doc_freq`    | 忽略至少在这么多文档中没有出现的术语。默认为`1`。 |
| `max_doc_freq`    | 忽略在这么多文档中出现的单词。默认为无界。      |
| `min_word_length` | 低于该字的最小字长将被忽略。默认为`0`。      |
| `max_word_length` | 字的最大字长将被忽略。默认为无界（`0`）。     |

### 行为(Behaviour)

term和字段统计不准确。删除的document不被考虑在内。这些信息仅仅是被请求文档所在分片的检索信息。因此，术语和字段统计信息只能用作相对度量值，而绝对值在这种情况下是没有意义的。默认情况下，当请求人工文档的term vector时，随机选择一个从中获取统计信息的分片。使用`routing`只投中一特定的碎片。

#### 示例：返回存储的term vector

首先，我们创建一个存储term vector的索引，请求如下：

```
PUT / twitter // twitter /
{“映射”：{{ “映射” ：{  
    “tweet”：{“tweet” ：{ 
      “属性”：{“属性” ：{ 
        “文字”：{“文字” ：{ 
          “type”：“text”，“type” ：“text” ， 
          “term_vector”：“with_positions_offsets_payloads”，“term_vector” ：“with_positions_offsets_payloads” ， 
          “store”：是的，“store” ：是的，  
          “分析仪”：“fulltext_analyzer”“分析仪” ：“fulltext_analyzer”  
         }，}，
         “全名”： {“全名” ：{ 
          “type”：“text”，“type” ：“text” ， 
          “term_vector”：“with_positions_offsets_payloads”，“term_vector” ：“with_positions_offsets_payloads” ， 
          “分析仪”：“fulltext_analyzer”“分析仪” ：“fulltext_analyzer”  
        }}
      }}
    }}
  }，}，
  “设置”：{“设置” ：{  
    “index”：{“index” ：{  
      “number_of_shards”：1，“number_of_shards” ：1 ，  
      “number_of_replicas”：0“number_of_replicas” ：0  
    }，}，
    “分析”：{“分析” ：{ 
      “分析仪”：{“分析仪” ：{ 
        “fulltext_analyzer”：{“fulltext_analyzer” ：{ 
          “type”：“custom”，“type” ：“custom” ， 
          “tokenizer”：“空白”，“tokenizer” ：“空白” ， 
          “过滤器”：[“过滤器” ：[ 
            “小写”，“小写字母” ，
            “type_as_payload”“type_as_payload”
          ]]
        }}
      }}
    }}
  }}
}}
```

其次，我们添加一些document：

```
curl -XPUT 'localhost:9200/twitter/tweet/1?pretty' -H 'Content-Type: application/json' -d'
{
  "fullname" : "John Doe",
  "text" : "twitter test test test "
}
'
curl -XPUT 'localhost:9200/twitter/tweet/2?pretty' -H 'Content-Type: application/json' -d'
{
  "fullname" : "Jane Doe",
  "text" : "Another twitter test ..."
}
'
```

以下请求返回`text`文档`1`（John Doe）中的所有字段信息和统计信息 ：

```
curl -XGET 'localhost:9200/twitter/tweet/1/_termvectors?pretty' -H 'Content-Type: application/json' -d'
{
  "fields" : ["text"],
  "offsets" : true,
  "payloads" : true,
  "positions" : true,
  "term_statistics" : true,
  "field_statistics" : true
}
'
```

```
{
    "_id": "1",
    "_index": "twitter",
    "_type": "tweet",
    "_version": 1,
    "found": true,
    "took": 6,
    "term_vectors": {
        "text": {
            "field_statistics": {
                "doc_count": 2,
                "sum_doc_freq": 6,
                "sum_ttf": 8
            },
            "terms": {
                "test": {
                    "doc_freq": 2,
                    "term_freq": 3,
                    "tokens": [
                        {
                            "end_offset": 12,
                            "payload": "d29yZA==",
                            "position": 1,
                            "start_offset": 8
                        },
                        {
                            "end_offset": 17,
                            "payload": "d29yZA==",
                            "position": 2,
                            "start_offset": 13
                        },
                        {
                            "end_offset": 22,
                            "payload": "d29yZA==",
                            "position": 3,
                            "start_offset": 18
                        }
                    ],
                    "ttf": 4
                },
                "twitter": {
                    "doc_freq": 2,
                    "term_freq": 1,
                    "tokens": [
                        {
                            "end_offset": 7,
                            "payload": "d29yZA==",
                            "position": 0,
                            "start_offset": 0
                        }
                    ],
                    "ttf": 2
                }
            }
        }
    }
}
```

#### 示例：实时生成term vector

未明确存储在索引中的term vector将自动计算。以下请求返回document `1`中字段的所有信息和统计信息，即使这些term尚未明确存储在索引中。请注意，对于字段`text`，这些term不会重新生成。

```
curl -XGET 'localhost:9200/twitter/tweet/1/_termvectors?pretty' -H 'Content-Type: application/json' -d'
{
  "fields" : ["text", "some_field_without_term_vectors"],
  "offsets" : true,
  "positions" : true,
  "term_statistics" : true,
  "field_statistics" : true
}
'
```

#### 例如：手动document

term vectot也可以为人造文档生成，也就是说文档不存在于索引中。例如，以下请求将返回与示例1相同的结果。使用的映射由`index`and 确定 `type`。

**如果启用了动态映射（默认），则不会在原始映射中的文档字段将被动态创建。**

```
curl -XGET 'localhost:9200/twitter/tweet/_termvectors?pretty' -H 'Content-Type: application/json' -d'
{
  "doc" : {
    "fullname" : "John Doe",
    "text" : "twitter test test test"
  }
}
'
```

##### 单个字段分析

另外，通过使用该`per_field_analyzer`参数可以提供与字段自身提供的有所不同。这对于以任何方式生成term vectors都很有用，特别是在使用人造文档时。当为已经存储了term vector的字段提供分析器时，term vector将被重新生成。

```
curl -XGET 'localhost:9200/twitter/tweet/_termvectors?pretty' -H 'Content-Type: application/json' -d'
{
  "doc" : {
    "fullname" : "John Doe",
    "text" : "twitter test test test"
  },
  "fields": ["fullname"],
  "per_field_analyzer" : {
    "fullname": "keyword"
  }
}
'
```

响应：

```
{
  "_index": "twitter",
  "_type": "tweet",
  "_version": 0,
  "found": true,
  "took": 6,
  "term_vectors": {
    "fullname": {
       "field_statistics": {
          "sum_doc_freq": 2,
          "doc_count": 4,
          "sum_ttf": 4
       },
       "terms": {
          "John Doe": {
             "term_freq": 1,
             "tokens": [
                {
                   "position": 0,
                   "start_offset": 0,
                   "end_offset": 8
                }
             ]
          }
       }
    }
  }
}
```

#### 示例：Terms filtering

最后，返回的term可以根据他们的tf-idf分数进行过滤。在下面的例子中，我们从具有给定“plot”字段值的人造文档中获得三个最“interesting”的关键字。请注意，关键字“Tony”或任何停用词不是响应的一部分，因为它们的tf-idf必须特别低。

```
curl -XGET 'localhost:9200/imdb/movies/_termvectors?pretty' -H 'Content-Type: application/json' -d'
{
    "doc": {
      "plot": "When wealthy industrialist Tony Stark is forced to build an armored suit after a life-threatening incident, he ultimately decides to use its technology to fight against evil."
    },
    "term_statistics" : true,
    "field_statistics" : true,
    "positions": false,
    "offsets": false,
    "filter" : {
      "max_num_terms" : 3,
      "min_term_freq" : 1,
      "min_doc_freq" : 1
    }
}
'
```

响应：

```
{
   "_index": "imdb",
   "_type": "movies",
   "_version": 0,
   "found": true,
   "term_vectors": {
      "plot": {
         "field_statistics": {
            "sum_doc_freq": 3384269,
            "doc_count": 176214,
            "sum_ttf": 3753460
         },
         "terms": {
            "armored": {
               "doc_freq": 27,
               "ttf": 27,
               "term_freq": 1,
               "score": 9.74725
            },
            "industrialist": {
               "doc_freq": 88,
               "ttf": 88,
               "term_freq": 1,
               "score": 8.590818
            },
            "stark": {
               "doc_freq": 44,
               "ttf": 47,
               "term_freq": 1,
               "score": 9.272792
            }
         }
      }
   }
}
```
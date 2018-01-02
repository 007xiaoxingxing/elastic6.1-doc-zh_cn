## Term Vectors

返回特定文档字段中的term信息和统计信息。文档可以存储在索引中或由用户人为提供。term vector是默认[实时](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#realtime)的，而不是实时的。这可以通过设置`realtime`参数来改变`false`。

```
GET / twitter / tweet / 1 / _termvectors
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/1.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/1.json)[ ]()

或者，您可以使用url中的参数指定要检索信息的字段

```
GET / twitter / tweet / 1 / _termvectors？fields = message
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/2.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/2.json)[ ]()

或者通过在请求正文中添加请求的字段（请参见下面的示例）。字段也可以使用通配符以与[多重匹配查询](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html)相似的方式指定

![警告](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/warning.png)

请注意，`/_termvector`在2.0中不推荐使用，并替换为`/_termvectors`。

### 返回值[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

可以要求三种类型的价值：*期限信息*，*期限统计* 和*现场统计*。默认情况下，所有字段的所有术语信息和字段统计信息都会返回，但是没有任何术语统计信

#### 术语信息[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

- 在该领域的短期频率（总是返回）
- 长期职位（`positions`：真）
- 开始和结束偏移量（`offsets`：true）
- 有效负载（`payloads`：真），作为base64编码的字节

如果所请求的信息没有存储在索引中，则将在可能的情况下即时计算。另外，对于甚至不存在于索引中但是由用户提供的文档，可以计算词条向量。

![警告](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/warning.png)

开始和结束偏移假定正在使用UTF-16编码。如果要使用这些偏移量来获取产生该令牌的原始文本，则应确保使用UTF-16编码的字符串也是使用子字符串编码的。

#### 术语统计[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

设置`term_statistics`为`true`（默认是`false`）将返回

- 总学期频率（所有文档中术语的出现频率）
- 文档频率（包含当前期限的文档数量）

默认情况下，这些值不会被返回，因为术语统计可能会有严重的性能影响。

#### 字段统计[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

设置`field_statistics`为`false`（默认是`true`）将省略：

- 文件数量（多少个文件包含这个字段）
- 文档频率总和（本领域所有术语的文档频率总和）
- 总术语频率之和（该术语中每个术语的总术语频率之和）

#### 术语过滤[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

使用参数`filter`，返回的条件也可以根据其tf-idf分数进行过滤。这可能是有用的，以便找出一个文档的一个很好的特征向量。此功能与“ [更像此查询”](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html)的[第二阶段](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html#mlt-query-term-selection)类似。见[例如5](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html#docs-termvectors-terms-filtering) 供使用。

支持以下子参数：

| `max_num_terms`   | 每个字段必须返回的最大条数。默认为`25`。     |
| ----------------- | -------------------------- |
| `min_term_freq`   | 在源文档中忽略小于这个频率的单词。默认为`1`。   |
| `max_term_freq`   | 在源文件中忽略超过这个频率的单词。默认为无界。    |
| `min_doc_freq`    | 忽略至少在这么多文档中没有出现的术语。默认为`1`。 |
| `max_doc_freq`    | 忽略在这么多文档中出现的单词。默认为无界。      |
| `min_word_length` | 低于该字的最小字长将被忽略。默认为`0`。      |
| `max_word_length` | 字的最大字长将被忽略。默认为无界（`0`）。     |

### 行为[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

术语和现场统计不准确。删除的文件不被考虑在内。这些信息仅仅是被请求文档所在分片的检索信息。因此，术语和字段统计信息只能用作相对度量值，而绝对值在这种情况下是没有意义的。默认情况下，当请求人工文档的术语向量时，随机选择一个从中获取统计信息的分片。使用`routing`只投中一特定的碎片。

#### 示例：返回存储的术语向量[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

首先，我们创建一个存储术语向量，有效载荷等的索引：

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

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/3.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/3.json)[ ]()

其次，我们添加一些文件：

```
PUT / twitter / tweet / 1/ twitter / tweet / 1
{{
  “全名”：“John Doe”，“全名” ：“John Doe” ，  
  “文字”：“微博测试测试”“文字” ：“微博测试测试”  
}}

PUT / twitter / tweet / 2/ twitter / tweet / 2
{{
  “全名”：“Jane Doe”，“全名” ：“Jane Doe” ，  
  “文字”：“另一个推特测试...”“文字” ：“另一个推特测试...”  
}}
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/4.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/4.json)[ ]()

以下请求返回`text`文档`1`（John Doe）中的所有字段信息和统计信息 ：

```
GET / twitter / tweet / 1 / _termvectors/ twitter / tweet / 1 / _termvectors
{{
  “fields”：[“text”]，“fields” ：[ “text” ]，  
  “抵消”：是的，“抵消” ：是的，  
  “有效载荷”：是的，“有效载荷” ：是的，  
  “职位”：是的，“职位” ：是的，  
  “term_statistics”：是的，“term_statistics” ：是的，  
  “field_statistics”：true“field_statistics” ：true  
}}
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/5.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/5.json)[ ]()

响应：

```
{
    “_id”：“1”，“_id” ：“1” ， 
    “_index”：“twitter”，“_index” ：“twitter” ， 
    “_type”：“鸣叫”，“_type” ：“鸣叫” ， 
    “_version”：1，“_version” ：1 ， 
    “找到”：是的，“找到” ：是的， 
    “拿”：6，“拿” ：6 ， 
    “term_vectors”：{“term_vectors” ：{ 
        “文字”：{“文字” ：{ 
            “field_statistics”：{“field_statistics” ：{ 
                “doc_count”：2，“doc_count” ：2 ， 
                “sum_doc_freq”：6，“sum_doc_freq” ：6 ， 
                “sum_ttf”：8“sum_ttf” ：8 
            }，}，
            “条款”：{“条款” ：{ 
                “测试”：{“测试” ：{ 
                    “doc_freq”：2，“doc_freq” ：2 ， 
                    “term_freq”：3，“term_freq” ：3 ， 
                    “令牌”：[“令牌” ：[ 
                        {{
                            “end_offset”：12，“end_offset” ：12 ， 
                            “payload”：“d29yZA ==”，“payload” ：“d29yZA ==” ， 
                            “职位”：1，“职位” ：1 ， 
                            “start_offset”：8“start_offset” ：8 
                        }，}，
                        {{
                            “end_offset”：17，“end_offset” ：17 ， 
                            “payload”：“d29yZA ==”，“payload” ：“d29yZA ==” ， 
                            “职位”：2，“职位” ：2 ， 
                            “start_offset”：13“start_offset” ：13 
                        }，}，
                        {{
                            “end_offset”：22，“end_offset” ：22 ， 
                            “payload”：“d29yZA ==”，“payload” ：“d29yZA ==” ， 
                            “职位”：3，“职位” ：3 ， 
                            “start_offset”：18“start_offset” ：18 
                        }}
                    ]]
                    “ttf”：4“ttf” ：4 
                }，}，
                “推特”： {“twitter” ：{ 
                    “doc_freq”：2，“doc_freq” ：2 ， 
                    “term_freq”：1，“term_freq” ：1 ， 
                    “令牌”：[“令牌” ：[ 
                        {{
                            “end_offset”：7，“end_offset” ：7 ， 
                            “payload”：“d29yZA ==”，“payload” ：“d29yZA ==” ， 
                            “位置”：0，“位置” ：0 ， 
                            “start_offset”：0“start_offset” ：0 
                        }}
                    ]]
                    “ttf”：2“ttf” ：2 
                }}
            }}
        }}
    }}
}}
```

#### 示例：在[编辑中](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)生成术语向量

未明确存储在索引中的术语向量将自动计算。以下请求返回文档中字段的所有信息和统计信息`1`，即使这些条款尚未明确存储在索引中。请注意，对于该字段`text`，这些条款不会重新生成。

```
GET / twitter / tweet / 1 / _termvectors/ twitter / tweet / 1 / _termvectors
{{
  “fields”：[“text”，“some_field_without_term_vectors”]，“fields” ：[ “text” ，“some_field_without_term_vectors” ]，   
  “抵消”：是的，“抵消” ：是的，  
  “职位”：是的，“职位” ：是的，  
  “term_statistics”：是的，“term_statistics” ：是的，  
  “field_statistics”：true“field_statistics” ：true  
}}
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/6.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/6.json)[ ]()

#### 例如：人造文件[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

术语向量也可以为人造文档生成，也就是说文档不存在于索引中。例如，以下请求将返回与示例1相同的结果。使用的映射由`index`and 确定 `type`。

**如果启用了动态映射（默认），则不会在原始映射中的文档字段将被动态创建。**

```
GET / twitter / tweet / _termvectors/ twitter / tweet / _termvectors
{{
  “doc”：{“doc” ：{  
    “全名”：“John Doe”，“全名” ：“John Doe” ，  
    “文字”：“微博测试测试”“文字” ：“微博测试测试”  
  }}
}}
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/7.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/7.json)[ ]()

##### 每场分析器[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

另外，通过使用该`per_field_analyzer`参数可以提供与现场不同的分析仪。这对于以任何方式生成术语向量都很有用，特别是在使用人造文档时。当为已经存储了术语矢量的字段提供分析器时，术语矢量将被重新生成。

```
GET / twitter / tweet / _termvectors/ twitter / tweet / _termvectors
{{
  “doc”：{“doc” ：{  
    “全名”：“John Doe”，“全名” ：“John Doe” ，  
    “文字”：“微博测试测试”“文字” ：“微博测试测试”  
  }，}，
  “fields”：[“fullname”]，“fields” ：[ “fullname” ]， 
  “per_field_analyzer”：{“per_field_analyzer” ：{  
    “全名”：“关键字”“全名” ：“关键字” 
  }}
}}
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/8.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/8.json)[ ]()

响应：

```
{
  “_index”：“twitter”，“_index” ：“twitter” ， 
  “_type”：“鸣叫”，“_type” ：“鸣叫” ， 
  “_version”：0，“_version” ：0 ， 
  “找到”：是的，“找到” ：是的， 
  “拿”：6，“拿” ：6 ， 
  “term_vectors”：{“term_vectors” ：{ 
    “全名”： {“全名” ：{ 
       “field_statistics”：{“field_statistics” ：{ 
          “sum_doc_freq”：2，“sum_doc_freq” ：2 ， 
          “doc_count”：4，“doc_count” ：4 ， 
          “sum_ttf”：4“sum_ttf” ：4 
       }，}，
       “条款”：{“条款” ：{ 
          “John Doe”：{“John Doe” ：{ 
             “term_freq”：1，“term_freq” ：1 ， 
             “令牌”：[“令牌” ：[ 
                {{
                   “位置”：0，“位置” ：0 ， 
                   “start_offset”：0，“start_offset” ：0 ， 
                   “end_offset”：8“end_offset” ：8 
                }}
             ]]
          }}
       }}
    }}
  }}
}}
```

#### 示例：术语筛选[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/termvectors.asciidoc)

最后，返回的条款可以根据他们的tf-idf分数进行过滤。在下面的例子中，我们从具有给定“绘图”字段值的人造文档中获得三个最“有趣”的关键字。请注意，关键字“Tony”或任何停用词不是响应的一部分，因为它们的tf-idf必须太低。

```
GET / imdb / movies / _termvectors/ imdb / movies / _termvectors
{{
    “doc”：{“doc” ：{ 
      “情节”：“当富有的实业家托尼·斯塔克被迫在威胁生命的事件之后建立一套装甲装甲时，他最终决定用技术来对付邪恶。“情节” ：“当富有的实业家托尼·斯塔克被迫在威胁生命的事件之后建立一套装甲装甲时，他最终决定用技术来对付邪恶。 
    }，}，
    “term_statistics”：是的，“term_statistics” ：是的，  
    “field_statistics”：true，“field_statistics” ：true ，  
    “职位”：假，“职位” ：假， 
    “抵消”：假，“抵消” ：假， 
    “过滤器”：{“过滤器” ：{  
      “max_num_terms”：3，“max_num_terms” ：3 ，  
      “min_term_freq”：1，“min_term_freq” ：1 ，  
      “min_doc_freq”：1“min_doc_freq” ：1  
    }}
}}
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/9.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-termvectors/9.json)[ ]()

响应：

```
{
   “_index”：“imdb”，“_index” ：“imdb” ， 
   “_type”：“电影”，“_type” ：“电影” ， 
   “_version”：0，“_version” ：0 ， 
   “找到”：是的，“找到” ：是的， 
   “term_vectors”：{“term_vectors” ：{ 
      “情节”：{“情节” ：{ 
         “field_statistics”：{“field_statistics” ：{ 
            “sum_doc_freq”：3384269，“sum_doc_freq” ：3384269 ， 
            “doc_count”：176214，“doc_count” ：176214 ， 
            “sum_ttf”：3753460“sum_ttf” ：3753460 
         }，}，
         “条款”：{“条款” ：{ 
            “装甲”：{“装甲” ：{ 
               “doc_freq”：27，“doc_freq” ：27 ， 
               “ttf”：27，“ttf” ：27 ， 
               “term_freq”：1，“term_freq” ：1 ， 
               “得分”：9.74725“得分” ：9.74725 
            }，}，
            “实业家”： {“实业家” ：{ 
               “doc_freq”：88，“doc_freq” ：88 ， 
               “ttf”：88，
               “term_freq”：1，
               “得分”：8.590818
            }，
            “stark”：{
               “doc_freq”：44，
               “ttf”：47，
               “term_freq”：1，
               “得分”：9.272792
            }
         }
      }
   }
}
```
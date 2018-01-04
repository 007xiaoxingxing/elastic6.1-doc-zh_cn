## URI Search

搜索请求可以通过提供请求参数纯粹使用URI来执行。在使用这种模式执行搜索时，并不是所有的搜索选项都是公开的，但是对于快速“curl test”来说，它可能非常方便。这里是一个例子：

```
curl -XGET 'localhost:9200/twitter/tweet/_search?q=user:kimchy&pretty'
```

这里是一个示例回应：

```
{
  "took" : 12,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 0.2876821,
    "hits" : [
      {
        "_index" : "twitter",
        "_type" : "tweet",
        "_id" : "gdXFuWABFKq0htem1BZ1",
        "_score" : 0.2876821,
        "_routing" : "kimchy",
        "_source" : {
          "user" : "kimchy",
          "postDate" : "2009-11-15T14:12:12",
          "message" : "trying out Elasticsearch"
        }
      }
    ]
  }
}

```

### 参数

URI中允许的参数是：

| 名称                    | 描述                                       |
| --------------------- | ---------------------------------------- |
| `q`                   | 查询字符串（映射到`query_string`查询，请参阅 [*Query String Query*](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html)以获取更多详细信息）。 |
| `df`                  | 在查询中未定义字段前缀时使用的默认字段。                     |
| `analyzer`            | 分析查询字符串时要使用的分析器名称。                       |
| `analyze_wildcard`    | 是否应该分析通配符和前缀查询。默认为`false`。               |
| `batched_reduce_size` | 在协调节点上一次减少的分片结果的数量。如果请求中的潜在分片数可能很大，则应该使用此值作为保护机制来减少每个搜索请求的内存开销。 |
| `default_operator`    | 要使用的默认运算符可以是`AND`或 `OR`。默认为`OR`。         |
| `lenient`             | 如果设置为true将导致基于格式的失败（如提供文本到数字字段）被忽略。默认为false。 |
| `explain`             | 对于每个命中，包含如何计算命中计分的解释。                    |
| `_source`             | 设置为`false`禁止检索`_source`字段。您也可以使用`_source_include`＆获取部分文档`_source_exclude`（请参阅[request body](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-source-filtering.html) 文档以获取更多详细信息） |
| `stored_fields`       | 选择性存储的文件字段返回给每个命中，逗号分隔。不指定任何值将导致没有字段返回。  |
| `sort`                | 排序执行。可以是`fieldName`，或者是 `fieldName:asc`/ 的形式`fieldName:desc`。fieldName可以是文档中的实际字段，也可以是`_score`根据分数表示排序的特殊名称。可以有几个`sort`参数（顺序是重要的）。 |
| `track_scores`        | 排序时，设置为`true`仍然跟踪分数，并返回它们作为每个命中的一部分。     |
| `track_total_hits`    | 设置为`false`禁用跟踪匹配查询的总点击次数。（请参阅[*索引分类*](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules-index-sorting.html)了解更多详情）。默认为true。 |
| `timeout`             | 搜索超时，限制搜索请求在指定的时间值内执行，并在到期时累积至点的保留时间。默认没有超时。 |
| `terminate_after`     | 为每个分片收集的文档的最大数量，一旦达到该数量，查询执行将提前终止。如果设置，则响应将具有一个布尔字段`terminated_early`来指示查询执行是否实际已经terminate_early。缺省为no terminate_after。 |
| `from`                | 从命中的索引开始返回。默认为`0`。                       |
| `size`                | 要返回的点击次数。默认为`10`。                        |
| `search_type`         | 要执行的搜索操作的类型。可以 `dfs_query_then_fetch`或`query_then_fetch`。默认为`query_then_fetch`。有关可以执行的不同[*Search Type*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-search-type.html)的更多详细信息，请参阅[*Search Type*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-search-type.html)。 |
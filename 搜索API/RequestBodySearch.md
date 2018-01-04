## Request Body Search

搜索请求可以在request body内使用包括[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)的来执行 。下边是一个例子：

```
curl -XGET 'localhost:9200/twitter/tweet/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```



这里是一个示例回应：

```
{
  "took" : 8,
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

| `timeout`             | 搜索超时，限制搜索请求在指定的时间值内执行，并在到期时累积至点的保留时间。默认没有超时。请参阅[Time units](https://www.elastic.co/guide/en/elasticsearch/reference/current/common-options.html#time-units)。 |
| --------------------- | ---------------------------------------- |
| `from`                | 从特定的偏移量中检索命中。默认为`0`。                     |
| `size`                | 要返回的hits次数。默认为`10`。如果您不关心获得一些命中，但只关于匹配和/或聚合的数量，设置值`0`将有助于性能。 |
| `search_type`         | 要执行的搜索操作的类型。可以 `dfs_query_then_fetch`或`query_then_fetch`。默认为`query_then_fetch`。查看更多[*搜索类型*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-search-type.html)。 |
| `request_cache`       | 设置为`true`或`false`启用或禁用请求的搜索结果，其中的缓存`size`为0，即聚合和建议（无排名靠前返回）。请参阅[碎片请求缓存](https://www.elastic.co/guide/en/elasticsearch/reference/current/shard-request-cache.html)。 |
| `terminate_after`     | 为每个分片收集的文档的最大数量，一旦达到该数量，查询执行将提前终止。如果设置，则响应将具有一个布尔字段`terminated_early`来指示查询执行是否实际已经terminate_early。缺省为no terminate_after。 |
| `batched_reduce_size` | 在协调节点上一次减少的分片结果的数量。如果请求中的潜在分片数可能很大，则应该使用此值作为保护机制来减少每个搜索请求的内存开销。 |

除此之外，`search_type`和`request_cache`必须作为查询字符串参数传递。搜索请求的其余部分应该在主体内传递。正文内容也可以作为名为REST的参数传递`source`。

HTTP GET和HTTP POST都可以用来执行搜索正文。由于不是所有的客户端都支持GET，所以POST也是允许的。

### 快速检查任何匹配的文档

如果我们只是想知道，如果有匹配的特定查询的任何文件，我们可以通过设置`size`来`0`表明，我们没有兴趣在搜索结果中。此外，我们可以设置`terminate_after`为`1` 指示只要找到第一个匹配文档（每个分片）就可以终止查询执行。

```
curl -XGET 'localhost:9200/_search?q=message:elasticsearch&size=0&terminate_after=1&pretty'
```

响应将不包含任何命中为`size`被设置为`0`。这 `hits.total`将等于`0`，表示没有匹配的文档，或者大于`0`意味着当提前终止时至少有与匹配查询的文档相同的文档。此外，如果查询被提前终止，该`terminated_early`标志将被设置为`true`。

```
{
  "took": 3,
  "timed_out": false,
  "terminated_early": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped" : 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.0,
    "hits": []
  }
}
```

将`took`在响应时间上包含了这个请求了处理，之后开始迅速节点收到的查询，直到所有与搜索相关的工作已经完成，并在上述JSON被返回到客户端之前的毫秒数。这意味着它包括在线程池中等待的时间，在整个集群中执行分布式搜索并收集所有结果。



## Query

搜索请求体内的查询元素允许使用[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)来定义查询。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```

## From / Size

结果的分页可以通过使用`from`和`size` 参数来完成。该`from`参数定义了您想要获取的第一个结果的偏移量。该`size`参数允许您配置要返回的最大命中数量。

虽然`from`并`size`可以被设置为请求参数，它们也可以在搜索身体内。`from`默认为`0`，`size` 默认为`10`。

```
curl -XGET 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
'
```

请注意，`from`+ `size`不能超过`index.max_result_window` 默认为10,000 的索引设置。查看[Scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html)或[Search After](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-search-after.html) API以获得更高效的深度滚动方法。
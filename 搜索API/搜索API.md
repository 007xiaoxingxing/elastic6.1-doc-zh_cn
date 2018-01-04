# 搜索API

除[*Explain API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-explain.html)接口以外，大多数搜索API是[multi-index，multi-type](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html#search-multi-index-type)。

## 路由

执行搜索时，它将广播到所有索引/索引碎片（副本之间的循环）。可以通过提供`routing`参数来控制将搜索哪些碎片。例如，在对推文进行索引时，路由值可以是用户名：

```
curl -XPOST 'localhost:9200/twitter/tweet?routing=kimchy&pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "postDate" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

在这种情况下，如果我们只想在特定用户的推特上搜索，我们可以将其指定为路由，导致只搜索相关的分片：

```
curl -XPOST 'localhost:9200/twitter/tweet/_search?routing=kimchy&pretty' -H 'Content-Type: application/json' -d'
{
    "query": {
        "bool" : {
            "must" : {
                "query_string" : {
                    "query" : "some query string here"
                }
            },
            "filter" : {
                "term" : { "user" : "kimchy" }
            }
        }
    }
}
'

```



路由参数可以是多值的，用逗号分隔的字符串表示。这将导致路由值匹配的相关分片。

## 自适应副本选择

作为以循环方式发送到数据副本的请求的替代方法，您可以启用自适应副本选择。这允许协调节点基于以下标准将请求发送到被认为是“最佳”的副本：

- 协调节点和包含数据副本的节点之间的过去请求的响应时间
- 过去搜索请求的时间花费在包含数据的节点上执行
- 包含数据的节点上的搜索线程池的队列大小

这可以通过将动态集群的`cluster.routing.use_adaptive_replica_selection`设置为`true`：

```
curl -XPUT 'localhost:9200/_cluster/settings?pretty' -H 'Content-Type: application/json' -d'
{
    "transient": {
        "cluster.routing.use_adaptive_replica_selection": true
    }
}
'

```



## 统计信息组

搜索可以与统计组相关联，统计组保持每个组的统计聚合。它可以稍后使用[索引统计](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-stats.html) API特别检索 。例如，这是一个搜索主体请求，将请求与两个不同的组关联：

```
curl -XPOST 'localhost:9200/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_all" : {}
    },
    "stats" : ["group1", "group2"]
}
'

```



## 全局搜索超时

独立搜索可以设置一个超时参数作为 [*请求正文搜索的一部分*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)。由于搜索请求可能来自多个来源，因此Elasticsearch为全局搜索超时设置了动态群集级别设置，适用于在[*请求正文搜索中*](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)未设置超时的所有搜索请求。默认值是没有全局超时。设置密钥是`search.default_search_timeout`，可以使用“ [*集群更新设置”*](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)接口进行[*设置*](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html)。设置此值可`-1`将全局搜索超时重置为不超时。

## 搜索取消

可以使用标准[任务取消](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html#task-cancellation) 机制取消搜索。默认情况下，正在运行的搜索仅检查是否在段边界上被取消，因此可以通过大段延迟取消。通过将动态群集级别设置`search.low_level_cancellation`为，可以提高搜索取消响应速度`true`。但是，它带来了更多的频繁取消检查的额外开销，这在大型快速运行的搜索查询中可能是显而易见的。更改此设置仅影响更改后开始的搜索。
## 删除API

删除API允许从特定索引中删除一个JSON文档。以下示例从称为twitter的索引中删除名为tweet的类型的JSON文档，其ID为1：

```shell
curl -XDELETE 'localhost:9200/twitter/tweet/1?pretty'
```

上述删除操作的结果是：

```shell
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "1",
    "_version" : 2,
    "_primary_term": 1,
    "_seq_no": 5,
    "result": "deleted"
}
```

### 版本（Versioning）

索引的每个文档都是版本化的。在删除文件时， `version`可以指定确认删除相关文件，实际上是删除文件，同时没有改变。每个在文档上执行的写入操作，包括删除，使其版本增加。

### 路由（Routing）

当使用控制路由的能力进行索引时，为了删除文档，还应提供路由值。例如：

```shell
curl -XDELETE 'localhost:9200/twitter/tweet/1?routing=kimchy&pretty'
```

以上将删除ID为1的推文，但会根据用户进行路由。请注意，发出一个没有正确路由的删除将导致文档不被删除。

当`_routing`映射设置为，`required`并且没有指定路由值时，删除API将抛出`RoutingMissingException`并拒绝请求。

### 自动创建索引 

如果使用[外部版本控制变体](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)，则删除操作将自动创建一个索引（如果尚未创建 索引）（检出用于手动创建索引的[创建索引API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)），并且还会自动创建特定类型的动态类型映射以前没有创建过（检出[put mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html)API用于手动创建类型映射）。

### 分布式(Distributed)

删除操作被散列到特定的分片ID中。然后它被重定向到该ID组中的主碎片，并且被复制（如果需要）到该组ID内的碎片副本。

### 等待活动碎片(Wait For Active Shards)

在发出删除请求时，可以设置该`wait_for_active_shards` 参数，以在开始处理删除请求之前要求最小数量的分片副本处于活动状态。有关更多详细信息和用法示例，请参阅 [此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)。

### 刷新

控制此请求所做的更改对搜索是否可见。看 [*?refresh*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)。

### 超时

当执行删除操作时，分配用于执行删除操作的主分片可能不可用。造成这种情况的一些原因可能是主分片正在从商店中恢复或正在进行重新安置。默认情况下，删除操作将在主分片上等待最多1分钟，然后出现故障并作出响应并显示错误。该`timeout`参数可以用来明确指定等待的时间。以下是将其设置为5分钟的示例：

```shell
curl -XDELETE 'localhost:9200/twitter/tweet/1?timeout=5m&pretty'
```
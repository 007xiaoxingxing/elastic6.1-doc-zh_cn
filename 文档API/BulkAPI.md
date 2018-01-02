## Bulk API

Bulk API使得可以在单个API调用中执行许多索引/删除操作。这可以大大提高索引速度。

**客户端支持批量请求**

一些官方支持的客户提供帮助以协助批量请求，并将文件从一个索引重新索引到另一个索引：

- Perl

  查看[搜索:: Elasticsearch :: Client :: 5_0 :: Bulk](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Bulk) 和[Search :: Elasticsearch :: Client :: 5_0 :: Scroll](https://metacpan.org/pod/Search::Elasticsearch::Client::5_0::Scroll)

- python

  请参阅[elasticsearch.helpers。*](http://elasticsearch-py.readthedocs.org/en/master/helpers.html)

REST API接口是`/_bulk`，它期望以下换行符分隔的JSON（NDJSON）结构：

```
action_and_meta_data\n
optional_source\n
action_and_meta_data\n
optional_source\n
....
action_and_meta_data\n
optional_source\n
```

**注**：最后一行数据必须以换行符结尾`\n`。每个换行符都可以以一个回车符开头`\r`。向这个接口发送请求时， `Content-Type`头部应该被设置为`application/x-ndjson`。

可能的操作有`index`，`create`，`delete`和`update`。 `index`并`create`期望在下一行有source参数，并且`op_type`与标准索引API 的参数具有相同的语义（即，如果已经存在具有相同索引和类型的文档，则创建将失败，而索引将根据需要添加或替换文档） 。`delete`不期望在下面的行上有一个源，并且具有与标准删除API相同的语义。 `update`期望在下一行指定部分doc，upsert和脚本及其选项。

如果您要提供文本文件输入`curl`，则**必须**使用 `--data-binary`标志而不是普通的`-d`。后者不保留换行符。例：

```shell
$ cat requests
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
$ curl -s -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/_bulk --data-binary "@requests"; echo
{"took":7, "errors": false, "items":[{"index":{"_index":"test","_type":"type1","_id":"1","_version":1,"result":"created","forced_refresh":false}}]}

```

由于这种格式使用文字`\n`作为分隔符，请确保JSON操作和来源不打印。这是一个正确的批量命令序列的例子：

```shell
curl -XPOST 'localhost:9200/_bulk?pretty' -H 'Content-Type: application/json' -d'
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_type" : "type1", "_id" : "2" } }
{ "create" : { "_index" : "test", "_type" : "type1", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
'
```

这个批量操作的结果是：

```
{
   "took": 30,
   "errors": false,
   "items": [
      {
         "index": {
            "_index": "test",
            "_type": "type1",
            "_id": "1",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 0,
            "_primary_term": 1
         }
      },
      {
         "delete": {
            "_index": "test",
            "_type": "type1",
            "_id": "2",
            "_version": 1,
            "result": "not_found",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 404,
            "_seq_no" : 1,
            "_primary_term" : 2
         }
      },
      {
         "create": {
            "_index": "test",
            "_type": "type1",
            "_id": "3",
            "_version": 1,
            "result": "created",
            "_shards": {
               "total": 2,
               "successful": 1,
               "failed": 0
            },
            "status": 201,
            "_seq_no" : 2,
            "_primary_term" : 3
         }
      },
      {
         "update": {
            "_index": "test",
            "_type": "type1",
            "_id": "1",
            "_version": 2,
            "result": "updated",
            "_shards": {
                "total": 2,
                "successful": 1,
                "failed": 0
            },
            "status": 200,
            "_seq_no" : 3,
            "_primary_term" : 4
         }
      }
   ]
}
```

接口是`/_bulk`，`/{index}/_bulk`和`{index}/{type}/_bulk`。当提供索引或索引/类型时，默认情况下，它们将在未明确提供的批量项目上使用。

格式说明。这里的想法是尽可能快地处理这个问题。由于某些操作将被重定向到其他节点上的其他分片，因此只能`action_meta_data`在接收节点端进行分析。

使用这个协议的客户端库应该努力在客户端做类似的事情，尽可能减少缓冲。

对批量操作的响应是一个很大的JSON结构，每个操作执行的结果都是单独的。单一行动的失败并不影响其余的行动。

在单个批量调用中没有执行“正确”的操作次数。您应该尝试不同的设置以找到适合您特定工作负载的最佳大小。

如果使用HTTP API，请确保客户端不会发送HTTP块，因为这会降低速度。

### 版本

每个批量项目可以包含使用该`version`字段的版本值 。它会根据`_version`映射自动遵循索引/删除操作的行为。它也支持`version_type`（参见[版本](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-versioning)）

### 路由

每个批量项目可以使用该`routing`字段包含路由值 。它会根据`_routing`映射自动遵循索引/删除操作的行为。

### 等待活动碎片

进行批量呼叫时，您可以设置该`wait_for_active_shards` 参数，以在开始处理批量请求之前要求最小数量的分片副本处于活动状态。有关更多详细信息和用法示例，请参阅 [此处](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)。

### 刷新

控制此请求所做的更改对搜索是否可见。看 [刷新](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)。

### 更新

当使用`update`动作`retry_on_conflict`可以作为动作本身的字段（不在额外的有效载荷行中）时，指定在版本冲突的情况下更新应该重试的次数。

的`update`动作有效载荷，支持下面的选项：`doc` （部分文档）， ，`upsert`，`doc_as_upsert`，`script`（`params`脚本）， `lang`（脚本）和`_source`。有关选项的详细信息，请参阅更新文档。更新操作示例：

```shell
curl -XPOST 'localhost:9200/_bulk?pretty' -H 'Content-Type: application/json' -d'
{ "update" : {"_id" : "1", "_type" : "type1", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"} }
{ "update" : { "_id" : "0", "_type" : "type1", "_index" : "index1", "retry_on_conflict" : 3} }
{ "script" : { "source": "ctx._source.counter += params.param1", "lang" : "painless", "params" : {"param1" : 1}}, "upsert" : {"counter" : 1}}
{ "update" : {"_id" : "2", "_type" : "type1", "_index" : "index1", "retry_on_conflict" : 3} }
{ "doc" : {"field" : "value"}, "doc_as_upsert" : true }
{ "update" : {"_id" : "3", "_type" : "type1", "_index" : "index1", "_source" : true} }
{ "doc" : {"field" : "value"} }
{ "update" : {"_id" : "4", "_type" : "type1", "_index" : "index1"} }
{ "doc" : {"field" : "value"}, "_source": true}
'
```

### 安全

请参阅[*基于URL的访问控制*](https://www.elastic.co/guide/en/elasticsearch/reference/current/url-access-control.html)
## 通过查询API更新（Update By Query API）

最简单的用法`_update_by_query`就是对索引中的每个文档执行更新，而不更改源文件。这是 [pick up a new property](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#picking-up-a-new-property)或一些其他的在线映射变化。这里是API：

```
curl -XPOST 'localhost:9200/twitter/_update_by_query?conflicts=proceed&pretty'
```

这将返回像这样的东西：

```shell
{
  "took" : 147,
  "timed_out": false,
  "updated": 120,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 120,
  "failures" : [ ]
}
```

`_update_by_query`获取索引启动时的快照，并使用`internal`版本控制索引它的索引。这意味着如果文档在拍摄快照和处理索引请求之间发生更改，将会发生版本冲突。当版本匹配时，文档被更新并且版本号增加。

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

由于`internal`版本控制不支持将值0作为有效的版本号，因此版本等于零的文档无法使用更新， `_update_by_query`并且会使请求失败。

所有更新和查询失败都会导致`_update_by_query`中止并返回`failures`响应。已执行的更新仍然坚持。换句话说，过程不回滚，只能中止。当第一次失败导致中止时，由失败的批量请求返回的所有失败都返回到`failures`元素中; 因此可能有相当多的失败实体。

如果你想简单地计数版本冲突不会导致`_update_by_query` 中止，你可以设置`conflicts=proceed`在网址或`"conflicts": "proceed"` 请求正文。第一个例子是这样做的，因为它只是试图获取在线映射更改，而版本冲突则意味着冲突文档在更新文档的开始`_update_by_query` 时间和更新时间之间更新。这很好，因为这个更新会提取在线映射更新。

回到API格式，你可以限制`_update_by_query`到一个单一的类型。这只会更新索引中的`tweet`文件`twitter`：

```
curl -XPOST 'localhost:9200/twitter/tweet/_update_by_query?conflicts=proceed&pretty'
```

您也可以`_update_by_query`使用 [查询DSL进行](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)限制。这将更新`twitter`用户索引中的所有文档 `kimchy`：

```
curl -XPOST 'localhost:9200/twitter/_update_by_query?conflicts=proceed&pretty' -H 'Content-Type: application/json' -d'
{
  "query": { (1)
    "term": {
      "user": "kimchy"
    }
  }
}
'
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#CO19-1) | 查询必须`query`以与[搜索API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)相同的方式作为值传递给密钥。您也可以使用`q` 与搜索api相同的方式使用该参数。 |
| ---------------------------------------- | ---------------------------------------- |
|                                          |                                          |

到目前为止，我们只是在不更改源文件的情况下更新文档。这对于[pick up new properties](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#picking-up-a-new-property)来说是非常有用的， 但这只是一半的乐趣。`_update_by_query` [支持脚本](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-using.html)来更新文档。这将增加`likes`所有kimchy的鸣叫字段：

```
curl -XPOST 'localhost:9200/twitter/_update_by_query?pretty' -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.likes++",
    "lang": "painless"
  },
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
'
```

就像在[更新API中一样，](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)您可以设置`ctx.op`更改执行的操作：

- `noop`

  设置`ctx.op = "noop"`如果你的脚本决定，它没有做任何更改。这会导致`_update_by_query`从其更新中忽略该文档。这个没有任何操作会`noop`在[响应主体](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#docs-update-by-query-response-body)的柜台 上显示。

- `delete`

  设置`ctx.op = "delete"`如果你的脚本决定，该文件必须被删除。删除将`deleted`在[response body](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#docs-update-by-query-response-body)上显示。

设置`ctx.op`为其他任何东西都是错误的。设置任何其他字段`ctx`是一个错误。

请注意，我们停止指定`conflicts=proceed`。在这种情况下，我们想要一个版本冲突来中止这个过程，所以我们可以处理这个失败。

这个API不允许你移动它接触到的文档，只是修改它的源代码。这是故意的！我们没有规定将文档从原始位置删除。

也可以一次完成多个索引和多个类型的整个事件，就像搜索API一样：

```
curl -XPOST 'localhost:9200/twitter,blog/tweet,post/_update_by_query?pretty'
```

如果您提供，`routing`那么路由被复制到滚动查询，将进程限制为匹配该路由值的分片：

```
curl -XPOST 'localhost:9200/twitter/_update_by_query?routing=1&pretty'
```

默认情况下`_update_by_query`使用滚动批处理1000.您可以使用`scroll_size`URL参数更改批处理大小：

```
curl -XPOST 'localhost:9200/twitter/_update_by_query?scroll_size=100&pretty'
```

`_update_by_query`也可以通过指定一个像这样使用[Ingest Node](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)功能`pipeline`：

```shell
curl -XPUT 'localhost:9200/_ingest/pipeline/set-foo?pretty' -H 'Content-Type: application/json' -d'
{
  "description" : "sets foo",
  "processors" : [ {
      "set" : {
        "field": "foo",
        "value": "bar"
      }
  } ]
}
'
curl -XPOST 'localhost:9200/twitter/_update_by_query?pipeline=set-foo&pretty'
```

### URL参数

除了标准的参数，如`pretty`，此更新通过查询API也支持`refresh`，`wait_for_completion`，`wait_for_active_shards`，和`timeout`。

`refresh`当请求完成时，发送将更新正在更新的索引中的所有分片。这与Index API的`refresh` 参数不同，后者只会导致接收新数据的分片被编入索引。

如果请求包含，`wait_for_completion=false`那么Elasticsearch将执行一些预检检查，启动请求，然后返回一个`task` 可以与[Tasks API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#docs-update-by-query-task-api) 一起使用的取消或获取任务状态的请求。Elasticsearch也将创建这个任务的记录作为一个文件在`.tasks/task/${taskId}`。这是你的保留或删除，如你所见。当你完成它，删除它，所以Elasticsearch可以回收它使用的空间。

`wait_for_active_shards`在继续处理请求之前，控制一个分片的副本必须有效。详情请看[这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards) 。`timeout`控制每个写请求等待不可用碎片变得可用的时间。两个工作正是他们在 [批量API中的工作方式](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)。

`requests_per_second`可以被设置为任何正十进制数（`1.4`，`6`， `1000`等）和节流速率`_update_by_query`通过填充每个批次由一等待时间发出索引操作的批次。限制可以通过设置`requests_per_second`为禁用`-1`。

限制是通过在批次间等待来完成的，以便在`_update_by_query`内部使用的滚动 可以被赋予一个考虑到填充的超时。填充时间是批量除以`requests_per_second`和写入时间之间的差值。默认情况下批量大小是`1000`，所以如果`requests_per_second`设置为`500`：

```
target_time = 1000 / 500 per second = 2 seconds
wait_time = target_time - delete_time = 2 seconds - .5 seconds = 1.5 seconds   
```

由于批处理是作为单个`_bulk`请求发出的，因此大批量处理会导致Elasticsearch创建很多请求，然后等待一段时间再开始下一个处理。这是“bursty”而不是"smooth"。默认是`-1`。

### 响应正文

JSON响应如下所示：

```
{
  "took" : 147,
  "timed_out": false,
  "total": 5,
  "updated": 5,
  "deleted": 0,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "failures" : [ ]
}
```

- `took`

  整个操作从开始到结束的毫秒数。

- `timed_out`

  如果通过查询执行的更新期间执行的任何请求超时 ，则将此标志设置为`true`。

- `total`

  成功处理的文档数量。

- `updated`

  成功更新的文档数量。

- `deleted`

  已成功删除的文档数量。

- `batches`

  通过查询更新拉回滚动响应的数量。

- `version_conflicts`

  查询更新的版本冲突次数。

- `noops`

  由于用于查询更新的脚本返回`noop`值，因此被忽略的文档数量`ctx.op`。

- `retries`

  通过查询更新尝试的重试次数。`bulk`是重试的批量操作`search`的数量，是重试的搜索操作的数量。

- `throttled_millis`

  请求睡眠符合的毫秒数`requests_per_second`。

- `requests_per_second`

  查询更新过程中每秒有效执行的请求数。

- `throttled_until_millis`

  在查询响应删除时，该字段应始终等于零。它只有在使用[Task API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#docs-update-by-query-task-api)时才有意义，它表示下一次（从epoch开始以毫秒为单位）再次执行一个受限制的请求以符合`requests_per_second`。

- `failures`

  所有索引失败的数组。如果这是非空的，则请求由于这些故障而中止。请参阅`conflicts`如何防止版本冲突中止操作。

### 与Task API一起使用

您可以使用[Task API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)获取所有正在运行的逐个查询请求的状态 ：

```shell
curl -XGET 'localhost:9200/_tasks?detailed=true&actions=*byquery&pretty'
```

响应如下所示：

```
{
  "nodes" : {
    "r1A2WoRbTwKZ516z6NEs5A" : {
      "name" : "r1A2WoR",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "attributes" : {
        "testattr" : "test",
        "portsfile" : "true"
      },
      "tasks" : {
        "r1A2WoRbTwKZ516z6NEs5A:36619" : {
          "node" : "r1A2WoRbTwKZ516z6NEs5A",
          "id" : 36619,
          "type" : "transport",
          "action" : "indices:data/write/update/byquery",
          "status" : {    （1）
            "total" : 6154,
            "updated" : 3500,
            "created" : 0,
            "deleted" : 0,
            "batches" : 4,
            "version_conflicts" : 0,
            "noops" : 0,
            "retries": {
              "bulk": 0,
              "search": 0
            }
            "throttled_millis": 0
          },
          "description" : ""
        }
      }
    }
  }
}
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#CO20-1) | 该对象包含实际状态。这就像是与该`total`领域的重要补充json的响应。`total`是reindex预期执行的操作的总数。您可以通过添加估计的进展`updated`，`created`以及`deleted`多个领域。当他们的总和等于该`total`字段时，请求将完成。 |
| ---------------------------------------- | ---------------------------------------- |
|                                          |                                          |

使用任务ID，您可以直接查找任务：

```
curl -XGET 'localhost:9200/_tasks/taskId:1?pretty'
```

这个API的优点是它集成了`wait_for_completion=false` 透明地返回已完成任务的状态。如果任务完成`wait_for_completion=false`并被设置，则它将返回一个 `results`或一个`error`字段。这个功能的成本是`wait_for_completion=false`创建的文档 `.tasks/task/${taskId}`。删除该文件由您决定。

### 与Cancel Task API一起使用

任何通过查询更新可以使用[Task Cancel API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)：

```
curl -XPOST 'localhost:9200/_tasks/task_id:1/_cancel?pretty'
```

`task_id`在上述Task API被使用到。

取消应该很快，但可能需要几秒钟。上面的任务状态API将继续列出任务，直到它被唤醒以取消自己。

### Rethrottling

`requests_per_second`通过使用`_rethrottle`API 的查询，可以在正在运行的更新中更改值：

```
curl -XPOST 'localhost:9200/_update_by_query/task_id:1/_rethrottle?requests_per_second=-1&pretty'
```

在`task_id`可以使用上述任务的API被发现。

就像在`_update_by_query`API 上设置它`requests_per_second` 可以`-1`禁用节流或任何十进制数字一样，`1.7`或`12`扼杀到该级别。加快查询速度的重新生效会立即生效，但是在完成当前批次后，重新生成查询会减慢查询生效。这可以防止滚动超时。

### 切片

通过查询更新支持[切片滚动](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)来并行化更新过程。这种并行化可以提高效率，并提供一种方便的方式将请求分解成更小的部分。

#### 手动切片

通过为每个请求提供切片ID和切片总数手动切片逐个查询：

```
curl -XPOST 'localhost:9200/twitter/_update_by_query?pretty' -H 'Content-Type: application/json' -d'
{
  "slice": {
    "id": 0,
    "max": 2
  },
  "script": {
    "source": "ctx._source[\u0027extra\u0027] = \u0027test\u0027"
  }
}
'
curl -XPOST 'localhost:9200/twitter/_update_by_query?pretty' -H 'Content-Type: application/json' -d'
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "script": {
    "source": "ctx._source[\u0027extra\u0027] = \u0027test\u0027"
  }
}
'
```

您可以验证你的操作：

```
curl -XGET 'localhost:9200/_refresh?pretty'
curl -XPOST 'localhost:9200/twitter/_search?size=0&q=extra:test&filter_path=hits.total&pretty'
```

这响应一个`total`：

```
{
  "hits": {
    "total": 120
  }
}
```

#### 自动切片

你也可以让查询更新自动并行使用 [切片滚动](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)切片`_uid`。使用`slices`指定片使用的数字：

```
curl -XPOST 'localhost:9200/twitter/_update_by_query?refresh&slices=5&pretty' -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source[\u0027extra\u0027] = \u0027test\u0027"
  }
}
'
```

你也可以验证你的操作：

```
curl -XPOST 'localhost:9200/twitter/_search?size=0&q=extra:test&filter_path=hits.total&pretty'

```

这响应一个`total`：

```shell
{
  "hits": {
    "total": 120
  }
}
```

设置`slices`为`auto`让Elasticsearch选择要使用的片数。此设置将使用每个分片一个切片，达到一定的限制。如果有多个源索引，则根据分片数量最少的索引来选择切片的数量。

添加`slices`到`_update_by_query`刚刚自动化在上面的部分中使用的手工操作，创建子请求，这意味着它有一些诡异的现象：

- 您可以在[任务API中](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#docs-update-by-query-task-api)看到这些请求 。这些子请求是请求任务的“子”任务`slices`。
- 获取请求任务`slices`的状态只包含已完成切片的状态。
- 这些子请求可单独解决，如取消和重新节流。
- 重新调整请求`slices`将按比例重新调整未完成的子请求。
- 取消请求`slices`将取消每个子请求。
- 由于`slices`每个子请求的性质不会得到一个完美的文档的一部分。所有的文件将被解决，但一些切片可能比其他切片大。期待更大的切片有更均匀的分布。
- 参数，如`requests_per_second`与`size`上一个请求`slices` 的比例向每个子请求。结合上面关于分布不均匀的一点，你应该得出结论，使用 `size`与`slices`可能不会导致完全`size`文档是`_update_by_query`ed。
- 每个子请求都会得到一个稍微不同的源索引快照，虽然这些都是在大约同一时间进行的。

##### 选择切片的数量

如果自动切片，设置`slices`于`auto`会选择大多数指数的合理数量。如果您正在手动切片或者调整自动切片，请使用这些指南。

当`slices`索引号中的数量等于索引中的分片数时，查询性能是最有效的。如果这个数字很大（例如500），选择一个较低的数字太多`slices`会损害性能。设置 `slices`高于碎片数量通常不会提高效率并增加开销。

更新性能通过可用资源与片数进行线性扩展。

查询或更新性能是否在运行时占主要地位取决于重新编制索引的文档和集群资源。

### 选择一个新的属性

假设您创建了一个没有动态映射的索引，并填充了数据，然后添加了一个映射值来从数据中获取更多的字段：

```shell
curl -XPUT 'localhost:9200/test?pretty' -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "test": {
      "dynamic": false,   （1）
      "properties": {
        "text": {"type": "text"}
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/test/test?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "text": "words words",
  "flag": "bar"
}
'
curl -XPOST 'localhost:9200/test/test?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "text": "words words",
  "flag": "foo"
}
'
curl -XPUT 'localhost:9200/test/_mapping/test?pretty' -H 'Content-Type: application/json' -d' （2）
{
  "properties": {
    "text": {"type": "text"},
    "flag": {"type": "text", "analyzer": "keyword"}
  }
}
'
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#CO21-1) | 这意味着新字段不会被索引，只是存储在`_source`。             |
| ---------------------------------------- | ---------------------------------------- |
| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/2.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html#CO21-2) | 这将更新映射以添加新`flag`字段。要拿起新的领域，你必须重新索引所有的文件。 |

搜索数据将找不到任何内容：

```
curl -XPOST 'localhost:9200/test/_search?filter_path=hits.total&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "flag": "foo"
    }
  }
}
'

```



```
{
  "hits" : {
    "total" : 0
  }
}
```

但是你可以发出一个`_update_by_query`请求来获取新的映射：

```
curl -XPOST 'localhost:9200/test/_update_by_query?refresh&conflicts=proceed&pretty'
curl -XPOST 'localhost:9200/test/_search?filter_path=hits.total&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "flag": "foo"
    }
  }
}
'

```



```
{
  "hits" : {
    "total" : 1
  }
}
```

将字段添加到多字段时，可以做同样的事情。
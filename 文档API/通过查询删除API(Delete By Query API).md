## 通过查询删除API[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/docs/delete-by-query.asciidoc)

`_delete_by_query`最简单的用法就是对每个匹配查询的文档执行删除操作。下面是API实例：

```shell
curl -XPOST 'localhost:9200/twitter/_delete_by_query?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}
'
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html#CO17-1) | 查询必须`query`以与[Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html)相同的方式作为值传递给密钥。您也可以使用`q` 与搜索api相同的方式使用该参数。 |
| ---------------------------------------- | ---------------------------------------- |
|                                          |                                          |

这将返回像这样的东西：

```shell
{
  "took" : 147,
  "timed_out": false,
  "deleted": 119,
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
  "total": 119,
  "failures" : [ ]
}
```

`_delete_by_query`获取索引启动时的快照，并删除使用`internal`版本控制发现的内容。这意味着，如果文档在拍摄快照和处理删除请求之间发生更改，将会发生版本冲突。当版本匹配时，文档被删除。

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

由于`internal`版本控制不支持将值0作为有效的版本号，因此版本等于零的文档不能被 `_delete_by_query`删除，并且会使请求失败。

在`_delete_by_query`执行过程中，顺序执行多个搜索请求，以找到所有要删除的匹配文档。每找到一批文件，就执行相应的批量请求，删除所有这些文件。如果搜索或批量请求被拒绝，则`_delete_by_query` 依靠默认策略重试拒绝的请求（最多10次，指数退格）。达到最大重试限制会导致`_delete_by_query` 中止，并且所有故障都会`failures`在响应中返回。已经执行的删除操作仍然存在。换句话说，过程不回滚，只能中止。当第一个失败导致中止时，由失败的批量请求返回的所有失败都将返回`failures` 元件; 因此可能有相当多的失败实体。

如果你想计算版本冲突，而不是让他们中止，然后设置`conflicts=proceed`在网址或`"conflicts": "proceed"`请求正文。

回到API格式，你可以限制`_delete_by_query`到一个单一的类型。这只会`tweet`从`twitter`索引中删除文件：

```shell
curl -XPOST 'localhost:9200/twitter/tweet/_delete_by_query?conflicts=proceed&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'
```

与搜索API一样，也可以一次删除多个索引和多个类型的文档：

```shell
curl -XPOST 'localhost:9200/twitter,blog/tweet,post/_delete_by_query?pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'
```

如果您提供，`routing`那么路由被复制到滚动查询，将进程限制为匹配该路由值的分片：

```
curl -XPOST 'localhost:9200/twitter/_delete_by_query?routing=1&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "range" : {
        "age" : {
           "gte" : 10
        }
    }
  }
}
'
```

默认情况下`_delete_by_query`使用滚动批处理1000.您可以使用`scroll_size`URL参数更改批处理大小：

```
curl -XPOST 'localhost:9200/twitter/_delete_by_query?scroll_size=5000&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
'

```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-delete-by-query/5.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-delete-by-query/5.json)[ ]()

### URL 参数

除了标准的参数，如`pretty`，删除通过查询API也支持`refresh`，`wait_for_completion`，`wait_for_active_shards`，和`timeout`。

`refresh`一旦请求完成，发送将会刷新查询所涉及的所有碎片。这与Delete API的`refresh` 参数不同，这个参数只会导致接收到删除请求的分片被刷新。

如果请求包含，`wait_for_completion=false`那么Elasticsearch将执行一些预检检查，启动请求，然后返回一个`task` 可以与[Tasks API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html#docs-delete-by-query-task-api) 一起使用的取消或获取任务状态的请求。Elasticsearch也将创建这个任务的记录作为一个文件在`.tasks/task/${taskId}`。这是你的保留或删除，如你所见。当你完成它，删除它，所以Elasticsearch可以回收它使用的空间。

`wait_for_active_shards`在继续处理请求之前，控制一个分片的副本必须有效。详情请看[这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards) 。`timeout`控制每个写请求等待不可用碎片变得可用的时间。两个工作正是他们在 [批量API中的工作方式](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)。

`requests_per_second`可以被设置为任何正十进制数（`1.4`，`6`， `1000`等）和节流速率`_delete_by_query`通过填充每个批次由一等待时间发出的删除操作的批次。限制可以通过设置`requests_per_second`为禁用`-1`。

限制是通过在批次间等待来完成的，以便在`_delete_by_query`内部使用的滚动 可以被赋予一个考虑到填充的超时。填充时间是批量除以`requests_per_second`和写入时间之间的差值。默认情况下批量大小是`1000`，所以如果`requests_per_second`设置为`500`：

```shell
target_time = 1000 / 500 per second = 2 seconds
wait_time = target_time - write_time = 2 seconds - .5 seconds = 1.5 seconds   
```

由于批处理是作为单个`_bulk`请求发出的，因此大批量处理会导致Elasticsearch创建很多请求，然后等待一段时间再开始下一个处理。这是“bursty”而不是“smooth”。默认是`-1`。

### 响应正文（Response body）

JSON响应如下所示：

```
{
  "took" : 147,
  "timed_out": false,
  "total": 119,
  "deleted": 119,
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

  如果在查询执行删除期间执行的任何请求超时 ，则将此标志设置为`true`。

- `total`

  成功处理的文档数量。

- `deleted`

  已成功删除的文档数量。

- `batches`

  通过查询删除回滚的滚动响应的数量。

- `version_conflicts`

  查询命中删除的版本冲突数。

- `noops`

  通过查询删除该字段总是等于零。它只存在于通过查询删除，通过查询更新和reindex APIs返回具有相同结构的响应。

- `retries`

  通过查询删除尝试的重试次数。`bulk`是重试的批量操作的数量，`search`是重试的搜索操作的数量。

- `throttled_millis`

  请求睡眠符合的毫秒数`requests_per_second`。

- `requests_per_second`

  在查询删除过程中每秒有效执行的请求数。

- `throttled_until_millis`

  在查询响应删除时，该字段应始终等于零。它只有在使用[Task API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html#docs-delete-by-query-task-api)时才有意义，它表示下一次（从epoch开始以毫秒为单位）再次执行一个受限制的请求以符合`requests_per_second`。

- `failures`

  所有索引失败的数组。如果这是非空的，则请求由于这些故障而中止。请参阅`conflicts`如何防止版本冲突中止操作。

### 与Task API一起使用

您可以使用[Task API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)获取任何正在运行的删除查询请求的状态 ：

```
curl -XGET 'localhost:9200/_tasks?detailed=true&actions=*/delete/byquery&pretty'
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
          "action" : "indices:data/write/delete/byquery",
          "status" : {    
            "total" : 6154,
            "updated" : 0,
            "created" : 0,
            "deleted" : 3500,
            "batches" : 36,
            "version_conflicts" : 0,
            "noops" : 0,
            "retries": 0,
            "throttled_millis": 0
          },
          "description" : ""
        }
      }
    }
  }
}
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html#CO18-1) | 该对象包含实际状态。这就像是与该`total`字段的重要补充json的响应。`total`是reindex预期执行的操作的总数。您可以通过添加估计的进展`updated`，`created`以及`deleted`多个字段。当他们的总和等于该`total`字段时，请求将完成。 |
| ---------------------------------------- | ---------------------------------------- |
|                                          |                                          |

使用任务ID，您可以直接查找任务：

```shell
curl -XPOST 'localhost:9200/_tasks/task_id:1/_cancel?pretty'
```

这个API的优点是它集成了`wait_for_completion=false` 透明地返回已完成任务的状态。如果任务完成`wait_for_completion=false`并被设置，那么它会回来 `results`或一个`error`字段。这个功能的成本是`wait_for_completion=false`创建的文档 `.tasks/task/${taskId}`。删除该文件由您决定。

### 与取消任务API一起使用

任何通过查询删除都可以使用[任务取消API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)：

```
curl -XPOST 'localhost:9200/_tasks/task_id:1/_cancel?pretty'
```

`task_id`可以使用上述任务API中被使用过。

取消应该很快，但可能需要几秒钟。上面的任务状态API将继续列出任务，直到它被唤醒以取消自己。

### Rethrottling

`requests_per_second`通过使用`_rethrottle`API 的查询，可以在正在运行的删除中更改值：

```
curl -XPOST 'localhost:9200/_delete_by_query/task_id:1/_rethrottle?requests_per_second=-1&pretty'
```

`task_id`可以使用上述任务API被使用过。

就像在`_delete_by_query`API 上设置它`requests_per_second` 可以`-1`禁用节流或任何十进制数字一样，`1.7`或`12`扼杀到该级别。加快查询速度的重新生效会立即生效，但是在完成当前批次后，重新生成查询会减慢查询生效。这可以防止滚动超时。

### 切片（Slicing）

通过查询删除支持[切片滚动](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)以并行化删除过程。这种并行化可以提高效率，并提供一种方便的方式将请求分解成更小的部分。

#### 手动切片（Manually slicing）

通过为每个请求提供切片ID和切片总数，手动切片删除查询：

```
curl -XPOST 'localhost:9200/twitter/_delete_by_query?pretty' -H 'Content-Type: application/json' -d'
{
  "slice": {
    "id": 0,
    "max": 2
  },
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
'
curl -XPOST 'localhost:9200/twitter/_delete_by_query?pretty' -H 'Content-Type: application/json' -d'
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
'
```

您可以验证哪些方面适用于：

```
curl -XGET 'localhost:9200/_refresh?pretty'
curl -XPOST 'localhost:9200/twitter/_search?size=0&filter_path=hits.total&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
'
```

这生成这样一个的`total`：

```
{
  "hits": {
    "total": 0
  }
}
```

#### 自动切片

您也可以让删除查询自动并行使用 [切片滚动](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)切片`_uid`。使用`slices`指定片使用的数字：

```
curl -XPOST 'localhost:9200/twitter/_delete_by_query?refresh&slices=5&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
'
```

你也可以验证你的操作：

```
curl -XPOST 'localhost:9200/twitter/_search?size=0&filter_path=hits.total&pretty' -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
'
```

这生成一个`total`：

```
{
  "hits": {
    "total": 0
  }
}
```

设置`slices`为`auto`让Elasticsearch选择要使用的片数。此设置将使用每个分片一个切片，达到一定的限制。如果有多个源索引，则根据分片数量最少的索引来选择切片的数量。

添加`slices`到`_delete_by_query`刚刚自动化在上面的部分中使用的手工工艺，创建子请求，这意味着它有一些诡异的结果：

- 您可以在[Task API中](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html#docs-delete-by-query-task-api)看到这些请求 。这些子请求是请求任务的“子”任务`slices`。
- 获取请求任务`slices`的状态只包含已完成切片的状态。
- 这些子请求可单独解决，如取消和重新节流。
- 重新调整请求`slices`将按比例重新调整未完成的子请求。
- 取消请求`slices`将取消每个子请求。
- 由于`slices`每个子请求的性质不会得到一个完美的文档的一部分。所有的文件将被解决，但一些切片可能比其他切片大。期待更大的切片有更均匀的分布。
- 参数，如`requests_per_second`与`size`上一个请求`slices` 的比例向每个子请求。结合上面关于分布不均匀的观点，你应该得出结论，使用 `size`与`slices`可能不会导致完全`size`文档是`_delete_by_query`ed。
- 每个子请求都会得到一个稍微不同的源索引快照，虽然这些都是在大约同一时间进行的。

##### 选择切片的数量

如果自动切片，设置`slices`于`auto`会选择大多数指数的合理数量。如果您正在手动切片或者调整自动切片，请使用这些指南。

当`slices`索引号中的数量等于索引中的分片数时，查询性能是最有效的。如果这个数字很大（例如500），选择一个较低的数字太多`slices`会损害性能。设置 `slices`高于碎片数量通常不会提高效率并增加开销。

删除性能在可用资源和切片数量之间线性变化。

查询或删除性能在运行时间中占主要地位取决于被重新索引的文档和集群资源。
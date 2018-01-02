## Reindex API

![重要](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/important.png)

重建索引不会尝试设置目标索引。它不会复制源索引的设置。您应该在运行`_reindex`操作之前设置目标索引，包括设置映射，碎片计数，副本等。

最基本的形式`_reindex`就是将文档从一个索引拷贝到另一个索引。这会将`twitter`索引中的文档复制到`new_twitter`索引中：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
'
```

这将返回像这样的东西：

```shell
{
  "took" : 147,
  "timed_out": false,
  "created": 120,
  "updated": 0,
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

就像[`_update_by_query`](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html)，`_reindex`得到源索引的快照，但它的目标必须是一个**不同的**索引，以便版本冲突是不可能的。该`dest`元素可以像索引API一样配置，以控制乐观并发控制。只要忽略 `version_type`（如上所述）或将其设置为`internal`将导致Elasticsearch盲目地将文档转储到目标文件中，覆盖所有碰巧具有相同类型和id的文件：

```shell
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "internal"
  }
}
'
```

设置`version_type`为`external`将导致Elasticsearch保留 `version`来源，创建缺少的任何文档，并更新目标索引中具有较早版本的任何文档，而不是源索引中的文档：

```shell
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  }
}
'
```

设置`op_type`到`create`会导致`_reindex`在目标指数只会造成文件丢失。所有现有的文档都会导致版本冲突：

```shell
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
'
```

默认情况下，版本冲突会中止这个`_reindex`过程，但是你可以通过`"conflicts": "proceed"`请求体中的设置对它们进行计数：

```shell
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "conflicts": "proceed",
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "op_type": "create"
  }
}
'
```

您可以通过添加类型`source`或通过添加查询来限制文档。这只会复制`tweet`的由制造`kimchy`到`new_twitter`：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter",
    "type": "tweet",
    "query": {
      "term": {
        "user": "kimchy"
      }
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
'

```



`index`而`type`在`source`既可以是列表，让您可以在一个请求中实现大量的资源复制。这将从twitter和blog索引中复制`tweet`和 `post`类型的文档。它会包括在twitter索引中的post类型和在blog索引中的tweet类型。如果你想更具体，你需要使用query。它不会处理ID冲突。目标指标将保持有效，但是由于迭代顺序没有被很好地定义，所以预测哪个文档将会生存并不容易。

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": ["twitter", "blog"],
    "type": ["tweet", "post"]
  },
  "dest": {
    "index": "all_together"
  }
}
'
```

也可以通过设置来限制处理文档的数量 `size`。这只会将单个文件复制`twitter`到 `new_twitter`：

```shell
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 1,
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
'

```



如果你想要twitter索引中的一组特定文件，你需要排序。排序使滚动效率降低，但在某些情况下，这是值得的。如果可能的话，更喜欢选择性的查询`size`和`sort`。这将复制10000个文件`twitter`到`new_twitter`：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 10000,
  "source": {
    "index": "twitter",
    "sort": { "date": "desc" }
  },
  "dest": {
    "index": "new_twitter"
  }
}
'

```



该`source`部分支持[search request](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-body.html)支持的所有元素 。例如，只有来自原始文档的字段子集可以使用源过滤重新编制索引，如下所示：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter",
    "_source": ["user", "tweet"]
  },
  "dest": {
    "index": "new_twitter"
  }
}
'

```



就像`_update_by_query`，`_reindex`支持修改文档的脚本。不同的是`_update_by_query`，脚本被允许修改文档的元数据。此示例bumps了源文档的版本：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter",
    "version_type": "external"
  },
  "script": {
    "source": "if (ctx._source.foo == \u0027bar\u0027) {ctx._version++; ctx._source.remove(\u0027foo\u0027)}",
    "lang": "painless"
  }
}
'
 
```

就像在`_update_by_query`，您可以设置`ctx.op`更改在目标索引上执行的操作：

- `noop`

  设置`ctx.op = "noop"`如果你的脚本决定，该文件没有在目的地标识被编入索引。这个没有任何操作会`noop`在[响应主体](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-response-body)的柜台上报告。

- `delete`

  设置`ctx.op = "delete"`如果你的脚本决定，该文件必须从目标索引中删除。删除将`deleted`在response body中有响应。

设置`ctx.op`为其他任何东西都是错误的。设置任何其他字段`ctx`是一个错误。

想想可能性！只要小心！拥有强大的力量。你可以改变：

- `_id`
- `_type`
- `_index`
- `_version`
- `_routing`

设置`_version`到`null`或从清除它`ctx`的地图就像是在一个索引请求不发送版本。无论目标版本或`_reindex`请求中使用的版本类型如何，都将导致该文档被覆盖在目标索引中。

默认情况下，如果`_reindex`使用路由查看文档，则路由将被保留，除非脚本更改了该路由。您可以设置`routing`在 `dest`改变这个请求：

- `keep`

  将发送给每个匹配的批量请求的路由设置为匹配的路由。默认。

- `discard`

  将为每个匹配发送的批量请求的路由设置为空。

- `=<some text>`

  将为每个匹配发送的批量请求的路由设置为`=`。

例如，可以使用以下请求将`source`具有公司名称`cat`的`dest`索引中的所有文档复制到路由设置为cat的索引中。

```shell
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "source",
    "query": {
      "match": {
        "company": "cat"
      }
    }
  },
  "dest": {
    "index": "dest",
    "routing": "=cat"
  }
}
'
```

默认情况下`_reindex`使用滚动批次为1000.您可以使用元素中的`size`字段更改批量大小`source`：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "source",
    "size": 100
  },
  "dest": {
    "index": "dest",
    "routing": "=cat"
  }
}
'

```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-reindex/13.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-reindex/13.json)[ ]()

Reindex也可以通过指定一个像这样使用[摄取节点](https://www.elastic.co/guide/en/elasticsearch/reference/current/ingest.html)功能 `pipeline`：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "source"
  },
  "dest": {
    "index": "dest",
    "pipeline": "some_ingest_pipeline"
  }
}
'
```

### 从远程重建索引

Reindex支持从远程Elasticsearch集群重建索引：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200",
      "username": "user",
      "password": "pass"
    },
    "index": "source",
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}
'
```

该`host`参数必须包含一个scheme，主机和端口（例如 `https://otherhost:9200`）。的`username`和`password`参数是可选的，当它们存在时重新索引将连接到使用基本认证远程Elasticsearch节点。`https`使用基本身份验证时一定要使用，否则密码将以纯文本形式发送。

远程主机必须使用该`reindex.remote.whitelist`属性在elasticsearch.yaml中显式列入白名单 。它可以设置为逗号分隔的允许的远程`host`和`port`组合列表（例如 `otherhost:9200, another:9200, 127.0.10.*:9200, localhost:*`）。白名单忽略Scheme - 仅使用主机和端口。

这个功能应该适用于你可能找到的任何版本的Elasticsearch远程集群。这应该允许您从任何版本的Elasticsearch升级到当前版本，方法是从旧版本的集群重新索引。

为了使查询发送到旧版本的Elasticsearch，`query`参数直接发送到远程主机而无需验证或修改。

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

从远程群集重新分派不支持 [手动](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-manual-slice)或 [自动分片](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-automatic-slice)。

从远程服务器重新编译使用默认最大大小为100MB的堆缓冲区。如果远程索引包含非常大的文档，则需要使用较小的批量。下面的例子设置了`10` 非常非常小的批量大小。

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200"
    },
    "index": "source",
    "size": 10,
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}
'

```



也可以在远程连接上设置套接字读取超时`socket_timeout`字段和连接超时 `connect_timeout`字段。两者都默认为三十秒。此示例将套接字读取超时设置为一分钟，将连接超时设置为十秒：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "remote": {
      "host": "http://otherhost:9200",
      "socket_timeout": "1m",
      "connect_timeout": "10s"
    },
    "index": "source",
    "query": {
      "match": {
        "test": "data"
      }
    }
  },
  "dest": {
    "index": "dest"
  }
}
'
  
  
```

### URL参数

除了标准参数等`pretty`，重新索引API还支持`refresh`，`wait_for_completion`，`wait_for_active_shards`，`timeout`，和 `requests_per_second`。

发送`refresh`url参数将导致请求写入的所有索引被刷新。这与Index API的`refresh` 参数不同，后者只会导致接收新数据的碎片被刷新。

如果请求包含，`wait_for_completion=false`那么Elasticsearch将执行一些预检检查，启动请求，然后返回一个`task` 可以与[Tasks API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-task-api) 一起使用的取消或获取任务状态的请求。Elasticsearch也将创建这个任务的记录作为一个文件在`.tasks/task/${taskId}`。这是你的保留或删除，如你所见。当你完成它，删除它，所以Elasticsearch可以回收它使用的空间。

`wait_for_active_shards`在继续重新索引之前，控制分片的数量必须有效。详情请看[这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards) 。`timeout`控制每个写请求等待不可用碎片变得可用的时间。两个工作正是他们在 [批量API中的工作方式](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)。

`requests_per_second`可以被设置为任何正十进制数（`1.4`，`6`， `1000`等），并在节流其中通过填充每个批次由一等待时间重新索引索引操作的问题批次速率。限制可以通过设置`requests_per_second`为禁用`-1`。

限制是通过在批次之间等待来完成的，这样重新索引在内部使用的滚动可以被赋予一个考虑到填充的超时。填充时间是批量除以`requests_per_second`和写入时间之间的差值 。默认情况下批量大小是 `1000`，所以如果`requests_per_second`设置为`500`：

```
target_time = 1000 / 500 per second = 2 seconds
wait_time = target_time - write_time = 2 seconds - .5 seconds = 1.5 seconds
```

由于批处理是作为单个`_bulk`请求发出的，因此大批量处理会导致Elasticsearch创建很多请求，然后等待一段时间再开始下一个处理。这是“突发”而不是“流畅”。默认是`-1`。

### 响应正文

JSON响应如下所示：

```
{
  "took": 639,
  "timed_out": false,
  "total": 5,
  "updated": 0,
  "created": 5,
  "deleted": 0,
  "batches": 1,
  "noops": 0,
  "version_conflicts": 2,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": 1,
  "throttled_until_millis": 0,
  "failures": [ ]
}
```

- `took`

  整个操作从开始到结束的毫秒数。

- `timed_out`

  如果重新索引期间执行的任何请求超时 ，则将此标志设置为`true`。

- `total`

  成功处理的文档数量。

- `updated`

  成功更新的文档数量。

- `created`

  已成功创建的文档数量。

- `deleted`

  已成功删除的文档数量。

- `batches`

  滚动响应的数量由reindex拉回。

- `noops`

  由于用于reindex的脚本为其返回`noop`值而被忽略的文档数量`ctx.op`。

- `version_conflicts`

  重新索引发生的版本冲突次数。

- `retries`

  reindex尝试重试的次数。`bulk`是重试的批量操作`search`的数量，是重试的搜索操作的数量。

- `throttled_millis`

  请求睡眠符合的毫秒数`requests_per_second`。

- `requests_per_second`

  重建索引期间有效执行的每秒请求数。

- `throttled_until_millis`

  在查询响应删除时，该字段应始终等于零。它只有在使用[Task API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-task-api)时才有意义，它表示下一次（从epoch开始以毫秒为单位）再次执行一个受限制的请求以符合`requests_per_second`。

- `failures`

  所有索引失败的数组。如果这是非空的，则请求由于这些故障而中止。请参阅`conflicts`如何防止版本冲突中止操作。

### 与Task API一起使用

您可以使用[Task API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)获取所有正在运行的重新索引请求的状态 ：

```
curl -XGET 'localhost:9200/_tasks?detailed=true&actions=*reindex&pretty'
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
          "action" : "indices:data/write/reindex",
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
            },
            "throttled_millis": 0
          },
          "description" : ""
        }
      }
    }
  }
}
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#CO23-1) | 该对象包含实际状态。这就像是与该`total`领域的重要补充json的响应。`total`是reindex预期执行的操作的总数。您可以通过添加估计的进展`updated`，`created`以及`deleted`多个字段。当他们的总和等于该`total`字段时，请求将完成。 |
| ---------------------------------------- | ---------------------------------------- |
|                                          |                                          |

使用任务ID，您可以直接查找任务：

```
curl -XGET 'localhost:9200/_tasks/taskId:1?pretty'

```



这个API的优点是它集成了`wait_for_completion=false` 透明地返回已完成任务的状态。如果任务完成`wait_for_completion=false`并被设置，则它将返回一个 `results`或一个`error`字段。这个功能的成本是`wait_for_completion=false`创建的文档 `.tasks/task/${taskId}`。删除该文件由您决定。

### 与Cancel Task API一起使用

使用[任务取消API](https://www.elastic.co/guide/en/elasticsearch/reference/current/tasks.html)可以取消任何重建索引：

```
curl -XPOST 'localhost:9200/_tasks/task_id:1/_cancel?pretty'

```



在`task_id`可以使用上述任务的API被发现。

取消应该很快，但可能需要几秒钟。上面的任务状态API将继续列出任务，直到它被唤醒以取消自己。

### Rethrottling

`requests_per_second`可以使用`_rethrottle`API 在正在运行的reindex上更改值：

```
curl -XPOST 'localhost:9200/_reindex/task_id:1/_rethrottle?requests_per_second=-1&pretty'
```

`task_id`在Task API已经被使用过。

就像在`_reindex`API 上设置它`requests_per_second` 可以`-1`禁用节流或任何十进制数字一样，`1.7`或`12`扼杀到该级别。加快查询速度的重新生效会立即生效，但是在完成当前批次后，重新生成查询会减慢查询生效。这可以防止滚动超时。

### 重建索引时更改字段的名称

`_reindex`可以用来建立一个索引与重命名字段的副本。假设你创建一个包含如下所示的文档的索引：

```
curl -XPOST 'localhost:9200/test/test/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "text": "words words",
  "flag": "foo"
}
'

```



但是你不喜欢这个名字，`flag`并且想替换它`tag`。 `_reindex`可以为您创建其他索引：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "test"
  },
  "dest": {
    "index": "test2"
  },
  "script": {
    "source": "ctx._source.tag = ctx._source.remove(\"flag\")"
  }
}
'

```



现在你可以得到新的文件：

```
curl -XGET 'localhost:9200/test2/test/1?pretty'

```



它会看起来像：

```
{
  "found": true,
  "_id": "1",
  "_index": "test2",
  "_type": "test",
  "_version": 1,
  "_source": {
    "text": "words words",
    "tag": "foo"
  }
}
```

或者你可以搜索`tag`或任何你想要的。

### 切片

Reindex支持[Sliced Scroll](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)来并行化重建索引过程。这种并行化可以提高效率，并提供一种方便的方式将请求分解成更小的部分。

#### 手动切片

通过为每个请求提供切片ID和切片总数手动切片重新索引请求：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter",
    "slice": {
      "id": 0,
      "max": 2
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
'
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter",
    "slice": {
      "id": 1,
      "max": 2
    }
  },
  "dest": {
    "index": "new_twitter"
  }
}
'
```

您可以验证哪些方面适用于：

```
curl -XGET 'localhost:9200/_refresh?pretty'
curl -XPOST 'localhost:9200/new_twitter/_search?size=0&filter_path=hits.total&pretty'
```



这生成一个`total`：

```
{
  "hits": {
    "total": 120
  }
} 
```

#### 自动切片

你也可以让reindex自动并行使用[切片滚动](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-scroll.html#sliced-scroll)切片`_uid`。使用`slices`指定片使用的数字：

```
curl -XPOST 'localhost:9200/_reindex?slices=5&refresh&pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "twitter"
  },
  "dest": {
    "index": "new_twitter"
  }
}
'
```



你也可以验证你的操作：

```
curl -XPOST 'localhost:9200/new_twitter/_search?size=0&filter_path=hits.total&pretty'、
```

这将生成一个`total`：

```
{
  "hits": {
    "total": 120
  }
}
```

设置`slices`为`auto`让Elasticsearch选择要使用的片数。此设置将使用每个分片一个切片，达到一定的限制。如果有多个源索引，则根据分片数量最少的索引来选择切片的数量。

添加`slices`到`_reindex`刚刚自动化在上面的部分中使用的手工工艺，创建子请求，这意味着它有一些诡异的现象：

- 您可以在[任务API中](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-task-api)看到这些请求。这些子请求是请求任务的“子”任务`slices`。
- 获取请求任务`slices`的状态只包含已完成切片的状态。
- 这些子请求可单独解决，如取消和重新节流。
- 重新调整请求`slices`将按比例重新调整未完成的子请求。
- 取消请求`slices`将取消每个子请求。
- 由于`slices`每个子请求的性质不会得到一个完美的文档的一部分。所有的文件将被解决，但一些切片可能比其他切片大。期待更大的切片有更均匀的分布。
- 参数，如`requests_per_second`与`size`上一个请求`slices` 的比例向每个子请求。结合上面关于分布不均匀的观点，你应该得出结论，使用 `size`与`slices`可能不会导致完全的`size`文件`_reindex`ed。
- 每个子请求都会得到一个稍微不同的源索引快照，虽然这些都是在大约同一时间进行的。

##### 选择切片的数量

如果自动切片，设置`slices`于`auto`会选择大多数指数的合理数量。如果您正在手动切片或者调整自动切片，请使用这些指南。

当`slices`索引号中的数量等于索引中的分片数时，查询性能是最有效的。如果这个数字很大（例如500），选择一个较低的数字太多`slices`会损害性能。设置 `slices`高于碎片数量通常不会提高效率并增加开销。

索引性能在可用资源和切片数量之间线性扩展。

查询或索引性能是否在运行时占主要地位取决于重新编制索引的文档和集群资源。

### Reindex daily indices

您可以`_reindex`结合使用“ [painless”](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-painless.html) 重新编制日常索引，以将新模板应用于现有文档。

假设您的索引包含以下文档：

```
curl -XPUT 'localhost:9200/metricbeat-2016.05.30/beat/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{"system.cpu.idle.pct": 0.908}
'
curl -XPUT 'localhost:9200/metricbeat-2016.05.31/beat/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{"system.cpu.idle.pct": 0.105}
'
```



`metricbeat-*`索引的新模板已经加载到Elasticsearch中，但是它只适用于新创建的索引。Painless可用于重新索引现有文档并应用新模板。

下面的脚本从索引名称中提取日期，并创建一个`-1`附加的新索引。所有数据`metricbeat-2016.05.31`将被重新编入`metricbeat-2016.05.31-1`。

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "metricbeat-*"
  },
  "dest": {
    "index": "metricbeat"
  },
  "script": {
    "lang": "painless",
    "source": "ctx._index = \u0027metricbeat-\u0027 + (ctx._index.substring(\u0027metricbeat-\u0027.length(), ctx._index.length())) + \u0027-1\u0027"
  }
}
'

```



以前的度量标准索引中的所有文档现在都可以在`*-1`索引中找到。

```
curl -XGET 'localhost:9200/metricbeat-2016.05.30-1/beat/1?pretty'
curl -XGET 'localhost:9200/metricbeat-2016.05.31-1/beat/1?pretty'
```

以前的方法也可以结合使用，[改变一个字段的名称，](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex-change-name) 只将现有的数据加载到新的索引，但如果需要也可以重命名字段。

### 提取索引的随机子集

可以使用Reindex提取索引的随机子集进行测试：

```
curl -XPOST 'localhost:9200/_reindex?pretty' -H 'Content-Type: application/json' -d'
{
  "size": 10,
  "source": {
    "index": "twitter",
    "query": {
      "function_score" : {
        "query" : { "match_all": {} },
        "random_score" : {}
      }
    },
    "sort": "_score"    
  },
  "dest": {
    "index": "random_twitter"
  }
}
' 
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#CO24-1) | REINDEX默认为通过`_doc`排序,所以除非你重写`_score`排序,否则`random_score`不会有任何效果。 |
| ---------------------------------------- | ---------------------------------------- |
|                                          |                                          |
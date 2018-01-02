## 更新API

更新API允许根据提供的脚本更新文档。操作从索引获取文档（与分片并置），运行脚本（使用可选的脚本语言和参数），并索引结果（也允许删除或忽略操作）。它使用版本控制来确保在“get”和“reindex”期间没有更新。

请注意，此操作仍然意味着文档的完全重新索引，它只是消除了一些网络往返，并减少了获取和索引之间的版本冲突的机会。该`_source`字段需要启用此功能才能正常工作。

例如，让我们索引一个简单的文档：

```shell
curl -XPUT 'localhost:9200/test/type1/1?pretty' -H 'Content-Type: application/json' -d'
{
    "counter" : 1,
    "tags" : ["red"]
}
'
```

### 脚本更新

现在，我们可以执行一个脚本来增加计数器：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    }
}
'
```

我们可以添加一个标签到标签列表（注意，如果标签存在，它仍然会添加它，因为它是一个列表）：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "script" : {
        "source": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params" : {
            "tag" : "blue"
        }
    }
}
'
```

此外`_source`，以下变量通过提供`ctx`映射：`_index`，`_type`，`_id`，`_version`，`_routing` 和`_now`（当前的时间戳）。

我们也可以在文档中添加一个新字段：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "script" : "ctx._source.new_field = \u0027value_of_new_field\u0027"
}
'
```

或从文档中删除一个字段：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "script" : "ctx._source.remove(\u0027new_field\u0027)"
}
'
```

而且，我们甚至可以改变执行的操作。如果该`tags`字段包含该示例，则删除该文档`green`，否则它将不执行任何操作（`noop`）：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "script" : {
        "source": "if (ctx._source.tags.contains(params.tag)) { ctx.op = \u0027delete\u0027 } else { ctx.op = \u0027none\u0027 }",
        "lang": "painless",
        "params" : {
            "tag" : "green"
        }
    }
}
'
```

### 使用部分文档更新

更新API还支持传递将被合并到现有文档中的部分文档（简单的递归合并，对象的内部合并，替换核心“key/value”和数组）。例如：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "doc" : {
        "name" : "new_name"
    }
}
'
```

如果同时`doc`和`script`指定，然后`doc`被忽略。最好的办法是将你的部分文档的字段对放在脚本本身中。

### 检测noop掉的更新（Detecting noop updates）

如果`doc`被指定，则其值与现有的值合并`_source`。默认情况下，不会更改任何内容的更新会检测到它们不会更改任何内容，并返回“result”：“noop”，如下所示：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "doc" : {
        "name" : "new_name"
    }
}
'
```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-update/8.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/docs-update/8.json)[ ]()

如果`name`是`new_name`在发送请求之前，那么整个更新请求将被忽略。如果请求被忽略`result`，则响应中的元素将返回`noop`。

```shell
{
   "_shards": {
        "total": 0,
        "successful": 0,
        "failed": 0
   },
   "_index": "test",
   "_type": "type1",
   "_id": "1",
   "_version": 6,
   "result": "noop"
}
```

您可以通过设置“detect_noop”：false来禁用此行为：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "doc" : {
        "name" : "new_name"
    },
    "detect_noop": false
}
'
```

### Upserts

如果文档不存在，则`upsert`元素的内容将作为新文档插入。如果文档确实存在，那么 `script`将会执行：

```shell
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "script" : {
        "source": "ctx._source.counter += params.count",
        "lang": "painless",
        "params" : {
            "count" : 4
        }
    },
    "upsert" : {
        "counter" : 1
    }
}
'
```

#### `scripted_upsert`

如果你希望你的脚本运行，而不管这个文档是否存在 - 也就是脚本处理初始化文档而不是 `upsert`元素 - 然后设置`scripted_upsert`为`true`：

```
curl -XPOST 'localhost:9200/sessions/session/dh3sgudg8gsrgl/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "scripted_upsert":true,
    "script" : {
        "id": "my_web_session_summariser",
        "params" : {
            "pageViewEvent" : {
                "url":"foo.com/bar",
                "response":404,
                "time":"2014-01-01 12:32"
            }
        }
    },
    "upsert" : {}
}
'
```

#### `doc_as_upsert`

而不是发送部分`doc`加`upsert`文档，设置 `doc_as_upsert`为`true`将使用的内容`doc`作为`upsert` 值：

```
curl -XPOST 'localhost:9200/test/type1/1/_update?pretty' -H 'Content-Type: application/json' -d'
{
    "doc" : {
        "name" : "new_name"
    },
    "doc_as_upsert" : true
}
'
```

### 参数

更新操作支持以下查询字符串参数：

| `retry_on_conflict`        | 在更新的获取和索引阶段之间，另一个进程可能已经更新了同一个文档。默认情况下，更新将失败并出现版本冲突异常。该`retry_on_conflict` 参数控制在最后抛出异常之前重试更新的次数。 |
| -------------------------- | ---------------------------------------- |
| `routing`                  | 路由用于将更新请求路由到右侧分片，并在正在更新的文档不存在时设置upsert请求的路由。不能用于更新现有文档的路由。 |
| `timeout`                  | 超时等待碎片变为可用。                              |
| `wait_for_active_shards`   | 在进行更新操作之前，需要激活的分片副本的数量。详情请看[这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)。 |
| `refresh`                  | 控制此请求所做的更改对搜索是否可见。看 [*?refresh*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)。 |
| `_source`                  | 允许控制是否以及如何在响应中返回更新的源代码。默认情况下，更新的源不会被返回。见[`source filtering`](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-request-source-filtering.html)细节。 |
| `version` ＆ `version_type` | 更新API在内部使用Elasticsearch的版本控制支持来确保文档在更新期间不会更改。您可以使用该`version` 参数来指定文档只有在其版本与指定的版本匹配时才能更新。通过设置版本类型，`force`您可以在更新后强制更新文档的新版本（小心使用！`force` 并不保证文档没有更改）。 |

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

### 更新API不支持外部版本控制

更新API不支持外部版本（版本类型`external`＆`external_gte`），因为它会导致Elasticsearch版本号与外部系统不同步。改用 [`index`API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)。
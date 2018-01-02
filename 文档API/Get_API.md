## Get API

get API允许从索引中获取一个JSON文档。以下示例将从名为twitter的索引中获取名为tweet的类型的JSON文档，其ID为0：

```
curl -XGET 'localhost:9200/twitter/tweet/0?pretty'
```

上面得到的结果是：

```
{
    "_index" : "twitter",
    "_type" : "tweet",
    "_id" : "0",
    "_version" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

上述结果包括`_index`，`_type`，`_id`和`_version` 我们希望检索，包括实际文档的`_source` 文档，如果可以发现（如由指示`found` 字段中响应）。

API还允许使用以下方式检查文档是否存在 `HEAD`：

```
curl -XHEAD 'localhost:9200/twitter/tweet/0?pretty'
```

### 实时（Realtime）

默认情况下，get API是实时的，并且不受索引刷新率的影响（当数据对于搜索可见时）。如果文档已更新但尚未刷新，get API将就地发出刷新调用以使文档可见。这也会使上次刷新后其他文档发生变化。为了禁用实时GET，可以将`realtime`参数设置为`false`。

### 源过滤（Source filtering）

默认情况下，get操作返回`_source`字段的内容，除非已经使用该`stored_fields`参数或者该`_source`字段被禁用。您可以`_source`使用`_source`参数关闭检索：

```shell
curl -XGET 'localhost:9200/twitter/tweet/0?_source=false&pretty'
```

如果您只需要完整的一个或两个字段，则`_source`可以使用`_source_include` &`_source_exclude`参数来包含或过滤出所需的部分。这对大型文档特别有用，在这种文档中部分检索可以节省网络开销。两个参数都采用逗号分隔的字段列表或通配符表达式。例：

```shell
curl -XGET 'localhost:9200/twitter/tweet/0?_source_include=*.id&_source_exclude=entities&pretty'
```

如果您只想指定包含，则可以使用较短的表示法：

```shell
curl -XGET 'localhost:9200/twitter/tweet/0?_source=*.id,retweeted&pretty'
```

### 存储的字段(Stored Fields)

get操作允许指定一组存储的字段，这些字段将通过传递`stored_fields`参数来返回。如果请求的字段没有被存储，它们将被忽略。考虑下面的映射：

```shell
curl -XPUT 'localhost:9200/twitter?pretty' -H 'Content-Type: application/json' -d'
{
   "mappings": {
      "tweet": {
         "properties": {
            "counter": {
               "type": "integer",
               "store": false
            },
            "tags": {
               "type": "keyword",
               "store": true
            }
         }
      }
   }
}
'
```

现在我们可以添加一个document：

```shell
curl -XPUT 'localhost:9200/twitter?pretty' -H 'Content-Type: application/json' -d'
{
   "mappings": {
      "tweet": {
         "properties": {
            "counter": {
               "type": "integer",
               "store": false
            },
            "tags": {
               "type": "keyword",
               "store": true
            }
         }
      }
   }
}
'
```

1. 并尝试检索它：

```shell
curl -XGET 'localhost:9200/twitter/tweet/1?stored_fields=tags,counter&pretty'
```

上面得到的结果是：

```shell
{
   "_index": "twitter",
   "_type": "tweet",
   "_id": "1",
   "_version": 1,
   "found": true,
   "fields": {
      "tags": [
         "red"
      ]
   }
}
```

从文档中获取的字段值总是以数组形式返回。由于该`counter`字段没有被存储，get请求只是在试图获取时忽略它`stored_fields.`

也可以像字段一样检索元数据字段`_routing`：

```shell
curl -XPUT 'localhost:9200/twitter/tweet/2?routing=user1&pretty' -H 'Content-Type: application/json' -d'
{
    "counter" : 1,
    "tags" : ["white"]
}
'
```

```shell
curl -XGET 'localhost:9200/twitter/tweet/2?routing=user1&stored_fields=tags,counter&pretty'
```

上面得到的结果是：

```shell
{
   "_index": "twitter",
   "_type": "tweet",
   "_id": "2",
   "_version": 1,
   "_routing": "user1",
   "found": true,
   "fields": {
      "tags": [
         "white"
      ]
   }
} 
```

也只有leaf fields可以通过该`stored_field`选项返回。所以对象字段不能被返回，这样的请求将失败。

### 直接获取`_source`

使用`/{index}/{type}/{id}/_source`接口来获取`_source`文档的字段，而不需要获取其他任何额外的内容。例如：

```shell
curl -XGET 'localhost:9200/twitter/tweet/1/_source?pretty'
```

您也可以使用相同的源过滤参数来控制`_source`将返回哪些部分：

```shell
curl -XGET 'localhost:9200/twitter/tweet/1/_source?_source_include=*.id&_source_exclude=entities&pretty'
```

请注意，_source接口还有一个HEAD变体，用于高效地测试文档_source的存在。如果现有文档在[映射中](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html)被禁用，则该文档将不会有_source 。

```
curl -XHEAD 'localhost:9200/twitter/tweet/1/_source?pretty'
```

### 路由

当使用控制路由的能力进行索引时，为了获得文档，还应该提供路由值。例如：

```
curl -XGET 'localhost:9200/twitter/tweet/2?routing=user1&pretty'
```

以上将得到id为2的tweet，但会根据用户进行路由。请注意，如果没有正确的路由，发出一个get将导致文件不被获取。

### 偏好（Preference）

控制`preference`可以实现指定由哪个分片副本执行获取请求。默认情况下，操作在碎片副本之间是随机的。

该`preference`可设置为：

- `_primary`

  操作只会在主碎片上执行。

- `_local`

  如果可能，该操作将优选在本地分配的碎片上执行。

- 自定义（字符串）值

  自定义值将用于保证相同的自定义值将使用相同的分片。当在不同的刷新状态下点击不同的碎片时，这可以帮助“jumping value”。示例值可以是Web会话标识或用户名。

### 刷新（Refresh）

该`refresh`参数可以设置为`true`，那么在get操作之前刷新相关分片并使其可搜索。设置它`true`应该经过仔细的思考和验证后才能完成，这不会给系统造成沉重的负担（并且会降低索引）。

### 分布式（Distributed）

get操作被哈希成一个特定的分片ID。然后它被重定向到该分片ID中的一个副本并返回结果。副本是该分片ID组中的主分片及其副本。这意味着我们将拥有越多的副本，我们将拥有更好的GET扩展。

### 版本控制支持（Versioning support）

`version`只有当其当前版本等于指定的文档时，才可以使用该参数来检索文档。除了`FORCE`总是检索文档的版本类型之外，所有版本类型的行为都是相同的。请注意，`FORCE`版本类型已被弃用。

在内部，Elasticsearch将旧文档标记为已删除，并添加了一个全新的文档。旧版本的文档不会立即消失，尽管您无法访问它。随着您继续索引更多数据，Elasticsearch将在后台清理已删除的文档。
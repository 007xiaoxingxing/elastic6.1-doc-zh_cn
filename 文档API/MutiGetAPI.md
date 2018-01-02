## Multi Get API

Multi GET API允许基于索引，type（可选）和id（以及可能的路由）获取多个文档。响应包括一个`docs` 包含所有获取文档的数组，每个元素的结构与[get](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html) API 提供的文档相似。这里是一个例子：

```
curl -XGET 'localhost:9200/_mget?pretty' -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}
'
```



的`mget`接口也可以针对索引（在这种情况下它不能在体内所需的）中使用：

```
curl -XGET 'localhost:9200/test/_mget?pretty' -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_type" : "type",
            "_id" : "1"
        },
        {
            "_type" : "type",
            "_id" : "2"
        }
    ]
}
'
```

并输入：

```
curl -XGET 'localhost:9200/test/type/_mget?pretty' -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_id" : "1"
        },
        {
            "_id" : "2"
        }
    ]
}
'
```

在这种情况下，`ids`元素可以直接用来简化请求：

```
curl -XGET 'localhost:9200/test/type/_mget?pretty' -H 'Content-Type: application/json' -d'
{
    "ids" : ["1", "2"]
}
'
```

### 可选类型

mget API允许`_type`是可选的。将其设置为`_all`或保留为空，以便在所有类型中获取与该ID匹配的第一个文档。

如果您没有设置类型，并且有许多文档共享相同的文档`_id`，则最终只会获得第一个匹配的文档。

例如，如果在typeA和typeB中有一个文档1，那么以下请求将只给出两次相同的文档：

```
curl -XGET 'localhost:9200/test/_mget?pretty' -H 'Content-Type: application/json' -d'
{
    "ids" : ["1", "1"]
}
'

```



您需要在这种情况下明确地设置`_type`：

```
GET / test / _mget // test / _mget /
{{
  “docs”：[“docs” ：[  
        {{
            “_type”： “的typeA”“_type” ：“typeA” ，
            “_id”：“1”“_id” ：“1”  
        }，}，
        {{
            “_type”： “TYPEB”“_type” ：“typeB” ，
            “_id”：“1”“_id” ：“1”  
        }}
    ]]
}}
```

### 源过滤（Source filtering）

默认情况下，该`_source`字段将返回每个文档（如果存储）。与[get](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#get-source-filtering) API 类似，您可以`_source`通过使用该`_source`参数仅检索部分（或根本不检索）。您还可以使用URL参数`_source`，`_source_include`及`_source_exclude`当没有每个文档的说明，指定默认值，这将被使用。

例如：

```
curl -XGET 'localhost:9200/_mget?pretty' -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "_source" : false
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2",
            "_source" : ["field3", "field4"]
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "3",
            "_source" : {
                "include": ["user"],
                "exclude": ["user.location"]
            }
        }
    ]
}
'
```

### 字段(Fields)

可以指定特定的存储字段以获取每个文档，类似于Get API 的[stored_fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html#get-stored-fields)参数。例如：

```
curl -XGET 'localhost:9200/_mget?pretty' -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "stored_fields" : ["field1", "field2"]
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2",
            "stored_fields" : ["field3", "field4"]
        }
    ]
}
'
```

或者，您可以`stored_fields`将查询字符串中的参数指定为缺省值，以应用于所有文档。

```
curl -XGET 'localhost:9200/test/type/_mget?stored_fields=field1,field2&pretty' -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_id" : "1" (1)
        },
        {
            "_id" : "2",
            "stored_fields" : ["field3", "field4"]  (2)
        }
    ]
}
'

```



| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-get.html#CO22-1) | 返回`field1`和`field2` |
| ---------------------------------------- | ------------------- |
| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/2.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-get.html#CO22-2) | 返回`field3`和`field4` |

### 路由

您还可以指定路由值作为参数：

```
curl -XGET 'localhost:9200/_mget?routing=key1&pretty' -H 'Content-Type: application/json' -d'
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "1",
            "routing" : "key2"
        },
        {
            "_index" : "test",
            "_type" : "type",
            "_id" : "2"
        }
    ]
}
'
```

在这个例子中，文档`test/type/2`将从对应于路由关键字的分片中获取，`key1`但是文档`test/type/1`将从对应于路由关键字的分片获取`key2`。

### 安全

请参阅[*基于URL的访问控制*](https://www.elastic.co/guide/en/elasticsearch/reference/current/url-access-control.html)
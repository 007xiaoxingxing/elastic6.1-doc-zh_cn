## Multi termvectors API

Multi termvectors API允许一次获取多个termvector。从中检索term vector的document由索引，type和ID指定。但是document也可以在请求本身中人为地提供。

响应包括一个`docs` 包含所有提取的termvector 的数组，每个元素都具有由[termvectors](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html) API 提供的结构。这里是一个例子：

```
curl -XPOST 'localhost:9200/_mtermvectors?pretty' -H 'Content-Type: application/json' -d'
{
   "docs": [
      {
         "_index": "twitter",
         "_type": "tweet",
         "_id": "2",
         "term_statistics": true
      },
      {
         "_index": "twitter",
         "_type": "tweet",
         "_id": "1",
         "fields": [
            "message"
         ]
      }
   ]
}
'

```



有关可能参数的说明，请参阅[termvectors](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html) API。

的`_mtermvectors`接口也可以针对索引（在这种情况下它不是请求体所必需的）中使用：

```
curl -XPOST 'localhost:9200/twitter/_mtermvectors?pretty' -H 'Content-Type: application/json' -d'
{
   "docs": [
      {
         "_type": "tweet",
         "_id": "2",
         "fields": [
            "message"
         ],
         "term_statistics": true
      },
      {
         "_type": "tweet",
         "_id": "1"
      }
   ]
}
'
```

并输入：

```
curl -XPOST 'localhost:9200/twitter/tweet/_mtermvectors?pretty' -H 'Content-Type: application/json' -d'
{
   "docs": [
      {
         "_id": "2",
         "fields": [
            "message"
         ],
         "term_statistics": true
      },
      {
         "_id": "1"
      }
   ]
}
'
```

如果所有请求的文档都是相同的索引并且具有相同的类型，并且参数相同，则可以简化请求：

```
curl -XPOST 'localhost:9200/twitter/tweet/_mtermvectors?pretty' -H 'Content-Type: application/json' -d'
{
    "ids" : ["1", "2"],
    "parameters": {
        "fields": [
                "message"
        ],
        "term_statistics": true
    }
}
'
```

另外，就像[termvectors](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html) API一样，可以为用户提供的文档生成term vector。使用的映射由`_index`和确定`_type`。

```
curl -XPOST 'localhost:9200/_mtermvectors?pretty' -H 'Content-Type: application/json' -d'
{
   "docs": [
      {
         "_index": "twitter",
         "_type": "tweet",
         "doc" : {
            "user" : "John Doe",
            "message" : "twitter test test test"
         }
      },
      {
         "_index": "twitter",
         "_type": "test",
         "doc" : {
           "user" : "Jane Doe",
           "message" : "Another twitter test ..."
         }
      }
   ]
}
'

```


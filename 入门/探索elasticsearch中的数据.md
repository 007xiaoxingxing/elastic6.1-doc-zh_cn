## 搜索数据(Exploring Your Data)

现在我们已经看到了一些基本知识，让我们尝试一下更加真实的数据集。我准备了一个关于客户银行账户信息的虚构的JSON文档样本。每个文档都有以下模式：

```
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```

满足你的好奇心，这个数据是使用[`www.json-generator.com/`](http://www.json-generator.com/)生成的，所以请忽略数据的实际值和语义，因为这些都是随机生成的。

### 加载示例数据集

您可以从[这里](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)下载示例数据集（accounts.json）。将其解压到我们当前的目录，并将其加载到我们的集群中，如下所示：

```shell
wget https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true
mv accounts.json?raw=true accounts.json
curl -H "Content-Type: application/json" -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@accounts.json"
curl 'localhost:9200/_cat/indices?v'
```

响应：

```shell
root@iZ3pwm54lxe41mZ:~# curl 'localhost:9200/_cat/indices?v'
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank     YemKBWUUQRqh_VU1PNA7Yw   5   1       1000            0    103.6kb        103.6kb
yellow open   customer uC3Ta7G8Q9i-7u2TtbT2qg   5   1          2            0      7.8kb          7.8kb
```

这意味着我们只是成功将1000个文件批量索引到银行索引（在帐户类型下）。

## 搜索API(Search API)

现在让我们开始一些简单的搜索。运行搜索有两种基本方式：一种是通过[REST请求URI](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/search-uri-request.html)发送搜索参数和通过[REST请求主体](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/search-request-body.html)发送它们。reques body允许您更具自主性，并以更易读的JSON格式定义您的搜索。我们将尝试请求URI方法的一个例子，但在本教程的剩余部分中，我们将专门使用请求主体方法。

用于搜索的REST API可以从`_search`端点访问。本示例返回银行索引中的所有文档：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty&pretty' | more
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{
  "took" : 26,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [
          0
        ]
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "1",
        "_score" : null,
        "_source" : {
          "account_number" : 1,
          "balance" : 39225,
          "firstname" : "Amber",
          "lastname" : "Duke",
          "age" : 32,
          "gender" : "M",
          "address" : "880 Holmes Lane",
          "employer" : "Pyrami",
```

我们首先解析搜索调用。我们`_search`在银行索引中搜索（端点），并且该`q=*`参数指示Elasticsearch匹配索引中的所有文档。该`sort=account_number:asc`参数指示`account_number`按升序对每个文档的字段进行排序。这个`pretty`参数再次告诉Elasticsearch返回JSON结果。

在响应体中，我们看到以下部分：

- `took` - Elasticsearch执行搜索的时间（以毫秒为单位）
- `timed_out` - 告诉我们搜索是否超时
- `_shards` - 告诉我们搜索了多少碎片，以及搜索碎片成功/失败的次数
- `hits` - 搜索结果
- `hits.total` - 符合我们搜索条件的文件总数
- `hits.hits` - 实际的搜索结果数组（默认为前10个文档）
- `hits.sort` - 对结果进行排序的键（按分数排序时丢失）
- `hits._score`并且`max_score`- 现在忽略这些字段

以上是使用替代请求主体方法的上述相同的确切搜索：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
' | more
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{
  "took" : 23,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
```

这里的区别在于，我们没有传入`q=*`URI，而是将一个JSON风格的查询请求体发送到`_search`API。我们将在下一节讨论这个JSON查询。

重要的是要明白，一旦你得到你的搜索结果，Elasticsearch完成了请求，并没有维护任何种类的服务器端资源或打开游标到你的结果。这与许多其他平台（如SQL）形成鲜明对比，其中您最初可能会首先获得查询结果的部分子集，然后如果要获取（或翻阅）其余部分，则必须连续返回到服务器的结果使用某种有状态的服务器端游标。



## 介绍查询语言（Introducing the Query Language）

Elasticsearch提供了一种可用于执行查询的JSON式特定于域的语言。这被称为[Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/query-dsl.html)。查询语言非常全面，可以乍一看吓人，但实际学习的最好方法是从几个基本的例子开始。

回到我们的最后一个例子，我们执行了这个查询：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} }
}
' | more
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    36    0     0  100    36      0   1545 --:--:-- --:--:-- --:--:--  1500{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "25",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 25,
          "balance" : 40540,
          "firstname" : "Virginia",
          "lastname" : "Ayala",
          "age" : 39,
          "gender" : "F",
          "address" : "171 Putnam Avenue",
          "employer" : "Filodyne",
          "email" : "virginiaayala@filodyne.com",
          "city" : "Nicholson",
          "state" : "PA"
        }
```

剖析上述响应，`query`部分告诉我们我们的查询定义是什么，`match_all`部分只是我们想要运行的查询的类型。该`match_all`查询仅仅是在指定索引的所有文件进行搜索。

除了`query`参数之外，我们还可以传递其他参数来影响搜索结果。除了在上面的例子中 `sort`，我们尝试一下`size`：

```
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": { "match_all": {} },
>   "size": 1
> }
> '
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "25",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 25,
          "balance" : 40540,
          "firstname" : "Virginia",
          "lastname" : "Ayala",
          "age" : 39,
          "gender" : "F",
          "address" : "171 Putnam Avenue",
          "employer" : "Filodyne",
          "email" : "virginiaayala@filodyne.com",
          "city" : "Nicholson",
          "state" : "PA"
        }
      }
    ]
  }
}
```

请注意，如果`size`未指定，则默认为10。

这个例子执行`match_all`并返回文档11到20：

```
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": { "match_all": {} },
>   "from": 10,
>   "size": 10
> }
> '
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "227",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 227,
          "balance" : 19780,
          "firstname" : "Coleman",
          "lastname" : "Berg",
          "age" : 22,
          "gender" : "M",
          "address" : "776 Little Street",
          "employer" : "Exoteric",
          "email" : "colemanberg@exoteric.com",
          "city" : "Eagleville",
          "state" : "WV"
        }
      },
```

在`from`（从0开始）参数规定了从启动该文件的索引和`size`参数指定了多少文件，返回从参数开始的。在实现分页搜索结果时，此功能非常有用。请注意，如果`from`未指定，则默认为0。

此示例`match_all`按帐户余额按降序对结果进行排序并返回前10个（默认大小）文档。

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": { "match_all": {} },
>   "sort": { "balance": { "order": "desc" } }
> }
> '
{
  "took" : 11,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "248",
        "_score" : null,
        "_source" : {
          "account_number" : 248,
          "balance" : 49989,
          "firstname" : "West",
          "lastname" : "England",
          "age" : 36,
          "gender" : "M",
          "address" : "717 Hendrickson Place",
          "employer" : "Obliq",
          "email" : "westengland@obliq.com",
          "city" : "Maury",
          "state" : "WA"
        },
        "sort" : [
          49989
        ]
      },
```

## 执行搜索(Executing Searches)

现在我们已经看到了一些基本的搜索参数，让我们再深入Query DSL。我们先来看看返回的文档字段。默认情况下，完整的JSON文档作为所有搜索的一部分返回。这被称为源（`_source`搜索命中字段）。如果我们不希望整个源文档返回，我们可以做到只按需要返回源内的几个字段。

这个例子展示了如何从搜索中返回两个字段`account_number`和`balance`（在`_source`）之内：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": { "match_all": {} },
>   "_source": ["account_number", "balance"]
> }
> '
{
  "took" : 32,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "25",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 25, //看这里，只返回了两个字段
          "balance" : 40540
        }
```

请注意，上面的例子简单地减少了`_source`字段。它仍然只会返回一个字段，`_source`但在其中，只有字段`account_number`，`balance`并包括在内。

如果你来自一个SQL背景，上面在概念上有点类似于`SQL SELECT FROM`字段列表。

现在我们来看看查询部分。以前，我们已经看过如何使用`match_all`查询来匹配所有文档。现在我们来介绍一个叫做[`match` query](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/query-dsl-match-query.html)的新查询，这个[查询](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/query-dsl-match-query.html)可以被看作是一个基本的搜索查询（即针对一个特定的字段或一组字段进行的搜索）。

此示例返回编号为20的帐户：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": { "match": { "account_number": 20 } }
> }
> '
{
  "took" : 42,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "20",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 20,
          "balance" : 16418,
          "firstname" : "Elinor",
          "lastname" : "Ratliff",
          "age" : 36,
          "gender" : "M",
          "address" : "282 Kings Place",
          "employer" : "Scentric",
          "email" : "elinorratliff@scentric.com",
          "city" : "Ribera",
          "state" : "WA"
        }
      }
    ]
  }
}
```

此示例返回地址中包含术语“mill”的所有帐户：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": { "match": { "address": "mill" } }
> }
> '
{
  "took" : 31,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 4,
    "max_score" : 4.89784,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "472",
        "_score" : 4.89784,
        "_source" : {
          "account_number" : 472,
          "balance" : 25571,
          "firstname" : "Lee",
          "lastname" : "Long",
          "age" : 32,
          "gender" : "F",
          "address" : "288 Mill Street",
          "employer" : "Comverges",
          "email" : "leelong@comverges.com",
          "city" : "Movico",
          "state" : "MT"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 4.8485627,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      },
```

此示例返回地址中包含术语“mill”或“lane”的所有帐户：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": { "match": { "address": "mill lane" } }
> }
> '
{
  "took" : 47,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 19,
    "max_score" : 8.398771,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 8.398771,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "472",
        "_score" : 4.89784,
        "_source" : {
          "account_number" : 472,
          "balance" : 25571,
          "firstname" : "Lee",
          "lastname" : "Long",
          "age" : 32,
          "gender" : "F",
          "address" : "288 Mill Street",
          "employer" : "Comverges",
          "email" : "leelong@comverges.com",
          "city" : "Movico",
          "state" : "MT"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "556",
        "_score" : 4.4402957,
        "_source" : {
          "account_number" : 556,
          "balance" : 36420,
          "firstname" : "Collier",
          "lastname" : "Odonnell",
          "age" : 35,
          "gender" : "M",
          "address" : "591 Nolans Lane",
          "employer" : "Sultraxin",
          "email" : "collierodonnell@sultraxin.com",
          "city" : "Fulford",
          "state" : "MD"
        }
      },

```

[在控制台中](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/_executing_searches/4.json)[复制为CURL ]()[视图](http://localhost:5601/app/kibana#/dev_tools/console?load_from=https://www.elastic.co/guide/en/elasticsearch/reference/current/snippets/_executing_searches/4.json)[ ]()

这个例子是`match`（`match_phrase`）的一个变体，返回在地址中包含短语“mill lane”的所有账户：

```
GET /银行/ _搜索
{
  “query”：{“match_phrase”：{“address”：“mill lane”}}
}
```

现在我们来介绍一下这个[`bool` query](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/query-dsl-bool-query.html)。该`bool`查询允许我们撰写较小的查询到使用布尔逻辑更大的查询。

此示例组成两个`match`查询，并返回地址中包含“mill”和“lane”的所有帐户：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": {
>     "bool": {
>       "must": [
>         { "match": { "address": "mill" } },
>         { "match": { "address": "lane" } }
>       ]
>     }
>   }
> }
> '
{
  "took" : 54,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 8.398771,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 8.398771,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      }
    ]
  }
}
```

在上面的例子中，`bool must`子句指定了一个文件被认为是匹配的所有查询。

相反，这个例子组成两个`match`查询，并返回地址中包含“mill”或“lane”的所有帐户：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": {
>     "bool": {
>       "should": [
>         { "match": { "address": "mill" } },
>         { "match": { "address": "lane" } }
>       ]
>     }
>   }
> }
> '
{
  "took" : 19,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 19,
    "max_score" : 8.398771,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "136",
        "_score" : 8.398771,
        "_source" : {
          "account_number" : 136,
          "balance" : 45801,
          "firstname" : "Winnie",
          "lastname" : "Holland",
          "age" : 38,
          "gender" : "M",
          "address" : "198 Mill Lane",
          "employer" : "Neteria",
          "email" : "winnieholland@neteria.com",
          "city" : "Urie",
          "state" : "IL"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "472",
        "_score" : 4.89784,
        "_source" : {
          "account_number" : 472,
          "balance" : 25571,
          "firstname" : "Lee",
          "lastname" : "Long",
          "age" : 32,
          "gender" : "F",
          "address" : "288 Mill Street",
          "employer" : "Comverges",
          "email" : "leelong@comverges.com",
          "city" : "Movico",
          "state" : "MT"
        }
      },
```

在上面的例子中，`bool should`子句指定了一个查询列表，其中任何一个查询都必须是true才能被视为匹配的文档。

此示例组成两个`match`查询，并返回地址中既不包含“mill”也不包含“lane”的所有帐户：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": {
>     "bool": {
>       "must_not": [
>         { "match": { "address": "mill" } },
>         { "match": { "address": "lane" } }
>       ]
>     }
>   }
> }
> '
{
  "took" : 33,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 981,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "25",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 25,
          "balance" : 40540,
          "firstname" : "Virginia",
          "lastname" : "Ayala",
          "age" : 39,
          "gender" : "F",
          "address" : "171 Putnam Avenue",
          "employer" : "Filodyne",
          "email" : "virginiaayala@filodyne.com",
          "city" : "Nicholson",
          "state" : "PA"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "44",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 44,
          "balance" : 34487,
          "firstname" : "Aurelia",
          "lastname" : "Harding",
          "age" : 37,
          "gender" : "M",
          "address" : "502 Baycliff Terrace",
          "employer" : "Orbalix",
          "email" : "aureliaharding@orbalix.com",
          "city" : "Yardville",
          "state" : "DE"
        }
      },
```

在上面的例子中，`bool must_not`子句指定了一个查询列表，其中任何一个查询都不可以被认为是匹配的。

我们可以在一个查询中同时结合`must`，`should`和`must_not`等bool条件。此外，我们可以使用`bool`条件来查询复杂的多级布尔逻辑条件。

这个例子返回所有40岁但不住ID（aho）的人的账号：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": {
>     "bool": {
>       "must": [
>         { "match": { "age": "40" } }
>       ],
>       "must_not": [
>         { "match": { "state": "ID" } }
>       ]
>     }
>   }
> }
> '
{
  "took" : 22,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 43,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "948",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 948,
          "balance" : 37074,
          "firstname" : "Sargent",
          "lastname" : "Powers",
          "age" : 40,
          "gender" : "M",
          "address" : "532 Fiske Place",
          "employer" : "Accuprint",
          "email" : "sargentpowers@accuprint.com",
          "city" : "Umapine",
          "state" : "AK"
        }
      },
```

## 执行过滤器（Excuting Filters）

在上一节中，我们跳过了一个叫做文档分数（搜索结果中的`_score`字段）的细节。得分是一个数字值，它是文档与我们指定的搜索查询匹配度的相对度量。分数越高，文档越相关，分数越低，文档就越不相关。

但查询并不总是需要产生分数，特别是当它们仅用于“过滤”文档集时。Elasticsearch检测这些情况并自动优化查询执行，以便不计算无用分数。

我们在前一节中介绍的[`bool`query](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/query-dsl-bool-query.html)也支持`filter`使用查询来限制将被其他子句匹配的文档的子句，而不改变计算分数的方式。作为一个例子，我们来介绍一下[`range`query](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/query-dsl-range-query.html)，它允许我们通过一系列值来过滤文档。这通常用于数字或日期过滤。

本示例使用bool查询返回余额在20000和30000之间的所有帐户。换句话说，我们要查找大于或等于20000且小于等于30000的帐户。

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "query": {
>     "bool": {
>       "must": { "match_all": {} },
>       "filter": {
>         "range": {
>           "balance": {
>             "gte": 20000,
>             "lte": 30000
>           }
>         }
>       }
>     }
>   }
> }
> '
{
  "took" : 43,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 217,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "253",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 253,
          "balance" : 20240,
          "firstname" : "Melissa",
          "lastname" : "Gould",
          "age" : 31,
          "gender" : "M",
          "address" : "440 Fuller Place",
          "employer" : "Buzzopia",
          "email" : "melissagould@buzzopia.com",
          "city" : "Lumberton",
          "state" : "MD"
        }
      },
      {
        "_index" : "bank",
        "_type" : "account",
        "_id" : "400",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 400,
          "balance" : 20685,
          "firstname" : "Kane",
          "lastname" : "King",
          "age" : 21,
          "gender" : "F",
          "address" : "405 Cornelia Street",
          "employer" : "Tri@Tribalog",
          "email" : "kaneking@tri@tribalog.com",
          "city" : "Gulf",
          "state" : "VT"
        }
      },
```

解释一下上述响应，bool查询包含`match_all`查询（查询部分）和`range`查询（过滤器部分）。我们可以将其他查询替换为查询和过滤器部分。在上述情况下，范围查询是非常有意义的，因为落入该范围的文档全部匹配“平等”，即没有文档比另一个文档更相关。

除了`match_all`，`match`，`bool`，和`range`查询，有很多可用的其他查询类型的，我们不会在这里讨论他们。由于我们已经对其工作原理有了一个基本的了解，所以将这些知识应用于其他查询类型的学习和实验并不难。



## 执行聚合（Executing Aggregations）

聚合提供了从数据中分组和提取统计数据的能力。考虑聚合的最简单方法是将其大致等同于SQL GROUP BY和SQL聚合函数。在Elasticsearch中，您可以执行搜索并返回匹配，同时还可以在一个响应中返回与匹配不同的聚合结果。这是非常强大和高效的，因为您可以运行查询和多个聚合，并且一次性获得两个（或两个）操作的结果，避免使用简洁和简化的API来减少网络往返。

首先，本示例按state对所有帐户进行分组，然后返回按降序（也是默认值）排序的前10个（默认）state：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "size": 0,
>   "aggs": {
>     "group_by_state": {
>       "terms": {
>         "field": "state.keyword"
>       }
>     }
>   }
> }
> '
{
  "took" : 107,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 20,
      "sum_other_doc_count" : 770,
      "buckets" : [
        {
          "key" : "ID",
          "doc_count" : 27
        },
        {
          "key" : "TX",
          "doc_count" : 27
        },
        {
          "key" : "AL",
          "doc_count" : 25
        },
        {
          "key" : "MD",
          "doc_count" : 25
        },
        {
          "key" : "TN",
          "doc_count" : 23
        },
```

在SQL中，上面的聚合在概念上类似于：

```sql
SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC
```

我们可以看到`ID`（爱达荷州）有27个账户，其次是`TX`（德克萨斯州）27个账户，其次是`AL`（阿拉巴马州）25个账户，等等。

请注意，我们设置`size=0`为不显示搜索匹配，因为我们只想看到响应中的聚合结果。

在前面的汇总基础上，本示例按状态计算平均账户余额（再次仅按按降序排列的前10个状态）：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "size": 0,
>   "aggs": {
>     "group_by_state": {
>       "terms": {
>         "field": "state.keyword"
>       },
>       "aggs": {
>         "average_balance": {
>           "avg": {
>             "field": "balance"
>           }
>         }
>       }
>     }
>   }
> }
> '
{
  "took" : 85,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : 20,
      "sum_other_doc_count" : 770,
      "buckets" : [
        {
          "key" : "ID",
          "doc_count" : 27,
          "average_balance" : {
            "value" : 24368.777777777777
          }
        },
        {
          "key" : "TX",
          "doc_count" : 27,
          "average_balance" : {
            "value" : 27462.925925925927
          }
        },
        {
          "key" : "AL",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 25739.56
          }
        },
        {
          "key" : "MD",
          "doc_count" : 25,
          "average_balance" : {
            "value" : 24963.52
          }
        },
```

注意我们是如何嵌套在`average_balance`聚合中的`group_by_state`聚合。这是所有聚合的通用模式。您可以任意嵌套聚合内的聚合，以便从数据中提取所需的旋转摘要。

基于以前的聚合，现在让我们按降序对平均余额进行排序：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "size": 0,
>   "aggs": {
>     "group_by_state": {
>       "terms": {
>         "field": "state.keyword",
>         "order": {
>           "average_balance": "desc"
>         }
>       },
>       "aggs": {
>         "average_balance": {
>           "avg": {
>             "field": "balance"
>           }
>         }
>       }
>     }
>   }
> }
> '
{
  "took" : 87,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound" : -1,
      "sum_other_doc_count" : 918,
      "buckets" : [
        {
          "key" : "AL",
          "doc_count" : 6,
          "average_balance" : {
            "value" : 41418.166666666664
          }
        },
        {
          "key" : "SC",
          "doc_count" : 1,
          "average_balance" : {
            "value" : 40019.0
          }
        },
        {
          "key" : "AZ",
          "doc_count" : 10,
          "average_balance" : {
            "value" : 36847.4
          }
        },
        {
          "key" : "VA",
          "doc_count" : 13,
          "average_balance" : {
            "value" : 35418.846153846156
          }
        },
```

这个例子演示了我们如何按年龄段（20-29岁，30-39岁和40-49岁）进行分组，然后按性别进行分组，然后最终得到每个年龄段的平均账户余额：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
> {
>   "size": 0,
>   "aggs": {
>     "group_by_age": {
>       "range": {
>         "field": "age",
>         "ranges": [
>           {
>             "from": 20,
>             "to": 30
>           },
>           {
>             "from": 30,
>             "to": 40
>           },
>           {
>             "from": 40,
>             "to": 50
>           }
>         ]
>       },
>       "aggs": {
>         "group_by_gender": {
>           "terms": {
>             "field": "gender.keyword"
>           },
>           "aggs": {
>             "average_balance": {
>               "avg": {
>                 "field": "balance"
>               }
>             }
>           }
>         }
>       }
>     }
>   }
> }
> '
{
  "took" : 37,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 0.0,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_age" : {
      "buckets" : [
        {
          "key" : "20.0-30.0",
          "from" : 20.0,
          "to" : 30.0,
          "doc_count" : 451,
          "group_by_gender" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "M",
                "doc_count" : 232,
                "average_balance" : {
                  "value" : 27374.05172413793
                }
              },
              {
                "key" : "F",
                "doc_count" : 219,
                "average_balance" : {
                  "value" : 25341.260273972603
                }
              }
            ]
          }
        },
        {
          "key" : "30.0-40.0",
          "from" : 30.0,
          "to" : 40.0,
          "doc_count" : 504,
          "group_by_gender" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "F",
                "doc_count" : 253,
                "average_balance" : {
                  "value" : 25670.869565217392
                }
              },
              {
                "key" : "M",
                "doc_count" : 251,
                "average_balance" : {
                  "value" : 24288.239043824702
                }
              }
            ]
          }
        },
```

还有很多其他聚合功能，我们在这里不会详细介绍。如果你想要做进一步的实验，该[聚合参考指南](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/search-aggregations.html)是一个很好的参考资料。



## 结论

Elasticsearch既是一个简单又复杂的产品。到目前为止，我们已经学习了什么是基础知识，如何看待它，以及如何使用一些REST API来使用它。希望本教程能够让您更好地理解Elasticsearch是什么，更重要的是，启发您进一步尝试其余的强大功能！
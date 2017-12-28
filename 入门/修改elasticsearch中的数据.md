## 修改数据

Elasticsearch几乎实时提供数据操作和搜索功能。默认情况下，从索引/更新/删除数据开始，直到搜索结果中显示的时间为止，您可以预期会有一秒的延迟（刷新间隔）。这是与SQL等其他平台的重要区别，其中数据在事务完成后立即可用。

### 索引/替换文档

我们以前看过我们如何索引一个文档。让我们再次运行创建index的命令：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XPUT 'localhost:9200/customer/doc/1?pretty&pretty' -H 'Content-Type: application/json' -d'
> {
>   "name": "John Doe"
> }
> '
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

上面将索引指定的文件到customer索引，ID为1.如果我们然后用不同的（或相同的）documnet再次执行上述命令，Elasticsearch将取代（即重新索引）一个新的文件现有的ID为1：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XPUT 'localhost:9200/customer/doc/1?pretty&pretty' -H 'Content-Type: application/json' -d'
> {
>   "name": "Jane Doe"
> }
> '
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

以上将ID号为1的文档的名称从“John Doe”改为“Jane Doe”。另一方面，如果我们使用不同的ID，则新文档将被索引，并且索引中已经存在的文档将保持不变。

```shell
root@iZ3pwm54lxe41mZ:~# curl -XPUT 'localhost:9200/customer/doc/2?pretty&pretty' -H 'Content-Type: application/json' -d'
> {
>   "name": "Jane Doe"
> }
> '
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "2",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

以上索引ID为2的新文档。

索引时，ID部分是可选的。如果未指定，Elasticsearch将生成一个随机ID，然后用它来索引文档。Elasticsearch生成的实际ID（或者我们在前面例子中明确指定的）会作为索引API调用的一部分返回。

这个例子展示了如何索引没有显式ID的文档：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XPOST 'localhost:9200/customer/doc?pretty&pretty' -H 'Content-Type: application/json' -d'
> {
>   "name": "Jane Doe"
> }
> '
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "CugEm2ABA2ItF2NYUf8e", //随机id
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

请注意，在上面的例子中，我们使用的是`POST`动词而不是PUT，因为我们没有指定一个ID。



## 更新文档(Updating Documents)

除了能够索引和替换文档之外，我们还可以更新文档。请注意，虽然Elasticsearch实际上并没有在原地进行更新。无论何时我们进行更新，Elasticsearch都会删除旧文档，然后索引一个新文档，一次性应用更新。

此示例显示如何通过将名称字段更改为“Jane Doe”来更新以前的文档（ID为1）：

```
root@iZ3pwm54lxe41mZ:~# curl -XPOST 'localhost:9200/customer/doc/1/_update?pretty&pretty' -H 'Content-Type: application/json' -d'
> {
>   "doc": { "name": "Jane Doe" }
> }
> '
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "noop",
  "_shards" : {
    "total" : 0,
    "successful" : 0,
    "failed" : 0
  }
}
```

这个例子展示了如何通过改变名称字段为“Jane Doe”来更新我们以前的文档（ID为1），同时为其添加一个年龄字段：

```
root@iZ3pwm54lxe41mZ:~# curl -XPOST 'localhost:9200/customer/doc/1/_update?pretty&pretty' -H 'Content-Type: application/json' -d'
> {
>   "doc": { "name": "Jane Doe", "age": 20 }
> }
> '
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

更新也可以通过使用简单的脚本来执行。此示例使用脚本将年龄增加5：

```
root@iZ3pwm54lxe41mZ:~# curl -XPOST 'localhost:9200/customer/doc/1/_update?pretty&pretty' -H 'Content-Type: application/json' -d'
> {
>   "script" : "ctx._source.age += 5"
> }
> '
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 4,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

在上例中，`ctx._source`引用即将更新的当前源文档。

Elasticsearch提供了在给定查询条件（如`SQL UPDATE-WHERE`语句）的情况下更新多个文档的能力。请参阅[`docs-update-by-query`API](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/docs-update-by-query.html)



## 删除文件

删除文件相当简单。此示例显示如何删除我们以前的客户，其ID为2：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XDELETE 'localhost:9200/customer/doc/2?pretty&pretty'
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "2",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
DELETE / customer / doc / 2？漂亮
```

查看[`_delete_by_query`API](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/docs-delete-by-query.html)以删除与特定查询匹配的所有文档。值得注意的是，删除整个索引而不是使用Delete By Query API删除所有文档会更有效率



## 批处理(Batch Processing)

除了能够索引，更新和删除单个文档之外，Elasticsearch还提供了使用[`_bulk`API](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/docs-bulk.html)批量执行上述任何操作的[功能](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/docs-bulk.html)。这个功能非常重要，因为它提供了一个非常有效的机制，尽可能快地完成多个操作，尽可能少的网络往返。

作为一个简单的例子，下面的调用在一个批量操作中索引两个文档（ID 1 - John Doe和ID 2 - Jane Doe）：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XPOST 'localhost:9200/customer/doc/_bulk?pretty&pretty' -H 'Content-Type: application/json' -d'
> {"index":{"_id":"1"}}
> {"name": "John Doe" }
> {"index":{"_id":"2"}}
> {"name": "Jane Doe" }
> '
{
  "took" : 41,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "customer",
        "_type" : "doc",
        "_id" : "1",
        "_version" : 5,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 5,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "index" : {
        "_index" : "customer",
        "_type" : "doc",
        "_id" : "2",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```

本示例更新第一个文档（ID为1），然后在一个批量操作中删除第二个文档（ID为2）：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XPOST 'localhost:9200/customer/doc/_bulk?pretty&pretty' -H 'Content-Type: application/json' -d'
> {"update":{"_id":"1"}}
> {"doc": { "name": "John Doe becomes Jane Doe" } }
> {"delete":{"_id":"2"}}
> '
{
  "took" : 46,
  "errors" : false,
  "items" : [
    {
      "update" : {
        "_index" : "customer",
        "_type" : "doc",
        "_id" : "1",
        "_version" : 6,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 6,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "delete" : {
        "_index" : "customer",
        "_type" : "doc",
        "_id" : "2",
        "_version" : 2,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}

```

请注意，对于删除操作，之后没有对应的源文档，因为删除操作只需要删除文档的标识。

批量API不会因其中一个操作失败而失败。如果一个动作因任何原因失败，它将继续处理其余的动作。批量API返回时，它将为每个操作提供一个状态（与发送的顺序相同），以便您可以检查特定操作是否失败。
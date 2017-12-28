## 探索（exploring）你的集群

### REST API

现在我们已经启动并运行了节点（和集群），下一步就是了解如何与它进行通信。幸运的是，Elasticsearch提供了一个非常全面和强大的REST API，您可以使用它来与群集进行交互。可以用API完成的几件事情如下：

- 检查您的群集，节点和索引运行状况，状态和统计信息
- 管理您的群集，节点和索引数据和元数据
- 对您的索引执行CRUD（创建，读取，更新和删除）和搜索操作
- 执行高级搜索操作，如分页，排序，过滤，脚本，聚合等等



## 集群健康状态

我们从一个基本的健康检查开始，我们可以用它来看看我们的集群是如何工作的。我们将使用curl来做到这一点，但是您可以使用任何工具来允许您进行HTTP / REST调用。假设我们仍然在开始Elasticsearch的同一个节点上，并打开另一个shell窗口。

要检查群集健康状况，我们将使用[`_cat`API](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/cat.html)。您可以 通过curl执行以下命令：

```shell
curl -XGET 'localhost:9200/_cat/health?v&pretty'
```

服务器响应是：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/_cat/health?v&pretty'
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1514427140 10:12:20  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

我们可以看到，我们的集群名为“elasticsearch”是处于green状态的。

每当我们要求群集健康状态，我们得到的响应会是green，yellow，或red。

- green 一切都很好（集群功能齐全）
- yellow 所有数据都可用，但一些副本尚未分配（群集完全可用）
- red 不知出于何种原因某些数据不可用（群集只具备部分功能）

**注意：**当一个群集为red状态时，它将继续提供来自可用碎片的搜索请求，但是您可能需要尽快修复它，因为有未分配的碎片。

同样从上面的响应中，我们可以看到总共有1个节点，而我们有0个碎片，因为我们还没有数据。请注意，由于我们使用的是默认集群名称（elasticsearch），并且由于Elasticsearch默认使用单播网络发现来查找同一台计算机上的其他节点，所以可能会意外启动计算机上的多个节点并使其全部加入一个集群。在这种情况下，您可能会在上述响应中看到多个节点。

我们也可以得到我们集群中的节点列表，使用curl执行如下请求：

```shell
curl -XGET 'localhost:9200/_cat/nodes?v&pretty'
```

得到的响应：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/_cat/nodes?v&pretty'
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1            6          93  87    2.58    2.44     2.39 mdi       *      2qiKZiO
```

在这里，我们可以看到我们的一个名为“2qiKZiO”的节点，它是当前在我们集群中的单个节点。



## 列出所有索引

现在我们来看看我们的指数：

```shell
curl -XGET 'localhost:9200/_cat/indices?v&pretty'
```

响应内容：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/_cat/indices?v&pretty'
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

这仅仅意味着我们在集群中还没有索引。



## 创建索引

现在让我们创建一个名为“customer”的索引，然后再列出所有的索引：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XPUT 'localhost:9200/customer?pretty&pretty'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "customer"
}
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/_cat/indices?v&pretty'
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 88gkXG1RTf-v7dOXAJitZA   5   1          0            0      1.1kb          1.1kb
```

第一个命令使用PUT动词创建名为“customer”的索引。我们简单地追加`pretty`到调用结束，告诉它打印JSON响应（如果有的话）。

第二个命令的响应显示，现在我们有一个名为customer的索引，它有5个主分片和1个副本（默认值），它包含0个文档。

您可能还会注意到customer索引标有黄色的健康状况。回想一下我们之前的讨论，黄色意味着一些复制品还没有被分配。这个索引发生的原因是因为Elasticsearch默认为这个索引创建了一个副本。由于此刻我们只有一个节点正在运行，因此，在另一个节点加入集群的较晚时间点之前，尚无法分配一个副本（以获得高可用性）。一旦该副本被分配到第二个节点上，该索引的健康状态将变成绿色。

## 索引和查询文档（Index and Query）

现在让我们把东西放入我们的customer索引。我们会将一个简单的customer文档编入customer索引，ID为1，如下所示：

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

客户索引里面成功创建了一个新的customer文档。该文件也有一个我们在索引时间指定的内部ID。

请注意，Elasticsearch不需要您先指定文档，然后再明确地创建一个索引。在前面的例子中，Elasticsearch将自动创建客户索引，如果事先不存在的话。

现在我们来检索我们刚编入索引的那个文档：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/customer/doc/1?pretty&pretty'
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```

这里除了一个字段以外没有什么特别的`found`，说明我们找到了一个带有请求ID 1的文档，另外一个字段`_source`返回了我们从前一步编入索引的完整的JSON文档。



## 删除索引

现在让我们删除刚刚创建的索引，然后再次列出所有索引：

```shell
root@iZ3pwm54lxe41mZ:~# curl -XDELETE 'localhost:9200/customer?pretty&pretty'
{
  "acknowledged" : true
}
root@iZ3pwm54lxe41mZ:~# curl -XGET 'localhost:9200/_cat/indices?v&pretty'
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

这意味着索引已经成功删除了，现在我们回到我们在集群中没有任何东西的状态。

在我们继续之前，让我们再仔细看看迄今为止学到的一些API命令：

```shell
curl -XPUT 'localhost:9200/customer?pretty'
curl -XPUT 'localhost:9200/customer/doc/1?pretty' -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
curl -XGET 'localhost:9200/customer/doc/1?pretty'
curl -XDELETE 'localhost:9200/customer?pretty'
```

如果我们仔细研究上面的命令，我们实际上可以看到我们如何在Elasticsearch中访问数据的模式。这种模式可以概括如下：

```shell
<REST Verb> /<Index>/<Type>/<ID>
```

这种REST访问模式在所有的API命令中都非常普遍，如果你能简单地记住它，你将在掌握Elasticsearch方面有一个良好的开端。


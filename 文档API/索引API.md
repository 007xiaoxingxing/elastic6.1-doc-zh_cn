## 索引API（Index API）

![重要](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/important.png)

请参阅[*移除映射类型*](https://www.elastic.co/guide/en/elasticsearch/reference/current/removal-of-types.html)。

索引API在特定的索引中添加或更新类型化的JSON文档，使其成为可搜索的。以下示例将JSON文档插入“twitter”索引中，名为“tweet”，ID为1的类型下：

```
curl -XPUT 'localhost:9200/twitter/tweet/1?pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'

```



上述索引操作的结果是：

```
{
  "_index" : "twitter",
  "_type" : "tweet",
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

的`_shards`报头提供关于索引操作的复制过程的信息。

- `total` - 表示索引操作应该执行多少个分片副本（主分片和副本分片）。
- `successful` - 表示索引操作成功的分片数量。
- `failed` - 在副本分片上的索引操作失败的情况下，包含与复制相关的错误的数组。

在索引操作成功的情况下`successful`至少是1。

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

当索引操作成功返回时，副本碎片可能不会全部启动（默认情况下，只有primary是必需的，但可以[更改](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html#index-wait-for-active-shards)此行为）。在这种情况下， `total`将等于基于`number_of_replicas`设置的总分片`successful`数，并等于启动的分片数（主复制数）。如果没有失败，`failed`将会是0。

### 自动创建索引

索引操作会自动创建一个索引（如果尚未创建索引）（检出 [创建索引API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)以手动创建索引），还会为特定类型（如果尚未创建）自动创建动态类型映射（签出该[put mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html) API用于手动创建类型映射）。

mapping本身非常灵活，schema-free。新的字段和对象将自动添加到指定类型的映射定义中。查看[映射](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) 部分了解映射定义的更多信息。

自动创建索引可以通过在所有节点的配置文件中设置`action.auto_create_index`为 `false`来禁用。自动映射创建可以通过设置被禁用 `index.mapper.dynamic`到`false`每个索引作为指标设置。

自动索引创建可以包括一个基于模式的白/黑名单，例如，设置`action.auto_create_index`为`+aaa*,-bbb*,+ccc*,-*`（+意思是允许的， - 意思是不允许的）。

### 版本（Versioning）

每个索引文档都有一个版本号。相关 `version`号码作为对索引API请求的响应的一部分返回。索引API可选地允许在`version`指定参数时进行 [optimistic concurency control](http://en.wikipedia.org/wiki/Optimistic_concurrency_control)。这将控制操作打算执行的文档的version。version用例的一个很好的例子就是执行一个事务性的read-then-update。从文档中指定一个version 最初的读取确保在此期间没有发生变化（阅读时为了更新，建议设置 `preference`为`_primary`）。例如：

```
curl -XPUT 'localhost:9200/twitter/tweet/1?version=2&pretty' -H 'Content-Type: application/json' -d'
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
'

```



**注意：**版本控制是完全实时的，不受搜索操作的近实时方面的影响。如果没有提供版本，则操作在没有任何版本检查的情况下执行。

默认情况下，使用内部版本控制，从1开始，每增加一个更新，包括删除。可选地，版本号可以用外部值补充（例如，如果保存在数据库中）。要启用这个功能，`version_type`应该设置为 `external`。提供的值必须是数字，长整数值大于或等于0，小于大约9.2e + 18。当使用外部版本类型时，系统将检查传递给索引请求的版本号是否大于当前存储的文档的版本，而不是检查匹配的版本号。如果为true，则将文档编入索引并使用新的版本号。如果提供的值小于或等于存储文档的版本号，则会发生版本冲突，索引操作将失败。

![警告](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/warning.png)

外部版本控制支持将值0作为有效的版本号。这允许版本与外部版本控制系统同步，其中版本号从零开始，而不是从一个开始。它具有副作用，版本号等于零的文档既不能使用[Update-By-Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html)进行更新，也不能使用[Delete By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)进行删除，只要版本号等于零即可。

一个好的副作用是，只要使用源数据库的版本号，就不需要对源数据库的更改所执行的异步索引操作进行严格排序。即使是使用外部版本控制来简化使用数据库中的数据来更新Elasticsearch索引的情况也是如此，因为如果索引操作由于任何原因而失序，则只使用最新版本。

#### 版本类型(Version types)

在上面解释的`internal`＆`external`版本类型旁边，Elasticsearch还支持其他类型的特定用例。以下是不同版本类型及其语义的概述。

- `internal`

  如果给定版本与存储文档的版本相同，则只索引文档。

- `external` 要么 `external_gt`

  如果给定的版本严格高于存储的文档的版本**或者**没有存在的文档，则只索引文档。给定的版本将被用作新版本，并将与新文档一起存储。提供的版本必须是非负数的长整数。

- `external_gte`

  如果给定版本**等于**或高于存储文档的版本，则只索引文档。如果没有现有的文件，操作也会成功。给定的版本将被用作新版本，并将与新文档一起存储。提供的版本必须是非负数的长整数。

**注意**：`external_gte`版本类型是用于特殊用途，应谨慎使用。如果使用不当，可能会导致数据丢失。还有另一个选项，`force`因为它可能会导致主分片和副本分片发生分化，因此已被弃用。

### 操作类型

索引操作也接受一个`op_type`可用于强制`create`操作的操作，允许“如果不在”行为。何时 `create`使用，索引操作将失败，如果通过该文档id已经存在索引中。

这是一个使用`op_type`参数的例子：

```
curl -XPUT 'localhost:9200/twitter/tweet/1?op_type=create&pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

另一个选项`create`是使用下面的uri：

```
curl -XPUT 'localhost:9200/twitter/tweet/1/_create?pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

### 自动ID生成

索引操作可以在不指定id的情况下执行。在这种情况下，会自动生成一个id。另外，`op_type` 会自动设置为`create`。这是一个例子（注意使用 **POST**而不是**PUT**）：

```
curl -XPOST 'localhost:9200/twitter/tweet/?pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```



上述索引操作的结果是：

```
{
  "_index" : "twitter",
  "_type" : "tweet",
  "_id" : "sSCaoWABy7CO3fxrZGCT",
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

### 路由(Routing)

默认情况下，分片位置 - 或`routing`- 通过使用文档的id值的散列进行控制。为了更加明确地控制，可以使用`routing`参数直接在每个操作的基础上指定馈送到路由器使用的散列函数的值。例如：

```
curl -XPOST 'localhost:9200/twitter/tweet?routing=kimchy&pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```

在上面的例子中，“tweet”文档根据提供的`routing`参数“kimchy” 被路由到一个分片。

设置显式映射时，`_routing`可以选择使用该字段来指示索引操作从文档本身提取路由值。这确实是以一个额外的文档解析过程的（非常小的）成本来实现的。如果`_routing`映射被定义并设置为`required`，则在没有提供或提取路由值的情况下，索引操作将失败。

### 分布式(Distributed)

索引操作将根据其路由（参见上面的“路由”部分）指向主碎片，并在包含此碎片的实际节点上执行。主分片完成操作后，如果需要，更新将分发到适用的副本。

### 等待活动碎片(Wait For Active Shards)

为了提高写入系统的弹性，索引操作可以配置为在继续操作之前等待一定数量的活动分片副本。如果所需数量的活动分片副本不可用，则写操作必须等待并重试，直到必要的分片副本已启动或发生超时。默认情况下，写入操作仅在等待主分片处于活动状态（ie `wait_for_active_shards=1`）之前处于活动状态。这个默认值可以通过设置动态地在索引设置中被覆盖`index.write.wait_for_active_shards`。要改变每个操作的这种行为，`wait_for_active_shards`可以使用请求参数。

有效值是`all`或者任何正整数，直到索引中每个分片的已配置副本总数（即`number_of_replicas+1`）。指定负值或大于分片数量的数字将引发错误。

例如，假设我们有三个节点的群集，`A`，`B`，和`C`，我们创建索引`index`设置为3的副本数量（导致4个碎片副本，一个副本多个比存在的节点）。如果我们尝试进行索引操作，默认情况下操作只会确保每个分片的主要副本可用，然后才能继续。这意味着，即使`B`和`C`下去，并`A`承载主要的分片副本，索引操作仍然只进行一个数据副本。如果`wait_for_active_shards`在请求中设置为`3`（并且所有3个节点都处于启动状态），那么索引操作将需要3个活动的分片副本，因此在集群中有3个活动节点，每个集群都拥有一个分片副本。但是，如果我们设置`wait_for_active_shards`为`all`（或`4`相同），则索引操作将不会继续，因为索引中没有每个分片的所有4个副本都处于活动状态。该操作将超时，除非在集群中引入新节点来托管分片的第四个副本。

需要注意的是，这种设置大大降低了写操作不写入所需数量的分片副本的可能性，但是这并不能完全消除这种可能性，因为在写操作开始之前发生这种检查。一旦写入操作正在进行，仍然有可能在任何数量的分片副本上复制失败，但是仍然可以在主分区上成功。在`_shards`写操作的响应部分揭示了其复制成功碎片的份数/失败。

```
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    }
}
```

### 刷新

控制此请求所做的更改对搜索是否可见。看 [刷新](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-refresh.html)。

### Noop更新

使用索引api更新文档时，即使文档没有更改，总是会创建新版本的文档。如果这是不可接受的使用`_update`api `detect_noop`设置为true。这个选项在索引api上不可用，因为索引api没有获取旧的源，也无法将它与新的源进行比较。

当noop更新不可接受时，并没有一个硬性规定。这是多种因素的结合，例如数据源发送实际上是否是更新的更新频率以及Elasticsearch每秒接收更新时在分片上运行的查询数量。

### 超时

当执行索引操作时，分配用于执行索引操作的主碎片可能不可用。造成这种情况的一些原因可能是主分片正在从网关进行恢复或正在进行重定位。默认情况下，索引操作将在主分片上等待最多1分钟，然后返回错误并作出响应。该`timeout`参数可以用来明确指定等待的时间。以下是将其设置为5分钟的示例：

```
curl -XPUT 'localhost:9200/twitter/tweet/1?timeout=5m&pretty' -H 'Content-Type: application/json' -d'
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
'
```
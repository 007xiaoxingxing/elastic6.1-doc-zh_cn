## `?refresh`

[Index](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)，[Update](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)，[Delete](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html)和 [Bulk](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)的API支持设置`refresh`来控制发生更改后对于搜索可见。这些是允许的参数值：

- 空字符串或 `true`

  操作发生后立即刷新相关的主要和副本碎片（而不是整个索引），以便更新的文档立即出现在搜索结果中。这应该**只**经过仔细思考和验证，它不会导致性能变差，无论从索引和搜索的角度。

- `wait_for`

  在回复之前，等待请求所做的更改通过刷新显示。这不会强制立即刷新，而是等待刷新。Elasticsearch自动刷新每个`index.refresh_interval`默认值为1秒的碎片。这个设置是 [动态的](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#dynamic-index-settings)。调用[*Refresh*](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-refresh.html) API或设置`refresh`为`true`支持它的任何API也会导致刷新，从而导致已经运行的请求`refresh=wait_for` 返回。

- `false` （默认）

  不采取刷新相关的行动。此请求所做的更改将在请求返回后的某个时间点显示出来。

### 选择使用哪个设置

除非您有充分的理由等待更改变为可见`refresh=false`，否则始终使用，或者，因为这是默认设置，请将该`refresh` 参数保留在URL之外。这是最简单和最快的选择。

如果您绝对必须使请求所做的更改与请求同步可见，那么您必须在将更多的负载加载到Elasticsearch（`true`）和等待更长的响应（`wait_for`）之间进行选择。这里有几点应该告知这个决定：

- 与索引`wait_for`相比，对索引做出的更多更改可以节省更多的工作量`true`。如果索引每次只更改一次， `index.refresh_interval`则不会保存工作。
- `true`创建效率较低的索引结构（细小的分段），这些结构稍后必须合并成更高效的索引结构（更大的分段）。这意味着`true`在索引时支付成本来创建微小的segment，在搜索的时候搜索细小的segment，并且在合并的时候创造更大的segment。
- 永远不要`refresh=wait_for`连续启动多个请求。取而代之的是将它们批量化为一个批量请求，`refresh=wait_for`而Elasticsearch将并行启动它们，并且只有在它们全部完成后才会返回。
- 如果刷新间隔设置为`-1`禁用自动刷新，则请求`refresh=wait_for`将无限期地等待，直到某个操作导致刷新。相反，设置`index.refresh_interval`比默认值短的东西`200ms`会使得`refresh=wait_for`返回速度更快，但是它仍然会产生效率低下的部分。
- `refresh=wait_for`只影响到它的请求，但是，通过立即强制刷新，`refresh=true`会影响其他正在进行的请求。一般来说，如果你有一个运行的系统，你不希望打扰，那么 `refresh=wait_for`是一个较小的修改。

### `refresh=wait_for`可以强制刷新

如果有一个`refresh=wait_for`请求到来 `index.max_refresh_listeners`（默认为1000）请求等待在该分片上刷新，则该请求的行为就像`refresh`设置为`true`：它会强制刷新。这保证了当一个 `refresh=wait_for`请求返回时，其更改对于搜索是可见的，同时阻止对被阻止的请求进行未经检查的资源使用。如果一个请求因为侦听器插槽用完而强制刷新，那么它的响应将包含`"forced_refresh": true`。

无论修改碎片多少次，批量请求在每个碎片上只占用一个插槽。

### 示例

这些将创建一个文件，并立即刷新索引，因此它是可见的：

```
curl -XPUT 'localhost:9200/test/test/1?refresh&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
curl -XPUT 'localhost:9200/test/test/2?refresh=true&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'

```

这些将创建一个document，而不做任何事情使其可见的搜索：

```
curl -XPUT 'localhost:9200/test/test/3?pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
curl -XPUT 'localhost:9200/test/test/4?refresh=false&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'

```



这将创建一个文档，并等待它成为可见的搜索：

```
curl -XPUT 'localhost:9200/test/test/4?refresh=wait_for&pretty' -H 'Content-Type: application/json' -d'
{"test": "test"}
'
```
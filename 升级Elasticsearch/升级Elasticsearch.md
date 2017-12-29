# 升级Elasticsearch

Elasticsearch通常可以使用[滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html) 过程进行升级，因此升级不会中断服务。但是，您可能需要使用[Reindex来升级](https://www.elastic.co/guide/en/elasticsearch/reference/current/reindex-upgrade.html)在旧版本中创建的索引。在6.0之前的主要版本之间进行升级需要[完全重启群集](https://www.elastic.co/guide/en/elasticsearch/reference/current/restart-upgrade.html)。

升级到新版本的Elasticsearch时，需要升级Elastic Stack中的每个产品。升级所需的步骤因所使用的产品而异。想要一个为您的堆栈量身定制的列表吗？试用我们的[交互式升级指南](https://www.elastic.co/products/upgrade_guide)。有关升级堆栈的更多信息，请参阅[升级弹性堆栈](http://www.elastic.co/guide/en/elastic-stack/6.1)。

![重要](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/important.png)

在升级Elasticsearch之前：

- 查看影响您应用程序的更改的[重大更改](https://www.elastic.co/guide/en/elasticsearch/reference/current/breaking-changes.html)。
- 检查[弃用日志](https://www.elastic.co/guide/en/elasticsearch/reference/current/logging.html#deprecation-logging)，看看您是否使用任何弃用的功能。
- 如果您使用自定义插件，请确保兼容版本可用。
- 升级生产群集之前，在开发环境中测试升级。
- 升级前[备份您的数据](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html)。你**不能回退**，除非你有你的数据备份到一个较早的版本。

下表显示何时可以执行滚动升级，什么时候需要重新索引或删除旧索引，以及何时需要完全重新启动群集。

| 升级                                       | 升级到   | 支持的升级类型                                  |
| ---------------------------------------- | ----- | ---------------------------------------- |
| `5.x`                                    | `5.y` | [滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html)（在哪里`y > x`） |
| `5.6`                                    | `6.x` | [滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html) [[a\]](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html#reindexfn) |
| `5.0-5.5`                                | `6.x` | [全集群重启](https://www.elastic.co/guide/en/elasticsearch/reference/current/restart-upgrade.html) [[a\]](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html#ftn.reindexfn) |
| `<5.x`                                   | `6.x` | [重建索引来升级](https://www.elastic.co/guide/en/elasticsearch/reference/current/reindex-upgrade.html) |
| `6.x`                                    | `6.y` | [滚动升级](https://www.elastic.co/guide/en/elasticsearch/reference/current/rolling-upgrades.html)（在哪里`y > x`）[[b\]](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html#id-1.4.2.5.1.5.5.3.1.3) |
| [[a\]](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html#reindexfn)在升级之前，您必须删除或重新索引在2.x中创建的任何索引。[[b\]](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html#id-1.4.2.5.1.5.5.3.1.3)从6.0.0之前的GA版本升级需要完整的群集重启。 |       |                                          |

![重要](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/important.png)

Elasticsearch可以读取在**以前的主要版本中**创建的索引。旧的索引必须重新编制索引或删除。Elasticsearch 6.x可以使用在Elasticsearch 5.x中创建的索引，但不能使用在Elasticsearch 2.x或之前创建的索引。Elasticsearch 5.x可以使用在Elasticsearch 2.x中创建的索引，但不能使用在1.x或之前创建的索引。

这也适用于使用[快照和还原进行](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-snapshots.html)备份的索引。如果索引最初是在2.x中创建的，即使快照是由5.x集群创建的，也不能恢复到6.x集群。

如果不兼容的索引存在，Elasticsearch节点将无法启动。

有关如何升级旧索引的信息，请参阅[Reindex升级](https://www.elastic.co/guide/en/elasticsearch/reference/current/reindex-upgrade.html)。


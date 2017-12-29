## 重要的Elasticsearch配置

虽然Elasticsearch只需要很少的配置，但有一些需要手动配置的设置，在投入生产之前一定要进行配置。

- [`path.data` 和 `path.logs`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings)
- [`cluster.name`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#cluster.name)
- [`node.name`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#node.name)
- [`bootstrap.memory_lock`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#bootstrap.memory_lock)
- [`network.host`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#network.host)
- [`discovery.zen.ping.unicast.hosts`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#unicast.hosts)
- [`discovery.zen.minimum_master_nodes`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#minimum_master_nodes)
- [JVM堆转储路径](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#heap-dump-path)

### `path.data`和`path.logs`

如果您正在使用`.zip`或`.tar.gz`存档，`data`和`logs` 目录是`$ES_HOME`的子文件夹。如果这些重要文件夹保留在其默认位置，则在将Elasticsearch升级到新版本时，这些文件夹被删除的风险很高。

在生产使用中，几乎都是要更改数据和日志文件夹的位置：

```
path:
  logs: /var/log/elasticsearch
  data: /var/data/elasticsearch
```

RPM和Debian发行版已经使用自定义路径，`data`和 `logs`。

`path.data`设置可以被设置为多条路径，在这种情况下，所有的路径将被用于存储数据（虽然属于单个碎片文件将全部存储相同的数据路径上）：

```
path:
  data:
    - /mnt/elasticsearch_1
    - /mnt/elasticsearch_2
    - /mnt/elasticsearch_3
```

### `cluster.name`

节点只能`cluster.name`在与群集中的所有其他节点共享群集时才能加入群集。默认名称是`elasticsearch`，但您应将其更改为描述集群用途的适当名称。

```
cluster.name：logging-prod
```

确保不要在不同的环境中重复使用相同的群集名称，否则可能会导致节点加入错误的群集。

### `node.name`

默认情况下，Elasticsearch将使用随机生成的UUID的前七个字符作为节点ID。请注意，节点ID是持久的，并且在节点重新启动时不会更改，因此默认节点名称也不会更改。

值得配置一个更有意义的名字，这个名字在重启节点后也会保持：

```
node.name：prod-data-2
```

该`node.name`如下，也可以设置为服务器的主机名：

```
node.name: ${HOSTNAME}
```

### `bootstrap.memory_lock`

对于您的节点的健康状况来说，没有任何一个JVM换出到磁盘是非常重要的。实现这一目标的一种方法是将`bootstrap.memory_lock`设置设置为`true`。

要使此设置生效，需要首先配置其他系统设置。有关如何正确设置内存锁定的更多详细信息，请参阅[启用`bootstrap.memory_lock`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html#mlockall)。

### `network.host`

默认情况下，Elasticsearch只绑定到loopback地址 - 例如`127.0.0.1` 和`[::1]`。这足以在服务器上运行单个开发节点。

![小费](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/tip.png)

实际上，多个节点可以从一个节点上的相同`$ES_HOME`位置启动。这可以用于测试Elasticsearch形成集群的能力，但它不是建议用于生产的配置。

为了与其他服务器上的节点进行通信并形成集群，您的节点将需要绑定到非环回地址。虽然有许多 [网络设置](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html)，通常您只需要配置 `network.host`：

```
network.host：192.168.1.10
```

该`network.host`设置也了解一些特殊的值，比如 `_local_`，`_site_`，`_global_`和喜欢修饰`:ip4`和`:ip6`，其中的细节中可以找到[的特殊值`network.host`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html#network-interface-values)。

![重要](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/important.png)

只要您提供自定义设置`network.host`，Elasticsearch就会假定您正在从开发模式转换到生产模式，并将多个系统启动检查从警告升级到异常。请参阅[开发模式与生产模式](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html#dev-vs-prod)[编辑](https://github.com/elastic/elasticsearch/edit/6.1/docs/reference/setup/sysconfig.asciidoc)以获取更多信息。

### `discovery.zen.ping.unicast.hosts`

开箱即用，没有任何网络配置，Elasticsearch将绑定到可用的环回地址，并将扫描端口9300到9305尝试连接到运行在同一服务器上的其他节点。这提供了自动集群体验，而无需进行任何配置。

当需要与其他服务器上的节点组成群集时，您必须提供群集中其他节点的种子列表，这些节点可能是活的并且可联系的。这可以指定如下：

```
discovery.zen.ping.unicast.hosts:
   - 192.168.1.10:9300
   - 192.168.1.11 （1）
   - seeds.mydomain.com （2）
```

| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/1.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#CO8-1) | 如果未指定 ，端口将默认为`transport.profiles.default.port`并回退`transport.tcp.port`。 |
| ---------------------------------------- | ---------------------------------------- |
| [![img](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/callouts/2.png)](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#CO8-2) | 解析为多个IP地址的主机名将尝试所有解析的地址。                 |

### `discovery.zen.minimum_master_nodes`

为防止数据丢失，配置`discovery.zen.minimum_master_nodes`设置至关重要， 以便每个符合主节点的节点都知道为了形成集群而必须可见*的主*节点的*最小数量*。

如果没有这个设置，那么遭受网络故障的集群就有可能将集群分成两个独立的集群 - 分裂的大脑 - 这将导致数据丢失。在[Avoiding split brain with`minimum_master_nodes`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html#split-brain)提供了更详细的解释。

为了避免脑裂，应将此设置设置为符合主数据节点的*法定人数*：

```
（master_eligible_nodes / 2）+ 1
```

换句话说，如果有三个主要节点，则最小主节点应设置为`(3 / 2) + 1`或`2`：

```
discovery.zen.minimum_master_nodes：2minimum_master_nodes：2
```

### JVM heap转储路径

该[RPM](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)和[Debian的](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html)软件包分发默认配置JVM倾倒在内存不足异常给堆 `/var/lib/elasticsearch`。如果该路径不适合用于存储堆转储，则应修改条目`-XX:HeapDumpPath=/var/lib/elasticsearch`中 [`jvm.options`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#jvm-options)到备用路径。如果你指定一个文件名而不是一个目录，JVM将重复使用同一个文件; 这是防止堆转储堆积在堆转储路径中的一种机制。或者，您可以通过操作系统配置计划任务，以删除比配置的时间早的堆转储。

请注意，归档分发默认情况下不配置堆转储路径。相反，JVM将默认转储到Elasticsearch进程的工作目录。如果你想配置一个堆转储路径，您应该修改条目`#-XX:HeapDumpPath=/heap/dump/path`中 [`jvm.options`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#jvm-options)删除注释标记`#`，并指定一个实际的路径。
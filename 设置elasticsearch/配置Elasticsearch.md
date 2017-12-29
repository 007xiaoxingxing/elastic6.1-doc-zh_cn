## 配置Elasticsearch

Elasticsearch提供良好的默认设置，只需要很少的配置。使用[*集群更新设置*](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-update-settings.html) API，可以在运行的集群上更改大多数设置 。

配置文件应包含特定于节点的设置（例如`node.name`和路径），或节点为了能够加入集群而需要的设置，例如`cluster.name`和`network.host`。

### 配置文件位置

Elasticsearch有三个配置文件：

- `elasticsearch.yml` 用于配置Elasticsearch
- `jvm.options` 用于配置Elasticsearch JVM设置
- `log4j2.properties` 用于配置Elasticsearch日志记录

这些文件位于config目录中，其默认位置取决于安装来自归档分发（`tar.gz`或 `zip`）还是软件包分发（Debian或RPM软件包）。

对于压缩包解压安装，config目录位置默认为 `$ES_HOME/config`。config目录的位置可以通过`ES_PATH_CONF`环境变量进行更改 ，如下所示：

```
ES_PATH_CONF=/path/to/my/config ./bin/elasticsearch
```

或者，您可以通过命令行或通过shell配置文件来`export`ES_PATH_CONF环境变量。

对于使用软件包安装，config目录位置默认为 `/etc/elasticsearch`。config目录的位置也可以通过`ES_PATH_CONF`环境变量来改变，但是请注意，在你的shell中设置是不够的。相反，这个变量来源于`/etc/default/elasticsearch`（对于Debian软件包）和 `/etc/sysconfig/elasticsearch`（对于RPM软件包）。您将需要相应地编辑`ES_PATH_CONF=/etc/elasticsearch`中的配置来更改config目录位置。

### 配置文件格式

配置格式是[YAML](http://www.yaml.org/)。以下是更改数据和日志目录路径的示例：

```
path:
    data: /var/lib/elasticsearch
    logs: /var/log/elasticsearch
```

设置也可以如下展开：

```
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

### 环境变量替换

使用`${...}`配置文件中的符号引用的环境变量将被替换为环境变量的值，例如：

```
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

### 提示设置

对于不希望存储在配置文件中的设置，可以使用该值`${prompt.text}`或`${prompt.secret}`在前台启动Elasticsearch。`${prompt.secret}`已禁用回声，以便输入的值不会显示在您的终端中; `${prompt.text}`将允许您在输入时看到该值。例如：

```
node:
  name: ${prompt.text}
```

启动Elasticsearch时，系统会提示您输入实际值，如下所示：

```
Enter value for [node.name]:
```

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

如果 在设置中使用`${prompt.text}`或者`${prompt.secret}`在设置中使用Elasticsearch，那么Elasticsearch将不会启动，并且该过程作为服务运行或在后台运行。
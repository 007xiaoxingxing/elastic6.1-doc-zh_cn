## 安装Elasticsearch

Elasticsearch以下列软件包格式提供：

| `zip`/`tar.gz` | 在`zip`和`tar.gz`包适合安装在任何系统上，并且是入门Elasticsearch在大多数系统中最容易的选择。[使用`.zip`或`.tar.gz`](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-targz.html)[安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html)或[在Windows上](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html)[安装Elasticsearch`.zip`](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html) |
| -------------- | ---------------------------------------- |
| `deb`          | 该`deb`软件包适用于Debian，Ubuntu和其他基于Debian的系统。Debian软件包可以从Elasticsearch网站下载，也可以从我们的Debian仓库下载。[用Debian软件包安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html) |
| `rpm`          | 该`rpm`软件包适合安装在Red Hat，Centos，SLES，OpenSuSE和其他基于RPM的系统上。RPM可以从Elasticsearch网站或从我们的RPM存储库下载。[用RPM安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html) |
| `msi`          | 该`msi`软件包适用于至少安装了.NET 4.5框架的Windows 64位系统上的安装，并且是在Windows上开始使用Elasticsearch的最简单的选择。MSI可以从Elasticsearch网站下载。[使用Windows MSI安装程序安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/windows.html) |
| `docker`       | Dokcer镜像可用于在Docker容器中安装Elasticsearch。它们可以从Elastic Docker Registry中下载。默认镜像附带 预装的[X-Pack](https://www.elastic.co/guide/en/x-pack/6.1/index.html)。[用Docker安装Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/docker.html) |

### 配置管理工具

我们还提供以下配置管理工具来帮助批量化部署：

| Puppet  | [puppet-elasticsearch](https://github.com/elastic/puppet-elasticsearch) |
| ------- | ---------------------------------------- |
| Chef    | [cookbook-elasticsearch](https://github.com/elastic/cookbook-elasticsearch) |
| Ansible | [ansible-elasticsearch](https://github.com/elastic/ansible-elasticsearch) |



## 用Docker安装Elasticsearch

Elasticsearch也可用作Docker镜像。这些镜像使用[centos：7](https://hub.docker.com/_/centos/)作为基本镜像，并且可以使用[X-Pack](https://www.elastic.co/guide/en/x-pack/6.1/xpack-introduction.html)。

所有发布的Docker镜像和标签列表可以在[www.docker.elastic.co](https://www.docker.elastic.co/)找到。源代码可以在[GitHub](https://github.com/elastic/elasticsearch-docker/tree/6.1)上[找到](https://github.com/elastic/elasticsearch-docker/tree/6.1)。

### 镜像类型

镜像有三种不同的配置或“风味”。`basic`版本随附有预装的X-Pack Basic功能，并通过免费许可证自动激活。`platinum` 版本含有一个30天的X-Pack全功能试用许可证。`oss` 不包括X-Pack，并且只包含开源Elasticsearch。

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

`platinum` 镜像中启用了[X-Pack Security](https://www.elastic.co/guide/en/x-pack/6.1/xpack-security.html)。要访问群集，有必要为`elastic`用户设置一个初始密码 。初始密码可以通过`ELASTIC_PASSWORD`环境变量在启动时设置 ：

```
docker run -e ELASTIC_PASSWORD=MagicWord docker.elastic.co/elasticsearch/elasticsearch-platinum:6.1.1
```

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

该`platinum`镜像包含30天的试用许可证。之后，您可以获取其中一个[available subscriptions](https://www.elastic.co/subscriptions)或还原为基本许可证。基本许可证是免费的，并包含一部分X-Pack功能。

获取Docker的Elasticsearch就像发出`docker pull`针对Elastic Docker registry的命令一样简单。

首先安装个docker先，我的系统是Ubuntu16.04

```shell
 wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_17.12.0~ce-0~ubuntu_amd64.deb
 dpkg -i docker-ce_17.12.0~ce-0~ubuntu_amd64.deb
```



可以使用以下命令检索Docker映像：

```shell
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.1.1
docker pull docker.elastic.co/elasticsearch/elasticsearch-platinum:6.1.1
docker pull docker.elastic.co/elasticsearch/elasticsearch-oss:6.1.1
```

### 从命令行运行Elasticsearch

#### 开发模式

Elasticsearch可以通过以下命令快速启动以进行开发或测试：

```shell
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.1.1

```

#### 生产模式

![重要](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/important.png)

该`vm.max_map_count`内核设置需要至少设置为`262144`用于生产。取决于您的平台：

- Linux的

  该`vm.max_map_count`设置应该在/etc/sysctl.conf中永久设置：

  ```
  $ grep vm.max_map_count /etc/sysctl.conf。max_map_count / etc / sysctl 。CONF
  vm.max_map_count = 262144。max_map_count = 262144
  ```

  要在实时系统类型上应用设置 `sysctl -w vm.max_map_count=262144`

- 与[Docker for Mac](https://docs.docker.com/engine/installation/mac/#/docker-for-mac) macOS

  该`vm.max_map_count`设置必须在xhyve虚拟机中设置：

  ```shell
  $ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
  ```

  用*root*登录并且没有密码。然后`sysctl`像为Linux一样配置设置：

  ```
  sysctl -w vm.max_map_count = 262144- w vm 。max_map_count = 262144
  ```

- Windows和MacOS在[Docker Toolbox](https://www.docker.com/products/docker-toolbox)上设置

  该`vm.max_map_count`设置必须通过docker-machine来设置：

  ```
  docker-machine ssh
  sudo sysctl -w vm.max_map_count=262144
  ```

以下示例显示了一个包含两个Elasticsearch节点的集群。打开集群，使用 [`docker-compose.yml`](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-prod-cluster-composefile)，只需键入：

```
docker-compose up
```

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

`docker-compose`没有在Linux上的Docker中预装。可以在[Docker Compose网页](https://docs.docker.com/compose/install/#install-using-pip)上找到有关安装的说明 。

节点`elasticsearch`侦听`localhost:9200`，而`elasticsearch2` 与`elasticsearch`通过docker网络通信。

这个例子也使用 [Docker named volumes](https://docs.docker.com/engine/tutorials/dockervolumes)，`esdata1`和`esdata2`若不存在将被创建。

`docker-compose.yml`：

```yml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.1.1
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.1.1
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local

networks:
  esnet:
```

要停止集群，请输入`docker-compose down`。数据卷将被保持不变，因此可以使用docker-compose up再次使用相同的数据启动群集。要销毁集群**和数据卷**，只需输入 `docker-compose down -v`。

#### 检查集群的状态：

```
curl http://127.0.0.1:9200/_cat/health
1472225929 15:38:49 docker-cluster green 2 2 4 2 0 0 0 0 - 100.0%       
```

日志消息转到控制台，由配置的Docker日志记录驱动程序处理。默认情况下，您可以使用`docker logs`访问日志。

### 用Docker配置Elasticsearch

Elasticsearch从下面的文件加载它的配置`/usr/share/elasticsearch/config/`。[*配置Elasticsearch*](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)和[设置JVM选项](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#jvm-options)中记录了这些配置文件。

镜像提供了几种配置Elasticsearch设置的方法，传统的方法是提供定制的文件，也就是说`elasticsearch.yml`。也可以使用环境变量来设置选项：

#### A.通过Docker环境变量呈现参数

例如，要定义群集名称，`docker run`您可以指定 `-e "cluster.name=mynewclustername"`。双引号是必需的。

#### B.绑定安装配置

创建您的自定义配置文件，并将其安装在镜像的相应文件上。例如绑定`custom_elasticsearch.yml`可通过`docker run`加参数来完成：

```
-v full_path_to/custom_elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```

![重要](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/important.png)

容器使用elasticsearch用户，uid:gid 1000:1000来运行Elasticsearch。绑定主机目录和文件，比如`custom_elasticsearch.yml`，这个文件 需要被这个用户访问。对于[数据和日志文件](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings)，例如`/usr/share/elasticsearch/data`，需要具备写权限。另请参阅下面的注1。

#### C.定制的镜像

在某些环境中，准备包含您的配置的自定义图像可能更有意义。写一个 `Dockerfile`来实现这一点也很简单：

```
FROM docker.elastic.co/elasticsearch/elasticsearch:6.1.1
COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/
```

然后，您可以使用类似的方式构建并运行镜像：

```
docker build --tag=elasticsearch-custom .
docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
```

#### D.覆盖镜像的默认[CMD](https://docs.docker.com/engine/reference/run/#cmd-default-command-or-options)

通过覆盖镜像的默认命令，可以将选项作为命令行选项传递给Elasticsearch进程。例如：

```
docker run <various parameters> bin/elasticsearch -Ecluster.name=mynewclustername
```

### 使用Elasticsearch Docker镜像配置SSL / TLS

请参阅[在Elasticsearch Docker映像中加密通信](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls-docker.html)。

### 生产使用注意事项和默认值

我们已经收集了一些生产用途的最佳实践。下面提到的任何Docker参数都假设使用了`docker run`。

1. 默认情况下，Elasticsearch在容器内部以`elasticsearch`用户运行，使用uid:gid`1000:1000`。

   ![警告](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/caution.png)

   [Openshift](https://docs.openshift.com/container-platform/3.6/creating_images/guidelines.html#openshift-specific-guidelines)是一个例外 ，它使用任意分配的用户ID来运行容器。Openshift将呈现持续的卷，而gid `0`将会在没有任何调整的情况下工作。

   如果要绑定本地目录或文件，请确保该用户可读，而[数据和日志目录](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings)另外需要写入权限。一个好的策略是授予组访问gid `1000`或`0`本地目录。作为一个例子，要准备一个本地目录来存储通过bind-mount的数据：

   ```
   mkdir esdatadir
   chmod g + rwx esdatadir
   chgrp 1000 esdatadir
   ```

   作为最后的手段，您还可以强制容器通过环境变量来更改用于[数据和日志文件](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#path-settings)的任何绑定挂载的所有权`TAKE_FILE_OWNERSHIP`。在这种情况下，它们将由uid:gid`1000:0`拥有，并根据需要提供对Elasticsearch进程的读/写访问。

2. 确保[nofile](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html) 和[nproc](https://www.elastic.co/guide/en/elasticsearch/reference/current/max-number-threads-check.html)可用于Elasticsearch容器的ulimits是非常重要的。验证 Docker守护程序的 [init系统](https://github.com/moby/moby/tree/ea4d1243953e6b652082305a9c3cda8656edab26/contrib/init)是否已经将这些值设置为可接受的值，如果需要，在守护进程中调整它们，或者在每个容器中覆盖它们，例如使用`docker run`：

   ```
   --ulimit nofile = 65536：65536
   ```

   ![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

   检查Docker守护进程的一种方法是默认的上述ulimits是运行：

   ```
   docker run --rm centos:7 /bin/bash -c 'ulimit -Hn && ulimit -Sn && ulimit -Hu && ulimit -Su'
   ```

3. 为了性能和节点稳定性需要禁用swap。这可以通过[Elasticsearch文档中](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html)提到的任何方法来实现 。如果你选择了这个 `bootstrap.memory_lock: true`方法，除了通过任何一种[配置方法](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-configuration-methods)`memlock: true`来定义之外，还需要在[Docker Daemon中](https://docs.docker.com/engine/reference/commandline/dockerd/#default-ulimits)定义的ulimit 或专门为容器设置的ulimit 。这在[docker-compose.yml中](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-prod-cluster-composefile)已经得到了证明 。如果使用`docker run`：

   ```
   -e "bootstrap.memory_lock=true" --ulimit memlock=-1:-1
   ```

4. 该镜像[暴露](https://docs.docker.com/engine/reference/builder/#/expose) TCP端口9200和9300.对于集群，建议随机化发布的端口`--publish-all`，除非您为每个主机固定一个容器。

5. 使用`ES_JAVA_OPTS`环境变量来设置堆大小。例如，使用16GB的使用`-e ES_JAVA_OPTS="-Xms16g -Xmx16g"` 在`docker run`时加上该参数。

6. 运行特定版本的Elasticsearch Docker镜像。例如`docker.elastic.co/elasticsearch/elasticsearch:6.1.1`。

7. `/usr/share/elasticsearch/data`如[生产示例中](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-prod-mode)所示， 始终使用绑定的卷，原因如下：

   1. 如果容器被杀死，你的elasticsearch节点的数据不会丢失
   2. Elasticsearch是I / O敏感的，Docker存储驱动程序对于快速I / O并不理想
   3. 它允许使用高级的 [Docker volume plugins](https://docs.docker.com/engine/extend/plugins/#volume-plugins)

8. 如果您正在使用devicemapper存储驱动程序，请确保您没有使用默认`loop-lvm`模式。将docker-engine配置为使用 [direct-lvm](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/#configure-docker-with-devicemapper) 。

9. 考虑通过使用不同的[日志记录驱动程序](https://docs.docker.com/engine/admin/logging/overview/)来集中您的日志。另外请注意，默认的json文件日志记录驱动程序不适合生产使用。
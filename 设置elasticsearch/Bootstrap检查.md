## Bootstrap检查

总的来说，我们有很多用户遭遇意外问题的经验，因为他们没有配置 [important settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)。在之前的Elasticsearch版本中，某些设置的配置错误被记录为警告。可以理解的是，用户有时会忽略掉这些日志消息。为了确保这些设置得到他们应得的关注，Elasticsearch在启动时进行Bootstrap检查。

这些Bootstrap检查各种Elasticsearch和系统设置，并将它们与Elasticsearch操作安全的值进行比较。如果Elasticsearch处于开发模式，那么失败的任何Bootstrap检查都会在Elasticsearch日志中显示为警告。如果Elasticsearch处于生产模式，则任何失败的引导检查都将导致Elasticsearch拒绝启动。

有一些引导程序检查始终强制执行，以防止Elasticsearch在不兼容的设置下运行。这些检查单独记录。

### 开发模式与生产模式

默认情况下，Elasticsearch绑定`localhost`作为[HTTP](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html)和 [transport（internal）](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-transport.html)的通信。这对Elasticsearch的downloading和playing以及日常开发都是很好的，但是对于生产系统来说却没有用处。要加入集群，必须通过传输通信访问Elasticsearch节点。要通过外部网络接口加入集群，节点必须将传输绑定到外部接口，而不是使用[single-node discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html#single-node-discovery)。因此，如果Elasticsearch节点无法通过外部网络接口与另一台计算机形成群集，并且在外部接口上可以加入群集，那么我们认为Elasticsearch节点处于开发模式，否则处于生产模式。

请注意，HTTP和transport可以通过[`http.host`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-http.html)和独立配置 [`transport.host`](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-transport.html); 这对于配置单个节点可通过HTTP进行测试而无需触发生产模式是很有用的。

### 单节点发现（single-node discovery）

我们注意到，一些用户需要将transport绑定到一个外部接口来测试他们对transport客户端的使用情况。对于这种情况，我们提供发现类型`single-node`（通过设置`discovery.type`来 配置`single-node`）; 在这种情况下，节点将选举自己的主人，不会加入任何其他节点的集群。

### 强制Bootstrap检查

如果在生产环境中运行单个节点，则可以避开引导程序检查（通过不将绑定绑定到外部接口，或通过将绑定绑定到外部接口并将发现类型设置为 `single-node`）。对于这种情况，可以通过将系统属性设置`es.enforce.bootstrap.checks`为`true` （在[设置JVM选项中](https://www.elastic.co/guide/en/elasticsearch/reference/current/setting-system-settings.html#jvm-options)设置此项，或者通过添加`-Des.enforce.bootstrap.checks=true` 到环境变量中`ES_JAVA_OPTS`）来强制执行引导程序检查。如果您处于这种特定情况，我们强烈建议您这样做。这个系统属性可以用来强制执行独立于节点配置的引导检查。



## 堆大小检查

如果JVM以不等的初始化堆大小和最大堆大小启动，则在系统使用过程中可能会因为JVM堆的大小调整而容易中断。为了避免这些调整大小的暂停，最好使用初始堆大小等于最大堆大小的JVM来启动。此外，如果[`bootstrap.memory_lock`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#bootstrap.memory_lock)启用，JVM将在启动时锁定堆的初始大小。如果初始堆大小不等于最大堆大小，则在调整大小后，将不会出现所有JVM堆都锁定在内存中的情况。要通过堆大小检查，您必须配置[堆大小](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)。

## 文件描述符检查

文件描述符是用于跟踪打开“files”的Unix结构。在Unix中，[一切都是一个文件](https://en.wikipedia.org/wiki/Everything_is_a_file)。例如，“文件”可以是物理文件，虚拟文件（例如`/proc/loadavg`）或网络套接字。Elasticsearch需要大量的文件描述符（例如，每个碎片由多个段和其他文件组成，加上到其他节点的连接等）。此引导程序检查在OS X和Linux上执行。要通过文件描述符检查，您可能必须配置[文件描述符](https://www.elastic.co/guide/en/elasticsearch/reference/current/file-descriptors.html)。

## 内存锁定检查

当JVM执行一个主要的gc时，它会触及堆的每一页。如果这些页面中的任何一个被换出到磁盘上，它们将不得不被换回到存储器中。这会导致大量的磁盘抖动，Elasticsearch更愿意用来处理请求。有几种方法可以配置系统禁止交换。一种方法是请求JVM通过`mlockall`（Unix）或虚拟锁（Windows）将内存锁定在内存中。这是通过Elasticsearch设置完成的 [`bootstrap.memory_lock`](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html#bootstrap.memory_lock)。但是，有些情况下这个设置可以传递给Elasticsearch，但Elasticsearch不能锁定堆（例如，如果`elasticsearch` 用户没有`memlock unlimited`）。该内存锁定检查验证**，如果**该`bootstrap.memory_lock`设置已启用，JVM已成功锁定堆。要通过内存锁定检查，您可能需要配置[`mlockall`](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-configuration-memory.html#mlockall)。

## 最大线程数检查

Elasticsearch通过将请求分成几个阶段执行请求，并将这些阶段交给不同的线程池执行者。Elasticsearch中有各种不同的[线程池执行](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-threadpool.html)器。因此，Elasticsearch需要创建大量线程的能力。线程检查的最大数量确保Elasticsearch进程有权在正常使用情况下创建足够的线程。此检查仅在Linux上执行。如果你在Linux上，为了传递最大数量的线程检查，你必须配置你的系统让Elasticsearch进程能够创建至少2048个线程。这可以通过`/etc/security/limits.conf` 使用`nproc`设置完成（这些操作需要使用root用户来进行）。

## 最大大小的虚拟内存检查

Elasticsearch和Lucene `mmap`非常有效地将一部分索引映射到Elasticsearch地址空间。这样可以将某些索引数据从JVM堆中删除，但在内存中可以快速访问。为了有效，Elasticsearch应该有无限的地址空间。最大大小的虚拟内存检查强制Elasticsearch进程具有无限的地址空间，并且仅在Linux上执行。要通过最大容量虚拟内存检查，您必须配置您的系统以允许Elasticsearch进程拥有无限制的地址空间。这可以通过`/etc/security/limits.conf`使用`as`设置来完成`unlimited`（这些操作需要使用root用户来进行）。

## 最大文件大小检查

作为单独分片的组件的分段文件和作为转换日志的组件的转换日志可以变大（超过几个G）。在Elasticsearch进程可以创建的文件的最大大小有限的系统上，这可能导致写入失败。因此，这里最安全的选项是最大文件大小是无限的，这是最大文件大小引导程序检查执行的。要通过最大文件检查，您必须配置您的系统以允许Elasticsearch进程能够写入无限大小的文件。这可以通过 `/etc/security/limits.conf`使用`fsize`设置来完成`unlimited`（这些操作需要使用root用户来进行）。

## 最大map数量检查

从前面的[观点来看](https://www.elastic.co/guide/en/elasticsearch/reference/current/max-size-virtual-memory-check.html)，要`mmap`有效地使用Elasticsearch，还需要能够创建许多内存映射区域。最大映射计数检查检查内核是否允许进程拥有至少262,144个内存映射区域，并仅在Linux上执行。要通过最大地图数量检查，您必须至少配置`vm.max_map_count`通过`sysctl`来设置

## 客户端JVM检查

OpenJDK派生的JVM提供了两种不同的JVM：客户端JVM和服务器JVM。这些JVM使用不同的编译器从Java字节码中生成可执行的机器码。客户端JVM针对启动时间和内存占用情况进行了调整，同时对服务器JVM进行了调整以实现最佳性能。两台虚拟机之间的性能差异可能很大。客户端JVM检查确保Elasticsearch不在客户端JVM内部运行。要通过客户端JVM检查，必须使用服务器VM启动Elasticsearch。在现代系统和操作系统上，服务器虚拟机是默认的。另外，Elasticsearch默认配置为强制服务器虚拟机。

## 使用串行收集器检查

针对不同工作负载的OpenJDK派生的JVM有各种gc。串行收集器尤其适用于单个逻辑CPU或非常小的堆，这两个堆都不适合运行Elasticsearch。与Elasticsearch一起使用串行收集器可能会破坏性能。串行收集器检查确保Elasticsearch未配置为与串行收集器一起运行。要通过串行收集器检查，不得使用串行收集器启动Elasticsearch（无论是使用JVM的缺省值还是已明确指定`-XX:+UseSerialGC`）。请注意，Elasticsearch附带的默认JVM配置将Elasticsearch配置为使用CMS收集器。

## 系统调用过滤器检查

Elasticsearch根据操作系统安装各种风格的系统调用过滤器（例如Linux上的seccomp）。安装这些系统调用过滤器是为了防止执行与分叉有关的系统调用的能力，作为针对Elasticsearch上的任意代码执行攻击的防御机制。系统调用过滤器检查确保如果系统调用过滤器被启用，则它们被成功安装。要通过系统调用过滤器检查，您必须修复系统中阻止系统调用过滤器安装（检查您的日志）的任何配置错误，或者**自行承担风险，**通过设置`bootstrap.system_call_filter`为禁用系统调用过滤器`false`。

## OnError和OnOutOfMemoryError检查

如果JVM遇到致命错误（）或 （）`OnError`，则`OnOutOfMemoryError`启用JVM选项并启用任意命令。但是，默认情况下，Elasticsearch系统调用筛选器（seccomp）已启用，并且这些筛选器可防止分叉。因此，使用or 和系统调用过滤器是不兼容的。该和 检查防止Elasticsearch从如果这两个JVM选项的使用和系统调用过滤器可启动。此检查始终执行。通过这个检查不要启用 也不; 相反，升级到Java 8u92并使用JVM标志。虽然这并不具有的全部功能，也没有，任意分叉不会被启用支持的Seccomp。`OnError``OutOfMemoryError``OnOutOfMemoryError``OnError``OnOutOfMemoryError``OnError``OnOutOfMemoryError``OnError``OnOutOfMemoryError``ExitOnOutOfMemoryError``OnError``OnOutOfMemoryError`

## 提前访问检查

OpenJDK项目提供即将发布的快照。这些版本不适合生产。早期访问检查检测这些早期访问快照。要通过此检查，您必须在JVM的发行版本上启动Elasticsearch。

## G1GC检查

已知JDK 8附带的HotSpot JVM的早期版本在启用G1GC收集器时会导致索引损坏。受影响的版本是早于JDK 8u40附带的HotSpot版本的版本。G1GC检查检测到这些HotSpot JVM的早期版本。


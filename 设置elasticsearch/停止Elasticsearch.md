## 停止Elasticsearch

Elasticsearch的有序关闭确保Elasticsearch有机会清理并关闭未处理的资源。例如，有序关闭的节点将从集群中删除自身，同步超时日志到磁盘，并执行其他相关的清理活动。您可以通过正确停止Elasticsearch来帮助确保有序关闭。

如果您将Elasticsearch作为服务运行，则可以通过安装提供的服务管理功能来停止Elasticsearch。

如果直接运行Elasticsearch，则可以通过在控制台中运行Elasticsearch或者通过发送`SIGTERM`到POSIX系统上的Elasticsearch进程来发送control-C来停止Elasticsearch 。您可以通过各种工具（例如`ps`或`jps`）获取PID以发送信号：

```
$ jps | grep Elasticsearch
Elasticsearch
```

从Elasticsearch启动日志：

```
[2016-07-07 12:26:18,908][INFO ][node                     ] [I8hydUG] version[5.0.0-alpha4], pid[15399], build[3f5b994/2016-06-27T16:23:46.861Z], OS[Mac OS X/10.11.5/x86_64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_92/25.92-b14]

```

或者通过指定一个位置在启动时写入一个PID文件（`-p <path>`）：

```
$ ./bin/elasticsearch -p /tmp/elasticsearch-pid -d
$ cat /tmp/elasticsearch-pid && echo
15516
$ kill -SIGTERM 15516
```

### 遇到致命的错误时停止

在Elasticsearch虚拟机的生命周期中，可能会出现某些致使虚拟机处于可疑状态的致命错误。这种致命错误包括内存不足错误，虚拟机中的内部错误以及严重的I / O错误。

当Elasticsearch检测到虚拟机遇到这样的致命错误时，Elasticsearch将尝试记录错误，然后暂停虚拟机。当Elasticsearch发起这样的关闭时，它不会像上面描述的那样经过有序的关闭。Elasticsearch进程也将返回一个特殊的状态码，指明错误的性质。

| JVM内部错误    | 128  |
| ---------- | ---- |
| 内存不足错误     | 127  |
| 堆栈溢出错误     | 126  |
| 未知的虚拟机错误   | 125  |
| 严重的I / O错误 | 124  |
| 未知的致命错误    | 1    |
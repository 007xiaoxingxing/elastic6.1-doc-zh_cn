# 设置Elasticsearch

本节包含有关如何设置Elasticsearch并使其运行的信息，其中包括：

- 下载
- 安装
- 开始
- 配置

## 支持的平台

正式支持的操作系统和JVM的[列表](https://www.elastic.co/support/matrix)可在此处获得： [支持列表](https://www.elastic.co/support/matrix)。Elasticsearch在列出的平台上进行了测试，但它也可能在其他平台上工作。

## Java（JVM）版本

Elasticsearch是使用Java构建的，并且至少需要 [Java 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)才能运行。只有Oracle的Java和OpenJDK被支持。所有Elasticsearch节点和客户端都应该使用相同的JVM版本。

我们建议安装Java版本**1.8.0_131或更高版本**。如果使用未知的Java版本，Elasticsearch将拒绝启动。

Elasticsearch将使用的Java版本可以通过设置`JAVA_HOME`环境变量进行配置。
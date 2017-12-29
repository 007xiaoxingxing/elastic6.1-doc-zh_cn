# 文档API

本节首先简要介绍Elasticsearch的[数据复制模型](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html)，然后详细描述以下CRUD API：

**单个文档API**

- [*索引API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)
- [*获取API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html)
- [*删除API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete.html)
- [*更新API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html)

**多文档API**

- [*多获取API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-get.html)
- [*批量API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html)
- [*通过查询API删除*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html)
- [*通过查询API更新*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update-by-query.html)
- [*Reindex API*](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html)

![注意](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/icons/note.png)

所有的CRUD API都是单一索引的API。该`index`参数接受单个索引名称，或者`alias`指向单个索引。
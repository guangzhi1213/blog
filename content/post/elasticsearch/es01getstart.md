---
title: "Es01getstart"
date: 2021-04-21T18:06:40+08:00
lastmod: 2021-04-21T18:06:40+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# 探索集群 入门

## REST API

现在我们已经启动并运行了节点（和集群），下一步是了解如何与之通信。幸运的是，Elasticsearch提供了一个非常全面和强大的REST API，您可以使用它与集群进行交互。使用API​​可以完成的一些事项如下：

* 检查群集，节点和索引运行状况，状态和统计信息
* 管理您的群集，节点和索引数据和元数据
* 对索引执行CRUD（创建，读取，更新和删除）和搜索操作
* 执行高级搜索操作，例如分页，排序，过滤，脚本编写，聚合等等

### 集群健康

让我们从基本运行状况检查开始，我们可以使用它来查看集群的运行情况。我们将使用curl来执行此操作，但您可以使用任何允许您进行HTTP / REST调用的工具。假设我们仍然在我们启动Elasticsearch的同一节点上打开另一个命令shell窗口。

要检查群集运行状况，我们将使用 `catAPI` 。您可以 通过单击“查看控制台”或单击下面的“COPY AS CURL”链接并将其粘贴到终端中，在Kibana控制台中运行以下命令curl。

`GET /_cat/health?v`

`curl -X GET "localhost:9200/_cat/health?v`

并回应：

```shell
epoch      timestamp cluster       status node.total node.data shards pri relo init unassign pending_tasks max_task_wait_time active_shards_percent
1475247709 17:01:49  elasticsearch green           1         1      0   0    0    0        0             0                  -                100.0%
```

我们可以看到名为“elasticsearch”的群集处于绿色状态。

每当我们要求群集健康时，我们要么获得绿色，黄色或红色。

* 绿色 - 一切都很好（集群功能齐全）
* 黄色 - 所有数据都可用，但尚未分配一些副本（群集功能齐全）
* 红色 - 某些数据由于某种原因不可用（群集部分功能）

注意：当群集为红色时，它将继续提供来自可用分片的搜索请求，但您可能需要尽快修复它，因为存在未分配的分片。

同样从上面的响应中，我们可以看到总共1个节点，并且我们有0个分片，因为我们还没有数据。请注意，由于我们使用默认群集名称（elasticsearch），并且由于Elasticsearch默认使用单播网络发现来查找同一台计算机上的其他节点，因此您可能会意外启动计算机上的多个节点并拥有它们所有人都加入一个集群。在这种情况下，您可能会在上面的响应中看到多个节点。

我们还可以获得群集中的节点列表，如下所示：

`GET /_cat/nodes?v`

`curl -X GET "localhost:9200/_cat/nodes?v"`

并回应：

```shell
ip        heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
127.0.0.1           10           5   5    4.46                        mdi      *      PB2SGZY
```

在这里，我们可以看到一个名为“PB2SGZY”的节点，它是我们集群中当前的单个节点。

### 列出所有索引

现在让我们来看看我们的索引：

`GET /_cat/indices?v`

`curl -X GET "localhost:9200/_cat/indices?v"`

并回应：

`health status index uuid pri rep docs.count docs.deleted store.size pri.store.size`

这仅仅意味着我们在集群中还没有索引。

### 创建索引

现在让我们创建一个名为“customer”的索引，然后再次列出所有索引：

```es
PUT /customer?pretty
GET /_cat/indices?v
```

```shell
curl -X PUT "localhost:9200/customer?pretty"
curl -X GET "localhost:9200/_cat/indices?v"
```

第一个命令使用PUT动词创建名为“customer”的索引。我们只是追加pretty到调用的末尾，告诉它打印JSON响应（如果有的话）。

并回应：

```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   1   1          0            0       260b           260b
```
第二个命令的结果告诉我们，我们现在有一个名为customer的索引，它有一个主分片和一个副本（默认值），它包含零文档。

您可能还注意到客户索引标记了黄色运行状况。回想一下我们之前的讨论，黄色表示某些副本尚未（尚未）分配。此索引发生这种情况的原因是因为默认情况下Elasticsearch为此索引创建了一个副本。由于我们目前只有一个节点在运行，因此在另一个节点加入集群的稍后时间点之前，尚无法分配一个副本（用于高可用性）。将该副本分配到第二个节点后，此索引的运行状况将变为绿色。

### 查询和索引文档

现在让我们在客户索引中加入一些内容。我们将一个简单的客户文档索引到客户索引中，ID为1，如下所示：

```es
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

```shell
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "John Doe"
}
'
```

回应:

```json
{
  "_index" : "customer",
  "_type" : "_doc",
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

从上面可以看出，在客户索引中成功创建了一个新的客户文档。该文档还具有我们在索引时指定的内部标识1。

值得注意的是，Elasticsearch不需要在将文档编入索引之前先显式创建索引。在前面的示例中，如果客户索引事先尚未存在，则Elasticsearch将自动创建客户索引。

我们现在检索刚刚编入索引的文档：

`GET /customer/_doc/1?pretty`

`curl -X GET "localhost:9200/customer/_doc/1?pretty"`

并回应：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 25,
  "_primary_term" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```

除了字段之外，这里没有任何异常found，声明我们找到了一个具有请求的ID 1和另一个字段的_source文档，它返回我们从上一步索引的完整JSON文档。

### 删除索引

现在让我们删除刚刚创建的索引，然后再次列出所有索引：

```es
DELETE / customer？漂亮
GET / _cat / indices？v
```

`curl -X DELETE "localhost:9200/customer?pretty"
curl -X GET "localhost:9200/_cat/indices?v"
`

并回应：

`health status index uuid pri rep docs.count docs.deleted store.size pri.store.size`

这意味着索引已成功删除，现在我们回到了我们在集群中没有任何内容的地方。

在我们继续之前，让我们再仔细看看到目前为止我们学到的一些API命令：

```es
PUT /customer
PUT /customer/_doc/1
{
  "name": "John Doe"
}
GET /customer/_doc/1
DELETE /customer
```

如果我们仔细研究上述命令，我们实际上可以看到我们如何在Elasticsearch中访问数据的模式。该模式可归纳如下：

`<HTTP Verb> /<Index>/<Endpoint>/<ID>`

这种REST访问模式在所有API命令中都非常普遍，如果您能够简单地记住它，您将在掌握Elasticsearch方面有一个良好的开端。

### 修改数据

Elasticsearch几乎实时提供数据操作和搜索功能。默认情况下，从索引/更新/删除数据到搜索结果中显示的时间，您可能会有一秒钟的延迟（刷新间隔）。这是与SQL等其他平台的重要区别，其中数据在事务完成后立即可用。

#### 索引/替换文档

我们之前已经看到了如何索引单个文档。让我们再次回忆一下这个命令：

```es
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

同样，上面将指定的文档索引到客户索引中，ID为1.如果我们再使用不同（或相同）的文档执行上述命令，Elasticsearch将替换（即重新索引）新文档。 ID为1的现有ID：

```es
PUT /customer/_doc/1?pretty
{
  "name": "Jane Doe"
}
```

以上内容将ID为1的文档名称从“John Doe”更改为“Jane Doe”。另一方面，如果我们使用不同的ID，则将索引新文档，并且索引中已有的现有文档保持不变。

```es
PUT /customer/_doc/2?pretty
{
  "name": "Jane Doe"
}
```

以上索引ID为2的新文档。

索引时，ID部分是可选的。如果未指定，Elasticsearch将生成随机ID，然后使用它来索引文档。Elasticsearch生成的实际ID（或前面示例中显式指定的内容）将作为索引API调用的一部分返回。

此示例显示如何在没有显式ID的情况下索引文档：

```es
POST /customer/_doc?pretty
{
  "name": "Jane Doe"
}
```

请注意，在上面的情况中，我们使用POST动词而不是PUT，因为我们没有指定ID。

#### 更新文档

除了能够索引和替换文档，我们还可以更新文档。请注意，Elasticsearch实际上并没有在引擎盖下进行就地更新。每当我们进行更新时，Elasticsearch都会删除旧文档，然后一次性对应用了更新的新文档编制索引。

此示例显示如何通过将名称字段更改为“Jane Doe”来更新以前的文档（ID为1）：

```es
POST /customer/_update/1?pretty
{
  "doc": { "name": "Jane Doe" }
}
```

此示例显示如何通过将名称字段更改为“Jane Doe”来更新我们以前的文档（ID为1），同时为其添加年龄字段：

```es
POST /customer/_update/1?pretty
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
```

也可以使用简单脚本执行更新。此示例使用脚本将年龄增加5：

```es
POST /customer/_update/1?pretty
{
  "script" : "ctx._source.age += 5"
}
```

在上面的示例中，ctx._source指的是即将更新的当前源文档。

Elasticsearch提供了在给定查询条件（如SQL UPDATE-WHERE语句）的情况下更新多个文档的功能。请参阅 [docs-update-by-queryAPI](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/docs-update-by-query.html)

#### 删除文档

删除文档非常简单。此示例显示如何删除ID为2的以前的客户：

`DELETE /customer/_doc/2?pretty`

请参阅[_delete_by_queryAPI](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/docs-delete-by-query.html)以删除与特定查询匹配的所有文档。值得注意的是，删除整个索引而不是使用Delete By Query API删除所有文档会更有效。

#### 批处理

除了能够索引，更新和删除单个文档之外，Elasticsearch还提供了使用 [_bulkAPI](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/docs-bulk.html) 批量执行上述任何操作的功能。此功能非常重要，因为它提供了一种非常有效的机制，可以尽可能快地进行多个操作，并尽可能少地进行网络往返。

作为一个简单的示例，以下调用在一个批量操作中索引两个文档（ID 1 - John Doe和ID 2 - Jane Doe）：

```es
POST /customer/_bulk?pretty
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
```

此示例更新第一个文档（ID为1），然后在一个批量操作中删除第二个文档（ID为2）：

```es
POST /customer/_bulk?pretty
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
```

请注意，对于删除操作，之后没有相应的源文档，因为删除只需要删除文档的ID。

Bulk API不会因其中一个操作失败而失败。如果单个操作因任何原因失败，它将继续处理其后的其余操作。批量API返回时，它将为每个操作提供一个状态（按照发送的顺序），以便您可以检查特定操作是否失败。

### 探索您的数据

#### 样本数据集

现在我们已经了解了基础知识，让我们尝试更真实的数据集。我准备了一份关于客户银行账户信息的虚构JSON文档样本。每个文档都有以下架构：

```es
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```

好奇的是，这些数据是使用www.json-generator.com/生成的，因此请忽略数据的实际值和语义，因为这些都是随机生成的。

#### 加载示例数据

您可以从[此处](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)下载样本数据集（accounts.json）。将它解压缩到我们当前的目录，然后将它加载到我们的集群中，如下所示：

`curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"`

`curl "localhost:9200/_cat/indices?v"`

并回应：

```html
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank  l7sSYV2cQXmu6_4rJWVIww   5   1       1000            0    128.6kb        128.6kb
```

这意味着我们只是成功地将1000个文档批量索引到银行索引中。





#### search api

现在让我们从一些简单的搜索开始吧。有运行检索两种基本方式：一种是通过发送搜索参数REST请求URI和其他通过发送他们REST请求主体。请求体方法允许您更具表现力，并以更可读的JSON格式定义搜索。我们将尝试一个请求URI方法的示例，但是对于本教程的其余部分，我们将专门使用请求体方法。

可以从_search端点访问用于搜索的REST API 。此示例返回银行索引中的所有文档：

`GET /bank/_search?q=*&sort=account_number:asc&pretty`

让我们首先剖析搜索电话。我们正在_search银行索引中搜索（端点），该q=*参数指示Elasticsearch匹配索引中的所有文档。该sort=account_number:asc参数指示使用account_number每个文档的字段以升序对结果进行排序。该pretty参数再次告诉Elasticsearch返回漂亮的JSON结果。

响应（部分显示）：

```json
{
  "took" : 63,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
        "value": 1000,
        "relation": "eq"
    },
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "_doc",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

至于回复，我们看到以下部分：

* took - Elasticsearch执行搜索的时间（以毫秒为单位）
* timed_out - 告诉我们搜索是否超时
* _shards - 告诉我们搜索了多少个分片，以及搜索成功/失败分片的计数
* hits - 搜索结果
* hits.total - 包含与我们的搜索条件匹配的文档总数的信息的对象
  * hits.total.value- 总命中数的值（必须在上下文中解释hits.total.relation）。
  * hits.total.relation- 是否hits.total.value是确切的命中计数，在这种情况下它等于"eq"或总命中数的下限（大于或等于），在这种情况下它等于gte。
* hits.hits - 实际的搜索结果数组（默认为前10个文档）
* hits.sort - 对结果进行排序键（如果按分数排序则丢失）
* hits._score并max_score- 暂时忽略这些字段

精度hits.total由请求参数控制track_total_hits，当设置为true时，请求将准确跟踪总命中（"relation": "eq"）。默认为10,000 这意味着总命中数被准确地跟踪到10,000文档。您可以通过track_total_hits显式设置为true 来强制进行准确计数。有关详细信息，请参阅请求正文文档。

以下是使用替代请求正文方法的上述完全相同的搜索：

```json
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

这里的不同之处在于，我们不是传入q=*URI，而是向_searchAPI 提供JSON样式的查询请求体。我们将在下一节讨论这个JSON查询。

重要的是要理解，一旦您获得了搜索结果，Elasticsearch就完全完成了请求，并且不会在结果中维护任何类型的服务器端资源或打开游标。这与SQL等许多其他平台形成鲜明对比，其中您可能最初预先获得查询结果的部分子集，然后如果要获取（或翻页）其余的则必须连续返回服务器使用某种有状态服务器端游标的结果。

#### 介绍查询语句

Elasticsearch提供了一种JSON样式的特定于域的语言，可用于执行查询。这被称为查询DSL。查询语言非常全面，乍一看可能令人生畏，但实际学习它的最佳方法是从一些基本示例开始。

回到上一个例子，我们执行了这个查询：

```es
GET /bank/_search
{
  "query": { "match_all": {} }
}
```

```shell
curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} }
}
'
```

解析上面的内容，该query部分告诉我们查询定义是什么，match_all部分只是我们想要运行的查询类型。该match_all查询仅仅是在指定索引的所有文件进行搜索。

除了query参数，我们还可以传递其他参数来影响搜索结果。在上面我们传入的部分的示例中 sort，我们传入size：

```es
GET /bank/_search
{
  "query": { "match_all": {} },
  "size": 1
}
```

请注意，如果size未指定，则默认为10。

此示例执行a match_all并返回文档10到19：

```es
GET /bank/_search
{
  "query": { "match_all": {} },
  "from": 10,
  "size": 10
}
```

在from（从0开始）参数规定了从启动该文件的索引和size参数指定了多少文件，返回从参数开始的。在实现搜索结果的分页时，此功能非常有用。请注意，如果from未指定，则默认为0。

此示例执行a match_all并按帐户余额降序对结果进行排序，并返回前10个（默认大小）文档。

```es
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```

#### 执行搜索

现在我们已经看到了一些基本的搜索参数，让我们再深入研究一下Query DSL。我们先来看一下返回的文档字段。默认情况下，完整的JSON文档作为所有搜索的一部分返回。这被称为源（_source搜索命中中的字段）。如果我们不希望返回整个源文档，我们只能请求返回源中的几个字段。

此示例显示如何从搜索中返回两个字段account_number和balance（内部_source）：

```es
GET /bank/_search
{
  "query": { "match_all": {} },
  "_source": ["account_number", "balance"]
}
```

请注意，上面的示例只是简化了_source字段。它仍将只返回一个名为_source但在其中的字段，仅包含字段account_number并balance包含在内。

如果您来自SQL背景，则上面的概念与SQL SELECT FROM字段列表有些相似。

现在让我们转到查询部分。以前，我们已经看到match_all查询如何用于匹配所有文档。现在让我们介绍一个名为match查询的新查询，它可以被认为是一个基本的现场搜索查询（即针对特定字段或字段集进行的搜索）。

此示例返回编号为20的帐户：

```es
GET /bank/_search
{
  "query": { "match": { "account_number": 20 } }
}
```

此示例返回地址中包含术语“mill”的所有帐户：

```es
GET /bank/_search
{
  "query": { "match": { "address": "mill" } }
}
```

此示例返回地址中包含术语“mill”或“lane”的所有帐户：

```es
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```

此示例是match（match_phrase）的变体，它返回地址中包含短语“mill lane”的所有帐户：

```es
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

我们现在介绍一下这个bool问题。该bool查询允许我们使用布尔逻辑将较小的查询组成更大的查询。

此示例组成两个match查询并返回地址中包含“mill”和“lane”的所有帐户：

```es
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，该bool must子句指定必须为true才能将文档视为匹配的所有查询。

相反，此示例组成两个match查询并返回地址中包含“mill”或“lane”的所有帐户：

```es
GET /bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，该bool should子句指定了一个查询列表，其中任何一个都必须为true才能使文档被视为匹配。

此示例组成两个match查询并返回地址中既不包含“mill”也不包含“lane”的所有帐户：

```es
GET /bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": { "address": "mill" } },
        { "match": { "address": "lane" } }
      ]
    }
  }
}
```

在上面的示例中，该bool must_not子句指定了一个查询列表，对于要被视为匹配的文档，这些查询都不能为true。

我们可以在查询中同时组合must，should和must_not子句bool。此外，我们可以bool在任何这些bool子句中组合查询来模仿任何复杂的多级布尔逻辑。

此示例返回任何40岁但未居住在ID（aho）中的人的所有帐户：

```es
GET /bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
```

### 执行过滤

在上一节中，我们跳过了一个称为文档分数的小细节（_score搜索结果中的字段）。分数是一个数值，它是文档与我们指定的搜索查询匹配程度的相对度量。分数越高，文档越相关，分数越低，文档的相关性越低。

但是查询并不总是需要产生分数，特别是当它们仅用于“过滤”文档集时。Elasticsearch检测这些情况并自动优化查询执行，以便不计算无用的分数。

我们在上一节中介绍的bool查询还支持一些filter子句，这些子句允许我们使用查询来限制将与其他子句匹配的文档，而不会更改计算分数的方式。作为示例，让我们介绍一下range查询，它允许我们按一系列值过滤文档。这通常用于数字或日期过滤。

此示例使用bool查询返回所有余额介于20000和30000之间的帐户。换句话说，我们希望找到余额大于或等于20000且小于或等于30000的帐户。

```es
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
```

解析上面的内容，bool查询包含match_all查询（查询部分）和range查询（过滤部分）。我们可以将任何其他查询替换为查询和过滤器部分。在上面的情况下，范围查询非常有意义，因为落入范围的文档都“相同”匹配，即，没有文档比另一文档更相关。

除了match_all，match，bool，和range查询，有很多可用的其他查询类型的，我们不会进入他们在这里。由于我们已经基本了解它们的工作原理，因此将这些知识应用于学习和试验其他查询类型应该不会太困难。

#### 执行聚合

聚合提供了从数据中分组和提取统计信息的功能。考虑聚合的最简单方法是将其大致等同于SQL GROUP BY和SQL聚合函数。在Elasticsearch中，您可以执行返回匹配的搜索，同时在一个响应中返回与命中所有内容分开的聚合结果。这是非常强大和高效的，因为您可以运行查询和多个聚合，并一次性获取两个（或任一）操作的结果，避免使用简洁和简化的API进行网络往返。

首先，此示例按状态对所有帐户进行分组，然后返回按计数降序排序的前10个（默认）状态（也是默认值）：

GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}

在SQL中，上述聚合在概念上类似于：

`SELECT state, COUNT(*) FROM bank GROUP BY state ORDER BY COUNT(*) DESC LIMIT 10;`

响应（部分显示）：

```es
{
  "took": 29,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped" : 0,
    "failed": 0
  },
  "hits" : {
     "total" : {
        "value": 1000,
        "relation": "eq"
     },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "group_by_state" : {
      "doc_count_error_upper_bound": 20,
      "sum_other_doc_count": 770,
      "buckets" : [ {
        "key" : "ID",
        "doc_count" : 27
      }, {
        "key" : "TX",
        "doc_count" : 27
      }, {
        "key" : "AL",
        "doc_count" : 25
      }, {
        "key" : "MD",
        "doc_count" : 25
      }, {
        "key" : "TN",
        "doc_count" : 23
      }, {
        "key" : "MA",
        "doc_count" : 21
      }, {
        "key" : "NC",
        "doc_count" : 21
      }, {
        "key" : "ND",
        "doc_count" : 21
      }, {
        "key" : "ME",
        "doc_count" : 20
      }, {
        "key" : "MO",
        "doc_count" : 20
      } ]
    }
  }
}
```

我们可以看到ID（Idaho爱达荷州）有27个账户，其次是TX（Texas德克萨斯州）的27个账户，其次是AL（Alabama阿拉巴马州）的25个账户，依此类推。

请注意，我们设置size=0为不显示搜索匹配，因为我们只想在响应中看到聚合结果。

在前一个聚合的基础上，此示例按州计算平均帐户余额（同样仅针对按降序排序的前10个州）：

```es
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

请注意我们如何嵌套average_balance聚合内的group_by_state聚合。这是所有聚合的常见模式。您可以在聚合中任意嵌套聚合，以从数据中提取所需的轮转摘要。

在前一个聚合的基础上，我们现在按降序排列平均余额：

```es
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

此示例演示了我们如何按年龄段（20-29岁，30-39岁和40-49岁）进行分组，然后按性别进行分组，最后得到每个年龄段的平均帐户余额：

```es
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_age": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_gender": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "average_balance": {
              "avg": {
                "field": "balance"
              }
            }
          }
        }
      }
    }
  }
}
```

还有许多其他聚合功能，我们在此不再详述。如果您想进行进一步的实验，[聚合参考指南](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/search-aggregations.html)是一个很好的起点。

#### 结论

Elasticsearch既简单又复杂。到目前为止，我们已经了解了它的基础知识，如何查看它，以及如何使用一些REST API来处理它。希望本教程能让您更好地了解Elasticsearch的内容，更重要的是，启发您进一步尝试其余的强大功能！

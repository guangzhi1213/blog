---
title: "Es05apidocument"
date: 2021-04-21T18:08:54+08:00
lastmod: 2021-04-21T18:08:54+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# api 文档

本节首先简要介绍Elasticsearch的数据复制模型，然后详细介绍以下CRUD API：

## 单文档API

* 索引API
* 获取API
* 删除API
* 更新API
* 多文档API

## Multi Get API

* 批量API
* 按查询API删除
* 按查询API更新
* Reindex API

所有CRUD API都是单索引API。该index参数接受单个索引名称，或者alias指向单个索引的名称。

## 阅读和编写的文档

### 简介

Elasticsearch中的每个索引都分为碎片 ，每个碎片可以有多个副本。这些副本称为 replication group ，在添加或删除文档时必须保持同步。如果我们不这样做，从一个副本中读取将导致与从另一个副本中读取的结果截然不同的结果。保持碎片副本同步并从中提供读取的过程就是我们所说的数据复制模型。

Elasticsearch的数据复制模型基于主备份模型，并在Pacific Research的Microsoft Research 论文中得到了很好的描述 。该模型基于具有来自复制组的单个副本，该副本充当主要分片。其他副本称为副本分片。主要作为所有索引操作的主要入口点。它负责验证它们并确保它们是正确的。一旦主要接受了索引操作，主要负责将操作复制到其他副本。

本节的目的是对Elasticsearch复制模型进行高级概述，并讨论它对写入和读取操作之间的各种交互的影响。

### 基本的写模型

Elasticsearch中的每个索引操作首先使用路由解析为 replication group ，通常基于文档ID。确定复制组后，操作将在内部转发到组的当前主分片。主分片负责验证操作并将其转发到其他副本。由于副本可以脱机，因此不需要将主副本复制到所有副本。相反，Elasticsearch维护应该接收操作的分片副本列表。此列表称为同步副本并由主节点维护。顾名思义，这些是“好”分片副本的集合，保证已经处理了已经向用户确认的所有索引和删除操作。主要负责维护此不变量，因此必须将所有操作复制到此集合中的每个副本。

主分片遵循以下基本流程：

1. 验证传入操作并在结构无效时拒绝它（例如：有一个对象字段，其中包含一个数字）
1. 在本地执行操作，即索引或删除相关文档。这也将验证字段的内容并在需要时拒绝（例如：关键字值太长，无法在Lucene中进行索引）。
1. 将操作转发到当前同步副本集中的每个副本。如果有多个副本，则这是并行完成的。
1. 一旦所有副本成功执行了操作并响应主服务器，主服务器就会确认成功完成对客户端的请求。

### 故障处理

在索引编制过程中可能会出现许多问题 - 磁盘可能会损坏，节点可能会相互断开连接，或者某些配置错误可能会导致复制副本上的操作失败，尽管它在主服务器上成功。这些很少见，但主要必须回应它们。

在主服务器本身发生故障的情况下，托管主服务器的节点将向主服务器发送有关它的消息。索引操作将等待（默认情况下最多1分钟），以便主服务器将其中一个副本提升为新的主数据库。然后，该操作将被转发到新的主要处理。请注意，主服务器还会监控节点的运行状况，并可能决定主动降级主服务器。当通过网络问题将拥有主节点的节点与群集隔离时，通常会发生这种情况。有关详细信息，请参见此处

一旦在主服务器上成功执行了操作，主服务器就必须在副本分片上执行它时处理潜在的故障。这可能是由副本上的实际故障或由于网络问题导致操作无法到达副本​​（或阻止副本响应）引起的。所有这些都具有相同的最终结果：作为同步副本集的一部分的副本错过了即将被确认的操作。为了避免违反不变量，主服务器向主服务器发送消息，请求从同步副本集中删除有问题的分片。只有在主设备确认删除了碎片后，主设备才会确认操作。

在将操作转发到副本时，主服务器将使用副本来验证它仍然是活动主服务器。如果主要由于网络分区（或长GC）而被隔离，则它可能会在意识到已降级之前继续处理传入的索引操作。复制品将拒绝来自陈旧主要操作的操作。当主服务器收到来自副本的响应时拒绝其请求，因为它不再是主服务器，那么它将联系主服务器并将知道它已被替换。然后将操作路由到新主服务器。

> 如果没有复本会怎么样？

> 这是一个有效的方案，可能由于索引配置或仅因为所有副本都已失败而发生。在这种情况下，主要是处理操作而没有任何外部验证，这可能看起来有问题。另一方面，主服务器本身不能使其他分片失败，但请求主服务器代表它执行此操作。这意味着主服务器知道主服务器是唯一的单个良好副本。因此，我们保证主服务器不会将任何其他（过时的）分片副本提升为新主分区，并且任何索引到主分区的操作都不会丢失。当然，由于此时我们只使用单个数据副本运行，因此物理硬件问题可能会导致数据丢失。有关缓解选项，请参阅等待活动碎片

## 基本阅读模型

Elasticsearch中的读取可以是ID非常轻量级的查找，也可以是具有复杂聚合的大量搜索请求，这些聚合会占用非常重要的CPU能力。主备份模型的一个优点是它使所有分片副本保持一致（除了正在进行的操作）。因此，单个同步副本足以提供读取请求。

当节点收到读取请求时，该节点负责将其转发到保存相关分片的节点，整理响应并响应客户端。我们将该节点称为该请求的协调节点。基本流程如下：

* 将读取请求解析为相关分片。请注意，由于大多数搜索将被发送到一个或多个索引，因此它们通常需要从多个分片中读取，每个分片代表数据的不同子集。
* 从分片复制组中选择每个相关分片的活动副本。这可以是主要副本或副本。默认情况下，Elasticsearch将简单地在分片副本之间循环。
* 将分片级读取请求发送到所选副本。
* 结合结果并做出回应。请注意，在通过ID查找的情况下，只有一个分片是相关的，可以跳过此步骤。

#### 故障处理

当分片无法响应读取请求时，协调节点将从同一复制组中选择另一个副本，并将分片级别搜索请求发送到该副本。重复失败可能导致没有可用的分片副本。在某些情况下，例如_search，Elasticsearch更愿意快速响应，尽管有部分结果，而不是等待问题得到解决（部分结果显示在_shards响应的标题中）。

### 一些简单的含义

这些基本流程中的每一个都决定了Elasticsearch如何作为读取和写入系统的行为。此外，由于读取和写入请求可以同时执行，因此这两个基本流程彼此交互。这有一些固有的含义：

高效的阅读
    在正常操作下，对每个相关复制组执行一次读取操作。只有在失败条件下，同一个分片的多个副本才会执行相同的搜索。
阅读未经承认
    由于主要的第一个索引在本地然后复制请求，因此并发读取可能在确认之前已经看到了更改。
默认情况下为两份
    此模型可以容错，同时仅保留两个数据副本。这与基于法定数量的系统形成对比，其中容错的最小副本数为3。

### 失败

在失败的情况下，以下是可能的：

单个分片可以减慢索引速度
    由于主服务器在每个操作期间等待同步副本集中的所有副本，因此单个慢速分片可能会降低整个复制组的速度。这是我们为上述阅读效率支付的价格。当然，单个慢速分片也会减慢已经路由到它的不幸搜索。
脏读
    隔离的主数据库可以暴露无法识别的写入。这是因为隔离的主服务器只有在向其副本发送请求或向主服务器发送请求时才会意识到它是隔离的。此时，操作已经索引到主服务器中，并且可以通过并发读取来读取。Elasticsearch通过每秒ping一次主服务器（默认情况下）并在没有master知道的情况下拒绝索引操作来减轻这种风险。

### The Tip of the Icebergedit

本文档提供了Elasticsearch如何处理数据的高级概述。当然，还有很多事情要发生在幕后。主要术语，集群状态发布和主选举等都可以在保持系统正常运行方面发挥作用。本文档也未涵盖已知和重要的错误（关闭和打开）。我们认识到GitHub很难跟上。为了帮助人们掌控这些，我们 在我们的网站上维护了一个专用的[弹性页面](https://www.elastic.co/guide/en/elasticsearch/resiliency/current/index.html)。我们强烈建议阅读它。

## 索引api

索引API在特定索引中添加或更新JSON文档，使其可搜索。以下示例将JSON文档插入ID为1的“twitter”索引中：

```es
PUT twitter/_doc/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述索引操作的结果是：

```es
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result" : "created"
}
```

`_shards` 报头提供关于索引操作的复制过程的信息：

`total`
    指示应在其上执行索引操作的分片副本（主分片和副本分片）的数量。
`successful`
    指示索引操作成功的分片副本数。
`failed`
    在副本分片上索引操作失败的情况下包含与复制相关的错误的数组。

索引操作成功的情况successful至少为1。

> 索引操作成功返回时，可能无法全部启动副本分片（默认情况下，只有primary需要，但可以更改此行为）。在这种情况下， total将等于基于number_of_replicas设置的总分片successful数，并且将等于已启动的分片数（主要副本和副本）。如果没有失败，则为failed0。

### 自动创建索引

如果索引尚不存在，则索引操作会自动创建索引，并应用已配置的任何索引模板。如果尚不存在，则索引操作还会创建动态映射。默认情况下，如果需要，新字段和对象将自动添加到映射定义中。有关映射定义的更多信息，请查看映射部分; 有关手动更新映射的信息，请查看put映射 API。

自动索引创建由action.auto_create_index 设置控制。此设置默认为true，表示始终自动创建索引。通过将此设置的值更改为这些模式的逗号分隔列表，仅允许对匹配特定模式的索引创建自动索引。也可以通过在列表中使用+or 作为前缀来明确允许和禁止它-。最后，可以通过将此设置更改为完全禁用false。

```es
PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*"  #1. 只允许自动创建索引叫twitter，index10没有其他的折射率匹配index1*，以及任何其他折射率匹配ind*。模式按照给定的顺序进行匹配。
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "false" #1. 完全禁用索引的自动创建。
    }
}

PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true"  #1. 允许使用任何名称自动创建索引。这是默认值。
    }
}
```

### 操作类型

索引操作还接受op_type可用于强制create操作的操作，允许“put-if-absent”行为。当 create使用时，如果该ID在文档中的索引已经存在索引操作将失败。

以下是使用op_type参数的示例：

```es
PUT twitter/_doc/1?op_type=create
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

另一个指定的选项create是使用以下uri：

```es
PUT twitter/_create/1
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

### 自动ID生成

可以在不指定id的情况下执行索引操作。在这种情况下，将自动生成id。此外，op_type 将自动设置为create。这是一个例子（注意 POST使用而不是PUT）：

```es
POST twitter/_doc/
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

上述索引操作的结果是：

```es
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "W0tpsmIBdwcYyG50zbta",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "result": "created"
}
```

### 乐观并发控制

索引操作可以是有条件的，只有在为文档的最后一次修改分配序列号和if_seq_no和if_primary_term参数指定的主要术语时才能执行索引操作。如果检测到不匹配，则操作将导致a VersionConflictException 和状态代码为409.有关详细信息，请参阅[乐观并发控制](https://www.elastic.co/guide/en/elasticsearch/reference/current/optimistic-concurrency-control.html)。

### 路由

默认情况下，分片放置？还是routing？通过使用文档的id值的哈希来控制。为了更明确的控制，可以使用routing参数在每个操作的基础上直接指定输入到路由器使用的散列函数的值。例如：

```es
POST twitter/_doc?routing=kimchy
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

在上面的示例中，“_ doc”文档根据提供的routing参数路由到分片：“kimchy”。

设置显式映射时，_routing可以选择使用该字段指示索引操作从文档本身提取路由值。这确实来自另一个文档解析过程的（非常小的）成本。如果_routing映射已定义并设置为required，则如果未提供或提取路由值，则索引操作将失败。

### 分布式

索引操作根据其路由指向主分片（请参阅上面的“路由”部分），并在包含此分片的实际节点上执行。主分片完成操作后，如果需要，更新将分发到适用的副本。

### 等待活动碎片

为了提高对系统写入的弹性，可以将索引操作配置为在继续操作之前等待一定数量的活动分片副本。如果必需数量的活动分片副本不可用，则写入操作必须等待并重试，直到必需的分片副本已启动或发生超时。默认情况下，写入操作仅等待主分片在继续（即wait_for_active_shards=1）之前处于活动状态。可以通过设置动态地在索引设置中覆盖此默认值index.write.wait_for_active_shards。要更改每个操作的此行为，wait_for_active_shards可以使用request参数。

有效值是all或任何正整数，直到索引中每个分片的已配置副本总数（即number_of_replicas+1）。指定负值或大于分片副本数的数字将引发错误。

例如，假设我们有三个节点的群集，A，B，和C，我们创建索引index设置为3的副本数量（导致4个碎片副本，一个副本多个比存在的节点）。如果我们尝试索引操作，默认情况下，操作只会确保每个分片的主副本在继续之前可用。这意味着即使B并且C关闭并A托管主分片副本，索引操作仍将仅使用一个数据副本。如果wait_for_active_shards在请求上设置3（并且所有3个节点都已启动），然后索引操作将在继续之前需要3个活动分片副本，这是一个应该满足的要求，因为群集中有3个活动节点，每个活动节点都拥有该分片的副本。但是，如果我们设置wait_for_active_shards为all（或者4相同），则索引操作将不会继续，因为我们没有在索引中激活每个分片的所有4个副本。除非在群集中启动新节点以托管分片的第四个副本，否则操作将超时。

重要的是要注意，此设置大大降低了写入操作不写入所需数量的分片副本的可能性，但它并未完全消除这种可能性，因为此检查在写入操作开始之前发生。一旦写入操作正在进行，复制在任何数量的分片副本上仍然可能失败，但仍然可以在主要副本上成功。在_shards写操作的响应部分揭示了其复制成功碎片的份数/失败。

```es
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    }
}
```

### 刷新

控制何时此请求所做的更改对搜索可见。请参阅 刷新。

### Noop更新

使用索引API更新文档时，即使文档未更改，也始终会创建新版本的文档。如果这是不可接受的，请使用设置为true 的_updateAPI detect_noop。此选项在索引API上不可用，因为索引API不会获取旧源，也无法将其与新源进行比较。

关于何时不接受noop更新，没有一条硬性规定。它是许多因素的组合，例如您的数据源发送实际noops更新的频率以及Elasticsearch在接收更新的分片上运行的每秒查询数。

### 超时

执行索引操作时，分配用于执行索引操作的主分片可能不可用。原因可能是主分片当前正从网关恢复或正在进行重定位。默认情况下，索引操作将在主分片上等待最多1分钟，然后失败并响应错误。该timeout参数可用于显式指定等待的时间。以下是将其设置为5分钟的示例：

```es
PUT twitter/_doc/1?timeout=5m
{
    "user" : "kimchy",
    "post_date" : "2009-11-15T14:12:12",
    "message" : "trying out Elasticsearch"
}
```

### 版本控制

每个索引文档都有一个版本号。默认情况下，使用从1开始的内部版本控制，并在每次更新时递增，包括删除。可选地，版本号可以设置为外部值（例如，如果在数据库中维护）。要启用此功能，version_type应将其设置为 external。提供的值必须是大于或等于0且小于大约9.2e + 18的数字长值。

使用外部版本类型时，系统会检查传递给索引请求的版本号是否大于当前存储文档的版本号。如果为true，则将索引文档并使用新版本号。如果提供的值小于或等于存储文档的版本号，则会发生版本冲突，索引操作将失败。例如：

```es
PUT twitter/_doc/1?version=2&version_type=external
{
    "message" : "elasticsearch now has versioning support, double cool!"
}
```

注意：版本控制是完全实时的，不受搜索操作的近实时方面的影响。如果未提供任何版本，则执行该操作而不进行任何版本检查。

由于提供的版本2高于当前文档版本1，因此上述操作将成功。如果文档已更新且其版本设置为2或更高，则索引命令将失败并导致冲突（409 http状态代码）。

一个好的副作用是，只要使用源数据库中的版本号，就不需要维护由于源数据库更改而执行的异步索引操作的严格排序。如果使用外部版本控制，即使使用数据库中的数据更新Elasticsearch索引的简单情况也会简化，因为如果索引操作由于某种原因而无序到达，则仅使用最新版本。

###版本类型

在external上面解释的版本类型旁边，Elasticsearch还支持特定用例的其他类型。以下是不同版本类型及其语义的概述。

`internal`

    仅在给定版本与存储文档的版本相同时才对文档编制索引。

`external or external_gt`

    如果给定版本严格高于存储文档的版本或者没有现有文档，则仅索引文档。给定版本将用作新版本，并将与新文档一起存储。提供的版本必须是非负长号。
`external_gte`

    仅在给定版本等于或高于存储文档的版本时索引文档。如果没有现有文档，操作也将成功。给定版本将用作新版本，并将与新文档一起存储。提供的版本必须是非负长号。

注意：external_gte版本类型适用于特殊用例，应小心使用。如果使用不当，可能会导致数据丢失。还有另一个选项，force它已被弃用，因为它可能导致主分片和副本分片发散。

## Get api

get API允许根据其id从索引中获取JSON文档。以下示例从名为twitter的索引获取一个JSON文档，其id值为0：

`GET twitter/_doc/0`

上述get操作的结果是：

```es
{
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "0",
    "_version" : 1,
    "_seq_no" : 10,
    "_primary_term" : 1,
    "found": true,
    "_source" : {
        "user" : "kimchy",
        "date" : "2009-11-15T14:12:12",
        "likes": 0,
        "message" : "trying out Elasticsearch"
    }
}
```

上述结果包括_index，_id，和_version 我们希望检索，包括实际文档的_source 文档，如果可以发现（如由指示found 字段中响应）。

API还允许使用HEAD例如以下内容检查文档是否存在 ：

`HEAD twitter/_doc/0`

### 实时

默认情况下，get API是实时的，并且不受索引刷新率的影响（当数据对搜索可见时）。如果文档已更新但尚未刷新，则get API将就地发出刷新调用以使文档可见。这也将使上次刷新后其他文档发生变化。为了禁用实时GET，可以将realtime参数设置为false

### 源过滤

默认情况下，_source 除非您使用了stored_fields参数或_source禁用了该字段，否则get操作将返回该字段的内容。您可以_source使用以下_source参数关闭检索：

`GET twitter/_doc/0?_source=false`

如果您只需要完整的一个或两个字段，则_source可以使用_source_includes 和_source_excludes参数来包含或过滤掉您需要的部分。这对于大型文档尤其有用，其中部分检索可以节省网络开销。这两个参数都使用逗号分隔的字段列表或通配符表达式。例：

`GET twitter/_doc/0?_source_includes=*.id&_source_excludes=entities`

如果您只想指定包含，则可以使用较短的表示法：

`GET twitter/_doc/0?_source=*.id,retweeted`

### 存储的字段

get操作允许指定将通过传递stored_fields参数返回的一组存储字段。如果未存储请求的字段，则将忽略它们。例如，考虑以下映射

```es
PUT twitter
{
   "mappings": {
       "properties": {
          "counter": {
             "type": "integer",
             "store": false
          },
          "tags": {
             "type": "keyword",
             "store": true
          }
       }
   }
}
```

现在我们可以添加一个文档：

```es
PUT twitter/_doc/1
{
    "counter" : 1,
    "tags" : ["red"]
}
```

然后尝试检索它：

`GET twitter/_doc/1?stored_fields=tags,counter`

上述get操作的结果是：

```es
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "1",
   "_version": 1,
   "_seq_no" : 22,
   "_primary_term" : 1,
   "found": true,
   "fields": {
      "tags": [
         "red"
      ]
   }
}
```

从文档本身获取的字段值始终作为数组返回。由于该counter字段未存储，因此get请求在尝试获取时只是忽略它stored_fields.

也可以检索字段之类的元数据字段_routing：

```es
PUT twitter/_doc/2?routing=user1
{
    "counter" : 1,
    "tags" : ["white"]
}
```

`GET twitter/_doc/2?routing=user1&stored_fields=tags,counter`

上述get操作的结果是：

```es
{
   "_index": "twitter",
   "_type": "_doc",
   "_id": "2",
   "_version": 1,
   "_seq_no" : 13,
   "_primary_term" : 1,
   "_routing": "user1",
   "found": true,
   "fields": {
      "tags": [
         "white"
      ]
   }
}
```

此外，只有叶子字段可以通过stored_field选项返回。因此无法返回对象字段，此类请求将失败。

### 直接获取_source

使用/{index}/_source/{id} endpoint 只获取_source文档的字段，而不包含任何其他内容。例如：

`GET twitter/_source/1`

您还可以使用相同的源过滤参数来控制_source将返回的部分：

`GET twitter/_source/1/?_source_includes=*.id&_source_excludes=entities`

注意，_source端点还有一个HEAD变体，可以有效地测试document _source的存在。如果在映射中禁用了现有文档，则该文档将没有_source 。

`HEAD twitter/_source/1`

### 路由

使用控制路由的能力进行索引时，为了获取文档，还应提供路由值。例如

`GET twitter/_doc/2?routing=user1`

以上将获得带有id = 2的推文，但将根据用户进行路由。请注意，在没有正确路由的情况下发出get将导致无法获取文档。

### 偏好

控制preference哪个分片副本执行get请求。默认情况下，操作在分片复制副本之间随机化。

该preference可设置为：

_local
    如果可能，操作将优选在本地分配的分片上执行。
自定义（字符串）值
    将使用自定义值来保证相同的分片将用于相同的自定义值。当在不同的刷新状态下击中不同的分片时，这可以帮助“跳跃值”。示例值可以是Web会话ID或用户名。

### 刷新

该refresh参数可以设置为true以刷新有关的碎片get操作之前，并使其可搜索。设置它true应该在经过仔细考虑和验证后，这不会导致系统负载过重（并减慢索引）。

### 分布式

get操作被散列为特定的分片ID。然后它被重定向到该分片ID中的一个副本并返回结果。副本是主分片及其在该分片ID组中的副本。这意味着我们拥有的副本越多，我们将拥有更好的GET缩放。

### 版本控制支持

version仅当文档的当前版本等于指定的版本时，才可以使用该参数检索文档。对于所有版本类型，此行为都是相同的，FORCE但始终检索文档的版本类型除外。请注意，FORCE不推荐使用版本类型。

在内部，Elasticsearch已将旧文档标记为已删除并添加了一个全新的文档。旧版本的文档不会立即消失，但您将无法访问它。当您继续索引更多数据时，Elasticsearch会在后台清除已删除的文档。

## 删除api

delete API允许根据其id从特定索引中删除JSON文档。以下示例从twitter使用ID 调用的索引中删除JSON文档1：

`DELETE /twitter/_doc/1`

上述删除操作的结果是：

```es
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    },
    "_index" : "twitter",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 2,
    "_primary_term": 1,
    "_seq_no": 5,
    "result": "deleted"
}
```

### 乐观并发控制

删除操作可以是有条件的，只有在为文档的最后一次修改分配序列号和if_seq_no和if_primary_term参数指定的主要术语时才能执行。如果检测到不匹配，则操作将导致a VersionConflictException 和状态代码为409.有关详细信息，请参阅乐观并发控制。

### 版本控制

索引的每个文档都是版本化的。删除文档时，version可以指定以确保我们尝试删除的相关文档实际上已被删除，并且在此期间它没有更改。对文档执行的每个写入操作（包括删除）都会导致其版本递增。删除文档的版本号在删除后仍可短时间使用，以便控制并发操作。已删除文档的版本保持可用的时间长度由index.gc_deletes索引设置决定，默认为60秒。

### 路由

使用控制路由的能力进行索引时，为了删除文档，还应提供路由值。例如：

`DELETE /twitter/_doc/1?routing=kimchy`

以上将删除带有id的推文1，但将根据用户进行路由。请注意，在没有正确路由的情况下发出删除将导致文档不被删除。

当_routing映射设置为required并且未指定路由值时，删除API将抛出RoutingMissingException并拒绝该请求。

### 自动索引创建

如果使用外部版本控制变体，则删除操作会自动创建索引（如果之前尚未创建）（请查看create index API 以手动创建索引）。

### 分布式

删除操作将散列为特定的分片ID。然后它被重定向到该id组中的主分片，并复制（如果需要）到该id组内的分片副本。

### 等待活动碎片

在发出删除请求时，您可以将wait_for_active_shards 参数设置为在开始处理删除请求之前要求最少数量的分片副本处于活动状态。有关更多详细信息和用法示例，请参见 此处。

### 刷新

控制何时此请求所做的更改对搜索可见。见 ?refresh。

### 超时

执行删除操作时，分配用于执行删除操作的主分片可能不可用。造成这种情况的一些原因可能是主分片当前正在从商店恢复或正在进行重定位。默认情况下，删除操作将在主分片上等待最多1分钟，然后失败并响应错误。该timeout参数可用于显式指定等待的时间。以下是将其设置为5分钟的示例：

`DELETE /twitter/_doc/1?timeout=5m`


## 按查询API 编辑

最简单的用法_delete_by_query就是对匹配查询的每个文档执行删除操作。这是API：

```es
POST twitter/_delete_by_query
{
  "query": { 
    "match": {
      "message": "some message"
    }
  }
}
```

必须query以与Search API相同的方式将查询作为值传递给键。您也可以使用q 与搜索API相同的方式使用该参数。

这将返回如下内容：

```es
{
  "took" : 147,
  "timed_out": false,
  "deleted": 119,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "total": 119,
  "failures" : [ ]
}
```

_delete_by_query在索引启动时获取索引的快照，并使用internal版本控制删除它找到的内容。这意味着如果文档在拍摄快照的时间和处理删除请求之间发生更改，则会出现版本冲突。当版本匹配时，文档将被删除。

> 由于internal版本控制不支持将值0作为有效版本号，因此无法使用版本等于零的文档删除， _delete_by_query并且将使请求失败。

在_delete_by_query执行期间，顺序执行多个搜索请求以便找到要删除的所有匹配文档。每次找到一批文档时，都会执行相应的批量请求以删除所有这些文档。如果搜索或批量请求被拒绝，则_delete_by_query 依赖于默认策略来重试被拒绝的请求（最多10次，指数后退）。达到最大重试次数限制会导致_delete_by_query 中止，并failures在响应中返回所有失败。已执行的删除仍然有效。换句话说，该过程不会回滚，只会中止。当第一个失败导致中止时，失败的批量请求返回的所有失败都将在failures 元件; 因此，可能存在相当多的失败实体。

如果您想计算版本冲突而不是导致它们中止，那么请conflicts=proceed在URL或"conflicts": "proceed"请求正文中设置。

回到API格式，这将删除twitter索引中的推文：

```es
POST twitter/_delete_by_query?conflicts=proceed
{
  "query": {
    "match_all": {}
  }
}
```

也可以一次删除多个索引的文档，就像搜索API一样：

```es
POST twitter,blog/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

如果您提供，routing则路由将复制到滚动查询，将进程限制为与该路由值匹配的分片：

```es
POST twitter/_delete_by_query?routing=1
{
  "query": {
    "range" : {
        "age" : {
           "gte" : 10
        }
    }
  }
}
```

默认情况下，_delete_by_query使用1000的滚动批次。您可以使用scroll_sizeURL参数更改批量大小：

```es
POST twitter/_delete_by_query?scroll_size=5000
{
  "query": {
    "term": {
      "user": "kimchy"
    }
  }
}
```

### URL参数

除了标准的参数，如pretty，删除通过查询API也支持refresh，wait_for_completion，wait_for_active_shards，timeout，和scroll。

发送refresh请求将在请求完成后刷新查询中的所有分片。这与delete API的refresh 参数不同，后者只会导致刷新接收删除请求的分片。与删除API不同，它不支持wait_for。

如果请求包含，wait_for_completion=false则Elasticsearch将执行一些预检检查，启动请求，然后返回task 可与Tasks API 一起使用以取消或获取任务状态的请求。Elasticsearch还将创建此任务的记录作为文档.tasks/task/${taskId}。这是你的保留或删除你认为合适。完成后，删除它，以便Elasticsearch可以回收它使用的空间。

wait_for_active_shards控制在继续请求之前必须激活碎片的副本数量。详情请见此处 。timeout控制每个写入请求等待不可用分片变为可用的时间。两者都完全适用于 Bulk API中的工作方式。由于_delete_by_query采用滚动搜索，你还可以指定scroll参数来控制多长时间保持“搜索上下文”活着，例如?scroll=10m。默认情况下，它是5分钟。

requests_per_second可以被设置为任何正十进制数（1.4，6， 1000等）和节流在该删除由查询通过填充每批与等待时间发出的删除操作的批次的速率。可以通过设置requests_per_second为禁用限制-1。

限制是通过在批处理之间等待来完成的，这样在_delete_by_query内部使用的滚动 可以被赋予考虑填充的超时。填充时间是批量大小除以requests_per_second写入所花费的时间之间的差异。默认情况下，批量大小为1000，因此如果requests_per_second设置为500：

```shell
target_time = 1000 / 500 per second = 2 seconds
wait_time = target_time - write_time = 2 seconds - .5 seconds = 1.5 seconds
```

由于批处理是作为单个_bulk请求发出的，因此大批量大小将导致Elasticsearch创建许多请求，然后等待一段时间再开始下一个集合。这是“突发”而不是“平滑”。默认是-1。

### 响应正文

JSON响应如下所示

```es
{
  "took" : 147,
  "timed_out": false,
  "total": 119,
  "deleted": 119,
  "batches": 1,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1.0,
  "throttled_until_millis": 0,
  "failures" : [ ]
}
```

took
    从整个操作的开始到结束的毫秒数。
timed_out
    true如果在通过查询执行删除期间执行的任何请求超时 ，则将此标志设置为。
total
    已成功处理的文档数。
deleted
    已成功删除的文档数。
batches
    通过查询删除回滚的滚动响应数。
version_conflicts
    按查询删除的版本冲突数。
noops
    对于按查询删除，此字段始终等于零。它只存在，以便通过查询删除，按查询更新和重新索引API返回具有相同结构的响应。
retries
    通过查询删除尝试的重试次数。bulk是重试的批量操作search的数量，是重试的搜索操作的数量。
throttled_millis
    请求睡眠符合的毫秒数requests_per_second。
requests_per_second
    在通过查询删除期间有效执行的每秒请求数。
throttled_until_millis
    该字段在_delete_by_query响应中应始终等于零。它只在使用Task API时有意义，它指示下一次（自纪元以来的毫秒），为了符合，将再次执行受限制的请求requests_per_second。
failures
    如果在此过程中存在任何不可恢复的错误，则会出现故障数组。如果这不是空的，那么请求因为那些失败而中止。使用批处理实现查询删除，任何故障都会导致整个进程中止，但当前批处理中的所有故障都将收集到阵列中。您可以使用该conflicts选项来防止reindex在版本冲突中中止。

### 使用Task API 

您可以使用Task API通过查询请求获取任何正在运行的删除的状态 ：

GET _tasks?detailed=true&actions=*/delete/byquery

响应如下：

```es
{
  "nodes" : {
    "r1A2WoRbTwKZ516z6NEs5A" : {
      "name" : "r1A2WoR",
      "transport_address" : "127.0.0.1:9300",
      "host" : "127.0.0.1",
      "ip" : "127.0.0.1:9300",
      "attributes" : {
        "testattr" : "test",
        "portsfile" : "true"
      },
      "tasks" : {
        "r1A2WoRbTwKZ516z6NEs5A:36619" : {
          "node" : "r1A2WoRbTwKZ516z6NEs5A",
          "id" : 36619,
          "type" : "transport",
          "action" : "indices:data/write/delete/byquery",
          "status" : {    
            "total" : 6154,
            "updated" : 0,
            "created" : 0,
            "deleted" : 3500,
            "batches" : 36,
            "version_conflicts" : 0,
            "noops" : 0,
            "retries": 0,
            "throttled_millis": 0
          },
          "description" : ""
        }
      }
    }
  }
}
```

该对象包含实际状态。它就像响应JSON一样，具有重要的total字段添加功能。total是reindex期望执行的操作总数。您可以通过添加估计的进展updated，created以及deleted多个领域。请求将在其总和等于total字段时结束。

使用任务ID，您可以直接查找任务：

`GET /_tasks/r1A2WoRbTwKZ516z6NEs5A:36619`

此API的优势在于它可以集成wait_for_completion=false 以透明地返回已完成任务的状态。如果任务已完成并wait_for_completion=false已设置在其上，则它将返回 results或带有error字段。此功能的成本是wait_for_completion=false创建的文档 .tasks/task/${taskId}。您可以删除该文档。

### 使用Cancel Task API 

可以使用任务取消API取消任何查询删除：

`POST _tasks/r1A2WoRbTwKZ516z6NEs5A:36619/_cancel`

可以使用任务API找到任务ID 。

取消应该很快发生，但可能需要几秒钟。上面的任务状态API将继续列出按查询删除任务的列表，直到此任务检查它已被取消并终止自身。

### Rethrottling 

requests_per_second可以使用_rethrottleAPI 通过查询在运行删除时更改值：

`POST _delete_by_query/r1A2WoRbTwKZ516z6NEs5A:36619/_rethrottle?requests_per_second=-1`

可以使用任务API找到任务ID 。

就像在查询API的删除上设置它一样，requests_per_second 可以是-1禁用限制或任何十进制数，如1.7或12限制到该级别。加速查询的Rethrottling会立即生效，但重新启动会减慢查询速度，这将在完成当前批处理后生效。这可以防止滚动超时。

### 切片

按查询删除支持切片滚动以并行化删除过程。这种并行化可以提高效率，并提供一种方便的方法将请求分解为更小的部分。

#### 手动切片

通过为每个请求提供切片ID和切片总数，手动切片查询删除：

```es
POST twitter/_delete_by_query
{
  "slice": {
    "id": 0,
    "max": 2
  },
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
POST twitter/_delete_by_query
{
  "slice": {
    "id": 1,
    "max": 2
  },
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

您可以验证哪些适用于：

```es
GET _refresh
POST twitter/_search?size=0&filter_path=hits.total
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

这样的结果是明智的total：

```es
{
  "hits": {
    "total" : {
        "value": 0,
        "relation": "eq"
    }
  }
}
```

### 自动切片

您还可以使用切片滚动切片打开，让逐个查询自动并行化 _id。使用slices指定片使用的数字：

```es
POST twitter/_delete_by_query?refresh&slices=5
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

您还可以验证以下内容：

```
POST twitter/_search?size=0&filter_path=hits.total
{
  "query": {
    "range": {
      "likes": {
        "lt": 10
      }
    }
  }
}
```

这样的结果是合理的total：

```es
{
  "hits": {
    "total" : {
        "value": 0,
        "relation": "eq"
    }
  }
}
```

设置slices为auto将让Elasticsearch选择要使用的切片数。此设置将使用每个分片一个切片，达到一定限制。如果有多个源索引，它将根据具有最小分片数的索引选择切片数。

添加slices到_delete_by_query刚刚自动化在上面的部分中使用的手工工艺，创建子请求，这意味着它有一些怪癖：

* 您可以在Tasks API中查看这些请求 。这些子请求是请求任务的“子”任务slices。
* 获取请求的任务状态slices仅包含已完成切片的状态。
* 这些子请求可单独寻址，例如取消和重新限制。
* 对请求进行slices重新规范将按比例重新调整未完成的子请求。
* 取消请求slices将取消每个子请求。
* 由于slices每个子请求的性质将无法获得完全均匀的文档部分。将解决所有文档，但某些切片可能比其他文件更大。期望更大的切片具有更均匀的分布。
* 像请求requests_per_second和size请求的参数slices 按比例分配给每个子请求。再加上上述有关分配不均正在点，你应该得出结论，使用 size与slices可能无法产生完全size的文件被删除。
* 每个子请求获得源索引的略有不同的快照，尽管这些快照几乎同时进行。

### 挑选切片数量

如果自动切片，设置slices为auto将为大多数索引选择合理的数字。如果您手动切片或以其他方式调整自动切片，请使用这些指南。

当数量slices等于索引中的分片数时，查询性能最有效。如果该数字很大（例如，500），请选择较小的数字，因为太多slices会损害性能。设置 slices高于分片数通常不会提高效率并增加开销。

删除性能在可用资源上以切片数量线性扩展。

查询或删除性能是否主导运行时取决于重新编制索引的文档和群集资源。



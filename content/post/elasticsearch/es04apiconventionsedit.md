---
title: "Es04apiconventionsedit"
date: 2021-04-21T18:08:46+08:00
lastmod: 2021-04-21T18:08:46+08:00
draft: false
keywords: []
description: "manjoc'blog"
tags: []
categories: []
author: "王清"
---

# api 约定

该Elasticsearch REST API的使用暴露JSON通过HTTP。

除非另有说明，否则本章中列出的约定可以在整个REST API中应用。

* 多个指数
* 索引名称中的日期数学支持
* 常见选项
* 基于URL的访问控制

## 多个索引 multi indices

大多数引用index参数的API都支持使用简单test1,test2,test3表示法（或_all所有索引）跨多个索引执行。它还支持通配符，例如：test*或*test或te*t或*test*，以及“排除”（-）的能力，例如：test*,-test3。

所有多索引API都支持以下url查询字符串参数：

ignore_unavailable: 控制是否忽略任何指定的索引是否不可用，包括不存在的索引或闭合索引。任一true或false 可以被指定。

allow_no_indices: 如果通配符索引表达式导致没有具体索引，则控制是否失败。任一true或false可以被指定。例如，如果foo*指定了通配符表达式并且没有可用的索引foo，那么根据此设置，请求将失败。当指定或不指定索引时_all，此设置也适用*。如果别名指向封闭索引，则此设置也适用于别名。

expand_wildcards: 控制通配符索引表达式可以扩展到的具体索引类型。如果open指定了，则通配符表达式将扩展为仅打开索引。如果closed指定了，则通配符表达式仅扩展为已关闭的索引。同时，两个values（open,closed）都可以指定为扩展到所有索引。 如果none指定，则将禁用通配符扩展。如果all 指定，则通配符表达式将扩展为所有索引（这相当于指定open,closed）。

上述参数的默认设置取决于所使用的API。

> 单个索引API（如[Document API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html)和 [单索引aliasAPI](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html)）不支持多个索引。

## 索引名称中的日期数学表达式支持

日期数学索引名称解析使您可以搜索一系列时间序列索引，而不是搜索所有时间序列索引并过滤结果或维护别名。限制搜索的索引数可减少群集上的负载并提高执行性能。例如，如果您在日常日志中搜索错误，则可以使用日期数学名称模板将搜索限制为过去两天。

几乎所有具有index参数的API都支持index参数值中的日期数学。

日期数学索引名称采用以下形式：

`<static_name{date_math_expr{date_format|time_zone}}>`

| static_name | 是名称的静态文本部分 | 
| date_math_expr | 是动态计算日期的动态日期数学表达式 | 
| date_format | 是应该呈现计算日期的可选格式。默认为yyyy.MM.dd。格式应与java-time兼容https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html | 
| time_zone | 是可选的时区。默认为utc。 | 

日期数学表达式与区域设置无关。因此，除了公历之外，不可能使用任何其他日历。

您必须将日期数学索引名称表达式括在尖括号内，并且所有特殊字符都应进行URI编码。例如：

```es
# GET /<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```

日期数学字符的百分比编码
用于日期舍入的特殊字符必须按如下URI编码

| < | %3C |
| > | %3E |
| / | %2F |
| { | %7B |
| } | %7D |
| | | %7C |
| + | %2B |
| : | %3A |
| , | %2C |

以下示例显示了日期数学索引名称的不同形式以及它们在给定当前时间时解析的最终索引名称是2024年3月22日中午utc。

| 表达 | 解决了 |
| ------ | ------ |
| <logstash-{now/d}> | logstash-2024.03.22 |
| <logstash-{now/M}> | logstash-2024.03.01 |
| <logstash-{now/M{yyyy.MM}}> | logstash-2024.03 |
| <logstash-{now/M-1M{yyyy.MM}}> | logstash-2024.02 |
| <logstash-{now/d{yyyy.MM.dd|+12:00}}> | logstash-2024.03.23 |


要在索引名称的静态模板中使用 `{` 和 `}` 字符，请使用反斜杠转义它们\，例如：

`<elastic\\{ON\\}-{now/M}> resolves to elastic{ON}-2024.03.01`

以下示例显示搜索请求，该搜索请求搜索过去三天的Logstash索引，假设索引使用默认的Logstash索引名称格式， logstash-yyyy.MM.dd。

```es
# GET /<logstash-{now/d-2d}>,<logstash-{now/d-1d}>,<logstash-{now/d}>/_search
GET /%3Clogstash-%7Bnow%2Fd-2d%7D%3E%2C%3Clogstash-%7Bnow%2Fd-1d%7D%3E%2C%3Clogstash-%7Bnow%2Fd%7D%3E/_search
{
  "query" : {
    "match": {
      "test": "data"
    }
  }
}
```

## 常见选项

以下选项可应用于所有REST API。

### pretty 输出

当附加?pretty=true到任何请求时，返回的JSON将被格式化（仅用于调试！）。另一种选择是设置?format=yaml哪个将导致以（有时）更可读的yaml格式返回结果。

### 易读的输出

统计数据以适合人类（例如"exists_time": "1h"或"size": "1kb"）和计算机（例如"exists_time_in_millis": 3600000或"size_in_bytes": 1024）的格式返回。可以通过添加?human=false 查询字符串来关闭人类可读取的值。当统计结果被监控工具消耗时，这是有意义的，而不是用于人类消费。human标志的默认值是 false。

### 日期数学表达式

它接受一个格式化的日期值大多数参数-比如gt和lt 中range查询，或from与to 在daterange聚集  -了解最新的数学。

表达式以锚定日期开始，可以是，也可以是以。now结尾的日期字符串||。此锚定日期可以选择后跟一个或多个数学表达式：

* +1h：加一个小时
* -1d：减去一天
* /d：回合到最近的一天

支持的时间单位与持续时间的时间单位支持的时间单位不同。支持的单位是：

| y | 年份 |
| M | 月 |
| w | 周 |
| d | 天 |
| h | 小时 |
| H | 小时 |
| m | 分钟 |
| s | 秒 |

假设now是2001-01-01 12:00:00，一些例子是：

| now+1h | now以毫秒加一小时。解决：2001-01-01 13:00:00 | 
| now-1h | now以毫秒减去一小时。解决：2001-01-01 11:00:00 | 
| now-1h/d | now以毫秒减去一小时，向下舍入到UTC 00:00。解决：2001-01-01 00:00:00 | 
| 2001.02.01\|\|+1M/d | 2001-02-01以毫秒加一个月。解决：2001-03-01 00:00:00 | 

#### 响应中的过滤

所有REST API都接受一个filter_path参数，该参数可用于减少Elasticsearch返回的响应。此参数采用逗号分隔的过滤器列表，用点表示法表示：

`GET /_search?q=elasticsearch&filter_path=took,hits.hits._id,hits.hits._score`

i响应:

```json
{
  "took" : 3,
  "hits" : {
    "hits" : [
      {
        "_id" : "0",
        "_score" : 1.6375021
      }
    ]
  }
}
```

它还支持*通配符以匹配字段名称的任何字段或部分：

`GET /_cluster/state?filter_path=metadata.indices.*.stat*`

响应:

```json
{
  "metadata" : {
    "indices" : {
      "twitter": {"state": "open"}
    }
  }
}
```

并且**通配符可用于包括字段而不知道字段的确切路径。例如，我们可以使用此请求返回每个段的Lucene版本：

`GET /_cluster/state?filter_path=routing_table.indices.**.state`

响应:

```json
{
  "routing_table": {
    "indices": {
      "twitter": {
        "shards": {
          "0": [{"state": "STARTED"}, {"state": "UNASSIGNED"}]
        }
      }
    }
  }
}
```

也可以通过在过滤器前面加上 `-` 来排除一个或多个字段：

`GET /_count?filter_path=-_shards`

```json

{
  "count" : 5
}
```

并且为了进行更多控制，可以将包含和排他过滤器组合在同一表达式中。在这种情况下，将首先应用独占过滤器，并使用包含过滤器再次过滤结果：

`GET /_cluster/state?filter_path=metadata.indices.*.state,-metadata.indices.logstash-*`

```json
{
  "metadata" : {
    "indices" : {
      "index-1" : {"state" : "open"},
      "index-2" : {"state" : "open"},
      "index-3" : {"state" : "open"}
    }
  }
}
```

请注意，Elasticsearch有时会直接返回字段的原始值，如_source字段。如果要筛选_source字段，则应考虑将已存在的_source参数（请参阅 获取API以获取更多详细信息）与以下filter_path 参数组合：

```shell
POST /library/book?refresh
{"title": "Book #1", "rating": 200.1}
POST /library/book?refresh
{"title": "Book #2", "rating": 1.7}
POST /library/book?refresh
{"title": "Book #3", "rating": 0.1}
GET /_search?filter_path=hits.hits._source&_source=title&sort=rating:desc
```

```json
{
  "hits" : {
    "hits" : [ {
      "_source":{"title":"Book #1"}
    }, {
      "_source":{"title":"Book #2"}
    }, {
      "_source":{"title":"Book #3"}
    } ]
  }
}
```

### flat 设置

该flat_settings标志会影响设置列表的呈现。当 flat_settings标志true，设置返回在一个平面格式：

`GET twitter/_settings?flat_settings=true`

返回:

```json
{
  "twitter" : {
    "settings": {
      "index.number_of_replicas": "1",
      "index.number_of_shards": "1",
      "index.creation_date": "1474389951325",
      "index.uuid": "n6gzFZTgS664GUfx0Xrpjw",
      "index.version.created": ...,
      "index.provided_name" : "twitter"
    }
  }
}
```

当flat_settings标志为时false，设置以更易读的结构化格式返回：

`GET twitter/_settings?flat_settings=false`

返回:

```json
{
  "twitter" : {
    "settings" : {
      "index" : {
        "number_of_replicas": "1",
        "number_of_shards": "1",
        "creation_date": "1474389951325",
        "uuid": "n6gzFZTgS664GUfx0Xrpjw",
        "version": {
          "created": ...
        },
        "provided_name" : "twitter"
      }
    }
  }
}
```

默认flat_settings设置为false。

#### 参数

rest参数（使用HTTP时，映射到HTTP URL参数）遵循使用下划线框的约定。

#### bool值

所有REST API参数（请求参数和JSON主体）都支持提供布尔值“false”作为值，false并将布尔值“true”作为值true。所有其他值都会引发错误。

#### number 值

除了string支持本机JSON数字类型之外，所有REST API都支持提供编号参数。

#### time units

每当需要指定持续时间时，例如对于timeout参数，持续时间必须指定单位，例如2d2天。支持的单位是：

| d | 天 |
| h | 小时 |
| m | 分钟 |
| s | 秒 |
| ms | 毫秒 |
| micros | 微秒 |
| nanos | 纳秒 |

#### byte site units

每当需要指定数据的字节大小时，例如在设置缓冲区大小参数时，该值必须指定单位，例如10kb10千字节。请注意，这些单位使用1024的幂，因此1kb意味着1024字节。支持的单位是：

| b | 字节 |
| kb | 千字节 |
| mb | 兆字节 |
| gb | 千兆字节 |
| tb | 兆兆字节 |
| pb | 拍字节 |

#### Unit-less quantitiesedit

无单位数量意味着它们没有“单位”，如“字节”或“赫兹”或“米”或“长吨”。

如果这些数量中的一个很大，我们会将其打印出来，例如 10m (10,000,000) 或 7k (7,000)。当我们的意思是87时，我们仍会打印87。这些是受支持的乘数：

| k | 公斤 |
| m | 兆 |
| g | 千兆 |
| t | 万亿 |
| p | 地图 |

#### 距离单位

无论何处需要指定距离，例如地理距离查询中的distance参数，如果未指定任何距离，则默认单位为米。距离可以用其他单位指定，例如"1km"或 "2mi"（2英里）。

完整的单位清单如下：

| 英里 | mi 要么 miles | 
| 码 | yd 要么 yards |
| 脚 | ft 要么 feet |
| 英寸 | in 要么 inch |
| 公里 | km 要么 kilometers |
| 仪表 | m 要么 meters |
| 厘米 | cm 要么 centimeters |
| 毫米 | mm 要么 millimeters |
| 海里 | NM，nmi或nauticalmiles |

#### fuzziness 模糊

一些查询和API支持参数，以允许使用参数进行不精确的模糊匹配fuzziness。

当查询text或keyword字段时，fuzziness被解释为 Levenshtein编辑距离  - 需要对一个字符串进行一个字符的更改，以使其与另一个字符串相同。

该fuzziness参数可以指定为：

0，1，2 :  允许的最大Levenshtein编辑距离（或编辑数）

 AUTO : 根据术语的长度生成编辑距离。可以可选地提供低距离和高距离参数AUTO:[low],[high]。如果未指定，则默认值为3和6，相当于AUTO:3,6长度的make： 0..2
必须完全匹配
3..5
允许一次编辑
`>5`
允许两次编辑
AUTO通常应该是首选值fuzziness。 

#### 启用堆栈跟踪编辑

默认情况下，当请求返回错误时，Elasticsearch不包含错误的堆栈跟踪。您可以通过将error_traceurl参数设置为来启用该行为 true。例如，默认情况下，当您向API 发送无效size参数时_search：

`POST /twitter/_search?size=surprise_me`

The response looks like:

```json
{
  "error" : {
    "root_cause" : [
      {
        "type" : "illegal_argument_exception",
        "reason" : "Failed to parse int parameter [size] with value [surprise_me]"
      }
    ],
    "type" : "illegal_argument_exception",
    "reason" : "Failed to parse int parameter [size] with value [surprise_me]",
    "caused_by" : {
      "type" : "number_format_exception",
      "reason" : "For input string: \"surprise_me\""
    }
  },
  "status" : 400
}
```

但如果你设置error_trace=true：

POST /twitter/_search?size=surprise_me&error_trace=true

response:

```json
{
  "error": {
    "root_cause": [
      {
        "type": "illegal_argument_exception",
        "reason": "Failed to parse int parameter [size] with value [surprise_me]",
        "stack_trace": "Failed to parse int parameter [size] with value [surprise_me]]; nested: IllegalArgumentException..."
      }
    ],
    "type": "illegal_argument_exception",
    "reason": "Failed to parse int parameter [size] with value [surprise_me]",
    "stack_trace": "java.lang.IllegalArgumentException: Failed to parse int parameter [size] with value [surprise_me]\n    at org.elasticsearch.rest.RestRequest.paramAsInt(RestRequest.java:175)...",
    "caused_by": {
      "type": "number_format_exception",
      "reason": "For input string: \"surprise_me\"",
      "stack_trace": "java.lang.NumberFormatException: For input string: \"surprise_me\"\n    at java.lang.NumberFormatException.forInputString(NumberFormatException.java:65)..."
    }
  },
  "status": 400
}
```

#### 在查询字符串编辑中请求正文

对于不接受非POST请求的请求主体的库，您可以将请求主体作为source查询字符串参数传递。使用此方法时，source_content_type还应使用指示源格式的媒体类型值传递参数，例如application/json。

#### 内容类型要求

必须使用Content-Type标头指定请求正文中发送的内容类型。此标头的值必须映射到API支持的其中一种受支持的格式。大多数API支持JSON，YAML，CBOR和SMILE。批量和多搜索API支持NDJSON，JSON和SMILE; 其他类型将导致错误响应。

此外，使用source查询字符串参数时，必须使用source_content_type查询字符串参数指定内容类型。

## 基于URL的访问控制

许多用户使用具有基于URL的访问控制的代理来保护对Elasticsearch索引的访问。对于多搜索， 多重获取和批量请求，用户可以选择在URL和请求正文中的每个单独请求中指定索引。这可以使基于URL的访问控制具有挑战性。

要防止用户覆盖URL中指定的索引，请将此设置添加到elasticsearch.yml文件中：

`rest.action.multi.allow_explicit_index：false`

默认值为true，但设置false为时，Elasticsearch将拒绝在请求正文中指定了显式索引的请求。

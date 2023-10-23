# ElasticSearch

## 基本概念：

### 集群（Cluster）：

集群是一个或多个节点（服务器）的集合，它们一起保存您的全部数据并提供跨所有节点的联合索引和搜索功能。集群由唯一名称标识，默认情况下为“elasticsearch”。此名称很重要，因为如果节点设置为通过其名称加入集群，则该节点只能成为集群的一部分。

确保您不会在不同的环境中重复使用相同的集群名称，否则您可能会以节点加入错误的集群而告终。例如，您可以将`logging-dev`、`logging-stage`和`logging-prod` 用于开发、暂存和生产集群。

请注意，集群中只有一个节点是有效且完全没问题的。此外，您还可以拥有多个独立的集群，每个集群都有自己唯一的集群名称。

### 节点（Node）

节点是集群中的单个服务器，用于存储数据并参与集群的索引和搜索功能。就像集群一样，节点由名称标识，默认情况下该名称是在启动时分配给节点的随机通用唯一标识符 (UUID)。如果您不需要默认值，您可以定义任何您想要的节点名称。此名称对于管理目的很重要，您想要识别网络中的哪些服务器对应于 Elasticsearch 集群中的哪些节点。

可以将节点配置为通过集群名称加入特定集群。默认情况下，每个节点都设置为加入名为的集群`elasticsearch`，这意味着如果您在网络上启动多个节点，并且假设它们可以相互发现，它们将自动形成并加入一个名为的集群`elasticsearch`。

在单个集群中，您可以拥有任意数量的节点。此外，如果您的网络上当前没有其他 Elasticsearch 节点在运行，则启动单个节点将默认形成一个名为 的新单节点集群`elasticsearch`。

### 索引（Index）

索引是具有某些相似特征的文档的集合。例如，您可以为客户数据创建一个索引，为产品目录创建另一个索引，为订单数据创建另一个索引。索引由名称（必须全部小写）标识，并且在对其中的文档执行索引、搜索、更新和删除操作时，此名称用于引用索引。

在单个集群中，您可以定义任意数量的索引。

### 类型（Type 6.0.0弃用）

类型曾经是索引的逻辑类别/分区，允许您在同一索引中存储不同类型的文档，例如，一种类型用于用户，另一种类型用于博客文章。不再可能在索引中创建多个类型，并且类型的整个概念将在以后的版本中删除。有关更多信息，请参阅[*删除映射类型*](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/removal-of-types.html)。

### 文档（Document）

文档是可以被索引的基本信息单元。例如，您可以有一个客户的文档，一个产品的另一个文档，以及一个订单的另一个文档。本文档以[JSON](http://json.org/)（JavaScript 对象表示法）表示，这是一种普遍存在的互联网数据交换格式。

在索引/类型中，您可以存储任意数量的文档。请注意，尽管文档物理上位于索引中，但实际上文档必须被索引/分配给索引内的类型。

### 分片和副本（Shards&Replicas）

索引可能会存储大量数据，这些数据可能会超过单个节点的硬件限制。例如，占用 1TB 磁盘空间的 10 亿文档的单个索引可能无法容纳在单个节点的磁盘上，或者速度太慢而无法单独满足来自单个节点的搜索请求。

为了解决这个问题，Elasticsearch 提供了将索引细分为多个部分的功能，这些部分称为分片。创建索引时，您可以简单地定义所需的分片数量。每个分片本身就是一个功能齐全且独立的“索引”，可以托管在集群中的任何节点上。

分片之所以重要，主要有两个原因：

- 它允许您水平拆分/缩放内容量
- 它允许您跨分片（可能在多个节点上）分布和并行化操作，从而提高性能/吞吐量

分片如何分布以及其文档如何聚合回搜索请求的机制完全由 Elasticsearch 管理，并且对您作为用户是透明的。

在随时可能发生故障的网络/云环境中，如果分片/节点因某种原因离线或消失，则非常有用并强烈建议使用故障转移机制。为此，Elasticsearch 允许您将索引分片的一个或多个副本制作成所谓的副本分片，或简称副本。

复制之所以重要，主要有两个原因：

- 它在分片/节点出现故障时提供高可用性。出于这个原因，重要的是要注意副本分片永远不会与复制它的原始/主分片分配在同一节点上。
- 它允许您扩展搜索量/吞吐量，因为可以在所有副本上并行执行搜索。

总结一下，每个索引都可以拆分成多个分片。索引也可以被复制零次（意味着没有副本）或更多次。复制后，每个索引将具有主分片（从中复制的原始分片）和副本分片（主分片的副本）。可以在创建索引时为每个索引定义分片和副本的数量。创建索引后，您可以随时动态更改副本数，但无法事后更改分片数。

默认情况下，Elasticsearch 中的每个索引都分配有 5 个主分片和 1 个副本，这意味着如果您的集群中至少有两个节点，您的索引将有 5 个主分片和另外 5 个副本分片（1 个完整副本），总共每个索引 10 个分片。

### 注意：

每个 Elasticsearch 分片都是一个 Lucene 索引。单个 Lucene 索引中可以包含的文档数量有上限。从 开始[`LUCENE-5843`](https://issues.apache.org/jira/browse/LUCENE-5843)，限制为`2,147,483,519`(= Integer.MAX_VALUE - 128) 个文件。[`_cat/shards`](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/cat-shards.html)您可以使用API监控分片大小。

## 探索集群

可以使用 API 完成的几件事如下：

- 检查您的集群、节点和索引健康状况、状态和统计信息
- 管理您的集群、节点和索引数据和元数据
- 对索引执行 CRUD（创建、读取、更新和删除）和搜索操作
- 执行高级搜索操作，例如分页、排序、过滤、脚本、聚合等



### 集群健康（Cluster Health）

让我们从基本的健康检查开始，我们可以用它来查看集群的运行情况。要检查集群健康状况，我们将使用[`_cat`API](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/cat.html)。

```
GET /_cat/health?v
```

- 绿色 - 一切都很好（集群功能齐全）
- 黄色 - 所有数据都可用，但一些副本尚未分配（集群功能齐全）
- 红色 - 某些数据出于某种原因不可用（集群部分功能正常）

**注意：**

当集群为红色时，它将继续为来自可用分片的搜索请求提供服务，但您可能需要尽快修复它，因为存在未分配的分片。

**获取节点列表**

```
GET /_cat/nodes?v
```

### 列出所有索引（list all indices）

```
GET /_cat/indices?v
```

### 创建索引（create index）

```
PUT /customer
```

第一个命令使用 PUT 动词创建名为“customer”的索引。

```
GET /_cat/indices?v
```

黄色表示某些副本尚未（尚未）分配。这个索引发生这种情况的原因是 Elasticsearch 默认为这个索引创建了一个副本。由于我们目前只有一个节点在运行，因此直到另一个节点加入集群后的某个时间点，才能分配一个副本（以实现高可用性）。一旦该副本被分配到第二个节点上，该索引的健康状态将变为绿色。

### 索引和查询文档

### 插入文档

```
PUT /customer/doc/1
{
  "name":"John Doe"
}
```

**响应**：

```json
{
  "_index" : "customer",
  "_type" : "doc",
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

从上面我们可以看出，在客户索引里面成功的新建了一个客户文档。该文档还有一个我们在索引时指定的内部 ID 1。

重要的是要注意，Elasticsearch 不要求您先显式创建索引，然后才能将文档索引到其中。在前面的示例中，如果客户索引事先不存在，Elasticsearch 将自动创建它。

**查看文档**

```
GET /customer/doc/1
```

**响应**：

```json
{
  "_index" : "customer",
  "_type" : "doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```

除了一个字段 ，这里没有什么不寻常的`found`，它表明我们找到了一个具有所请求 ID 1 的文档和另一个字段 ，`_source`它返回我们从上一步索引的完整 JSON 文档。

### 删除索引

```
DELETE /customer
```

## 更新数据

### 更新文档

除了能够索引和替换文档之外，我们还可以更新文档。请注意，尽管 Elasticsearch 实际上并不在后台进行就地更新。每当我们进行更新时，Elasticsearch 都会删除旧文档，然后一次性为新文档建立索引并应用更新。

```
POST /customer/doc/1/_update
{
   "doc":{"name": "Haha Doe"}
}
```

同时向其添加年龄字段来更新我们之前的文档（ID 为 1）

```
POST /customer/doc/1/_update
{
   "doc": {"name": "Haha Doe", "age": 20}
}
```

也可以使用简单的脚本来执行更新。此示例使用脚本将年龄增加 5：

```
POST /customer/doc/1/_update
{
    "script": "ctx._scouce.age += 5"
}
```

在上面的示例中，`ctx._source`指的是即将更新的当前源文档。

### 删除文档

此示例显示如何删除 ID 为 2 的先前客户：

```
DELETE /customer/doc/2
```

请参阅[`_delete_by_query`API](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/docs-delete-by-query.html)以删除与特定查询匹配的所有文档。值得注意的是，使用 Delete By Query API 删除整个索引比删除所有文档要高效得多。

### 批量处理

除了能够对单个文档进行索引、更新和删除之外，Elasticsearch 还提供了使用[`_bulk`API](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/docs-bulk.html)批量执行上述任何操作的能力。此功能很重要，因为它提供了一种非常有效的机制，可以尽可能快地执行多项操作，同时尽可能减少网络往返次数。

```
POST /customer/doc/_bulk
{"index":{"_id":"1"}}
{"name":"John Doe"}
{"index":{"_id":"2"}}
{"name":"Jane Doe"}
```

此示例更新第一个文档（ID 为 1），然后在一次批量操作中删除第二个文档（ID 为 2）：

```
POST /customer/doc/_bulk
{"update":{"_id":"1"}}
{"doc":{"name":"John Doe becomes Jane Doe"}}
{"delete":"_id":"2"}
```

注意上面的删除动作，后面没有对应的源文档，因为删除只需要要删除的文档的ID。

批量 API 不会因其中一项操作失败而失败。如果单个操作由于某种原因失败，它将继续处理它之后的其余操作。当批量 API 返回时，它将提供每个操作的状态（按照发送顺序），以便您可以检查特定操作是否失败。

## 搜索数据

### 搜索api

运行搜索有两种基本方法：一种是通过[REST 请求 URI发送搜索参数，另一种是通过](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/search-uri-request.html)[REST 请求正文](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/search-request-body.html)发送它们。请求正文方法允许您更具表现力，并以更具可读性的 JSON 格式定义您的搜索。我们将尝试请求 URI 方法的一个示例，但在本教程的其余部分，我们将专门使用请求主体方法。

```
GET /bank/_search?q=*&sort=account_number:asc
```

我们`_search`在银行索引中搜索 (endpoint)，`q=*`参数指示 Elasticsearch 匹配索引中的所有文档。该`sort=account_number:asc`参数表示使用`account_number`每个文档的字段对结果进行升序排序。同样，该`pretty`参数只是告诉 Elasticsearch 返回打印精美的 JSON 结果。

以及响应（部分显示）：

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
    "total" : 1000,
    "max_score" : null,
    "hits" : [ {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "0",
      "sort": [0],
      "_score" : null,
      "_source" : {"account_number":0,"balance":16623,"firstname":"Bradshaw","lastname":"Mckenzie","age":29,"gender":"F","address":"244 Columbus Place","employer":"Euron","email":"bradshawmckenzie@euron.com","city":"Hobucken","state":"CO"}
    }, {
      "_index" : "bank",
      "_type" : "account",
      "_id" : "1",
      "sort": [1],
      "_score" : null,
      "_source" : {"account_number":1,"balance":39225,"firstname":"Amber","lastname":"Duke","age":32,"gender":"M","address":"880 Holmes Lane","employer":"Pyrami","email":"amberduke@pyrami.com","city":"Brogan","state":"IL"}
    }, ...
    ]
  }
}
```

至于响应，我们看到以下部分：

- `took`– Elasticsearch 执行搜索的时间（以毫秒为单位）
- `timed_out`– 告诉我们搜索是否超时
- `_shards`– 告诉我们搜索了多少分片，以及搜索成功/失败的分片数量
- `hits`- 搜索结果
- `hits.total`– 符合我们搜索条件的文档总数
- `hits.hits`– 搜索结果的实际数组（默认为前 10 个文档）
- `hits.sort`- 结果的排序键（如果按分数排序则丢失）
- `hits._score`和`max_score`- 现在忽略这些字段

这是使用替代请求正文方法进行的完全相同的搜索：

```

GET /bank/_search
{
  "query":{"match_all":{}},
  "sort":[
     {"account_number":"asc"}
  ]
}
```

这里的不同之处在于，我们没有传入`q=*`URI，而是向`_search`API 提供了一个 JSON 样式的查询请求主体。我们将在下一节中讨论此 JSON 查询。

重要的是要了解，一旦您获得搜索结果，Elasticsearch 就完全完成了请求，并且不会维护任何类型的服务器端资源或打开游标到您的结果中。这与许多其他平台（例如 SQL）形成鲜明对比，在 SQL 中，您最初可能会预先获得查询结果的部分子集，然后如果要获取（或分页）其余部分，则必须不断返回服务器使用某种有状态服务器端游标的结果。

### 查询语言

Elasticsearch 提供了一种 JSON 风格的领域特定语言，您可以使用它来执行查询。这称为[查询 DSL](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl.html)。查询语言非常全面，乍一看可能令人生畏，但实际学习它的最佳方法是从几个基本示例开始。

回到上一个例子，我们执行了这个查询：

```
GET /bank/_search
{
  "query":{"match_all":{}},
  "sort":[
     {"account_number":"asc"}
  ]
}
```

剖析以上内容，这`query`部分告诉我们查询定义是什么，而这`match_all`部分只是我们要运行的查询类型。`match_all`查询只是搜索指定索引中的所有文档。

除了`query`参数之外，我们还可以传递其他参数来影响搜索结果。上一节的例子中我们传入了 `sort`，这里我们传入`size`：

```
GET /bank/_search
{
  "query":{"match_all":{}},
  "size":1
}
```

请注意，如果`size`未指定，则默认为 10。

此示例执行 `match_all`并返回文档 10 到 19：

```
GET /bank/_search
{
   "query":{"match_all":{}},
   "from":10,
   "size":10
}
```

参数（从`from`0 开始）指定从哪个文档索引开始，`size`参数指定从 from 参数开始返回多少文档。此功能在实现搜索结果分页时很有用。请注意，如果`from`未指定，则默认为 0。

此示例执行 a`match_all`并按帐户余额降序对结果进行排序，并返回前 10 个（默认大小）文档。

```
GET /bank/_search
{
    "query":{"match_all":{}},
    "sort":{"balance":{"order":"desc"}}
}
```

### 执行搜索

现在我们已经了解了一些基本的搜索参数，让我们深入了解 Query DSL。我们先来看看返回的文档字段。默认情况下，完整的 JSON 文档作为所有搜索的一部分返回。这称为源（`_source`搜索命中中的字段）。如果我们不希望返回整个源文档，我们可以请求仅返回源中的几个字段。

此示例显示如何从搜索中返回两个字段`account_number`和`balance`（在 内`_source`）：

```
GET /bank/_search
{
   "query":{"match_all":{}},
   "_source":["account_number","balance"]
}
```

具体查询(match)

```
GET /bank/_search
{
   "query":{"match":{"account_number":10}}
}
```

match_phrase

此示例是`match`( `match_phrase`) 的变体，它返回地址中包含短语“mill lane”的所有帐户：

```
GET /bank/_search
{
  "query":{"match_phrase":{"address":"mill lane"}}
}
```

布尔查询

现在让我们介绍[`bool`查询](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-bool-query.html)。查询允许我们使用布尔逻辑将较小的`bool`查询组合成较大的查询。

此示例包含两个`match`查询并返回地址中包含“mill”和“lane”的所有帐户：must == and

```
GET /bank/_search
{
   "query":{
       "bool":{
          "must":[
             {"match":{"address":"mill"}},
             {"match":{"address":"lane"}}
          ]
       }
   }
}
```

在上面的示例中，该`bool must`子句指定了所有查询必须为真才能将文档视为匹配项。

相反，此示例组成两个`match`查询并返回地址中包含“mill”或“lane”的所有帐户：should == or

```
GET /bank/_search
{
   "query":{
      "bool":{
         "should":{
            {"match":{"address":"mill"}},
            {"match":{"address":"lane"}}
         }
      }
   }
}
```

在上面的示例中，该`bool should`子句指定了一个查询列表，其中任何一个查询都必须为真，文档才会被视为匹配项。

此示例包含两个`match`查询并返回地址中既不包含“mill”也不包含“lane”的所有帐户：

```
GET /bank/_search
{
   "query":{
        "bool":{
            "must_not":{
               {"match":{"address":"mill"}},
               {"match":{"address":"lane"}}
            }
        }
   }
}
```

在上面的示例中，该`bool must_not`子句指定了一个查询列表，其中没有一个查询必须为真才能将文档视为匹配项。

我们可以在查询中同时组合`must`、`should`和`must_not`子句。`bool`此外，我们可以`bool`在这些`bool`子句中的任何一个内编写查询，以模仿任何复杂的多级布尔逻辑。

此示例返回 40 岁但不住在 ID中的任何人的所有帐户：

```
GET /bank/_search
{
  "query":{
     "bool":{
        "must":[
           {"match":{"age":40}}
        ],
        "must_not":[
           {"match":{"state":"ID"}}
        ]
     }
  }
}
```

### 过滤(Filters)

在上一节中，我们跳过了一个称为文档评分（`_score`搜索结果中的字段）的小细节。分数是一个数值，它是文档与我们指定的搜索查询匹配程度的相对度量。分数越高，文档越相关，分数越低，文档越不相关。

但查询并不总是需要产生分数，特别是当它们仅用于“过滤”文档集时。Elasticsearch 会检测这些情况并自动优化查询执行，以免计算出无用的分数。

我们在上一节中介绍的[`bool`查询](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-bool-query.html)还支持`filter`子句，这些子句允许使用查询来限制将被其他子句匹配的文档，而不改变分数的计算方式。作为示例，让我们介绍[`range`查询](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/query-dsl-range-query.html)，它允许我们按一系列值过滤文档。这通常用于数字或日期过滤。

此示例使用 bool 查询返回余额在 20000 和 30000 之间（含）的所有帐户。换句话说，我们要查找余额大于或等于 20000 且小于或等于 30000 的帐户。

```
GET /bank/_search
{
   "query":{
       "bool":{
          "must":{"match_all":{}},
          "filter":{
               "range":{
                  "blance":{
                     "gte":20000,
                     "lte":30000
                  }
               }
          }
       }
   }
}
```

剖析以上内容，bool 查询包含一个`match_all`查询（查询部分）和一个`range`查询（过滤器部分）。我们可以将任何其他查询替换为查询和过滤器部分。在上述情况下，范围查询非常有意义，因为落入该范围内的文档都“同等”匹配，即没有文档比另一个文档更相关。

除了`match_all`、`match`、`bool`和`range`查询之外，还有许多其他可用的查询类型，我们在这里不再赘述。由于我们已经对它们的工作原理有了基本的了解，因此应用这些知识来学习和试验其他查询类型应该不会太困难。

## 映射（Mapping）

### 删除映射类型

在 Elasticsearch 6.0.0 或更高版本中创建的索引可能只包含一个[映射类型](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/mapping.html#mapping-type)。在 5.x 中创建的具有多种映射类型的索引将继续像以前一样在 Elasticsearch 6.x 中运行。Elasticsearch 7.0.0 中的 API 中的类型将被弃用，并在 8.0.0 中完全删除。

### 什么是映射类型

自 Elasticsearch 的第一个版本以来，每个文档都存储在一个索引中并分配一个映射类型。映射类型用于表示被索引的文档或实体的类型，例如 `twitter`索引可能有一个`user`类型和一个`tweet`类型。

每个映射类型都可以有自己的字段，因此`user`类型可能有一个 `full_name`字段、一个`user_name`字段和一个`email`字段，而 `tweet`类型可以有一个`content`字段、一个`tweeted_at`字段和像 `user`类型一样的`user_name`字段。

每个文档都有一个`_type`包含类型名称的元字段，并且可以通过在 URL 中指定类型名称来将搜索限制为一种或多种类型：

```
GET twitter/user,tweet/_search
{
  "query": {
    "match": {
      "user_name": "kimchy"
    }
  }
}
```

该`_type`字段与文档的组合`_id`生成一个`_uid` 字段，因此具有相同类型的不同类型的文档`_id`可以存在于单个索引中。

映射类型还用于在文档之间建立 [父子关系](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/mapping-parent-field.html) ，因此类型文档`question`可以是类型文档的父代`answer`。

### 为什么要删除映射类型

最初，我们所说的“索引”类似于 SQL 数据库中的“数据库”，而“类型”相当于“表”。

这是一个糟糕的类比，导致了错误的假设。在 SQL 数据库中，表是相互独立的。一个表中的列与另一个表中具有相同名称的列无关。对于映射类型中的字段，情况并非如此。

在 Elasticsearch 索引中，不同映射类型中具有相同名称的字段在内部由相同的 Lucene 字段支持。换句话说，使用上面的示例，类型中的`user_name`字段`user`存储在与类型中的字段完全相同的`user_name`字段中`tweet`，并且两个 `user_name`字段在两种类型中必须具有相同的映射（定义）。

这可能会导致挫败感，例如，您希望`deleted`在 同一索引中既是一种`date`类型的字段又是另一种类型的字段。`boolean`

最重要的是，将具有很少或没有共同字段的不同实体存储在同一索引中会导致数据稀疏并干扰 Lucene 有效压缩文档的能力。

出于这些原因，我们决定从 Elasticsearch 中删除映射类型的概念。

### 映射类型的替代方案

**（1）每种文档类型的索引**

第一种选择是为每个文档类型创建一个索引。`twitter`您可以将推文和用户存储在`tweets`索引中，而不是将推文和用户存储在单个索引中`user`。索引之间是完全独立的，因此索引之间不会有字段类型冲突。

这种方法有两个好处：

- 数据更可能是密集的，因此受益于 Lucene 中使用的压缩技术。
- 用于全文搜索评分的术语统计更可能是准确的，因为同一索引中的所有文档都代表一个实体。

每个索引的大小都可以根据它将包含的文档数量进行适当调整：您可以使用较少数量的主分片`users`和较大数量的主分片`tweets`。

**（2）自定义类型字段**

当然，一个集群中可以存在多少个主分片是有限制的，因此您可能不希望将整个分片浪费在仅包含几千个文档的集合上。在这种情况下，您可以实现自己的自定义`type` 字段，其工作方式与旧的`_type`.

让我们以上面的`user`/`tweet`为例。最初，工作流程看起来像这样：

```
PUT twitter
{
   "mappings":{
      "user":{
         "propertiess":{
             "name":{"type":"keyword"},
             "user_name":{"type":"keyword"},
             "email":{"type":"keyword"}
         }
      },
      "tweet":{
         "properties":{
             "content":{"type":"text"},
             "user_name":{"type":"keyword"},
             "tweeted_at":{"type":"date"}
         }
      }
   }
}

PUT twitter/user/kimchy
{
  "name": "shay Banon",
  "user_name": "kimchy",
  "email": "shay@kimchy.com"
}

PUT twitter/tweet/1
{
  "user_name": "kimchy",
  "tweeted_id": "2017-10-24T09:00:00Z",
  "content": "Types are going away"
}

GET twitter/tweet/_search
{
  "query":{
     "match":{
        "user_name": "kimchy"
     }
  }
}
```

可以通过添加自定义类型字段来实现相同的目的：如下所示：

```
PUT twitter
{
  "mapping":{
    "doc": {
      "properties": {
         "type": {"type": "keyword"},
         "name": {"type": "text"},
         "user_name": {"type": "keyword"},
         "email": {"type": "keyword"},
         "content": {"type": "text"}, 
         "tweeted_at": {"type": "date"}
      }
    }
  }
}

PUT twitter/doc/user-kimchy
{
  "type": "user",
  "name": "Shay Banon", 
  "user_name": "Kimchy", 
  "email": "shay@kimchy.com"
}

PUT twitter/doc/tweet-1
{
  "type": "tweet",
  "user_name": "kimchy", 
  "tweeted_at": "2017-10-24T09:00:00Z",
  "content": "Types are going away"
}

GET twitter/_search
{
  "query":{
     "bool":{
       "must":{
          "user_name": "kimchy"
       }
     },
     "filter": {
        "match": {
           "type": "tweet"
        }
     }
  }
}
```

显示类型字段取代了隐式_type字段

### 映射参数

**analyze**

字符串字段的值[`analyzed`](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/mapping-index.html)通过 [分析器](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/analysis.html)传递，以将字符串转换为*标记*或 *术语*流。例如`"The quick Brown Foxes."`，根据使用的分析器，字符串可能会被分析为标记：`quick`, `brown`, `fox`。这些是为该字段编制索引的实际术语，这使得*在* 大文本块中有效地搜索单个单词成为可能。

这个分析过程不仅需要在索引时发生，还需要在查询时发生：查询字符串需要通过相同（或类似）的分析器，以便它试图找到的术语与那些术语的格式相同存在于索引中。

Elasticsearch 附带了许多[预定义的分析器](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/analysis-analyzers.html)，无需进一步配置即可使用。它还附带许多 [character filters](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/analysis-charfilters.html)、[tokenizers](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/analysis-tokenizers.html)和[*Token Filters*](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/analysis-tokenfilters.html)，它们可以组合起来为每个索引配置自定义分析器。

可以为每个查询、每个字段或每个索引指定分析器。在索引时，Elasticsearch 将按以下顺序查找分析器：

- 在`analyzer`字段映射中定义。
- `default`在索引设置中 命名的分析器。
- 分析[`standard`](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/analysis-standard-analyzer.html)仪。

在查询时，还有几层：

- 在[全文查询](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/full-text-queries.html)`analyzer`中定义。
- 在`search_analyzer`字段映射中定义。
- 在`analyzer`字段映射中定义。
- `default_search`在索引设置中 命名的分析器。
- `default`在索引设置中 命名的分析器。
- 分析[`standard`](https://www.elastic.co/guide/en/elasticsearch/reference/6.0/analysis-standard-analyzer.html)仪。

为特定字段指定分析器的最简单方法是在字段映射中定义它，如下所示：

```
PUT /my_index
{
  "mappings": {
    "my_type": {
       "properties": {
          "text": {  // 1 使用默认的standard analyzer
             "type": "text",
             "fields": {
                "english": {  //2 使用 English analyzer
                  "type": "type", 
                  "analyze": "english"
                }
             }
          }
       }
    }
  }
}

GET my_index/_analyze  //3 返回the, quick, brown, foxes
{
  "field": "text",
  "text": "The quick Brown Foxes."
}

GET my_index/_analyze  //4 返回 quick, brown, fox
{
  "field": "text.english",
  "text": "The quick Brown Foxes."
}
```














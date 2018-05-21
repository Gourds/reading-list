---
title: ES权威指南【阅读笔记】
date: 2018-05-21 10:05:16
tags: [读书]
---

[ES权威指南](https://es.xiaoleilu.com)

Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎。目前Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。要使用Lucene这个引擎库，必须要使用Java作为开发语言并将其直接集成到应用中，这个过程十分复杂。而ES就是使用Java开发并使用Lucene作为核心来实现索引和搜索功能，但是它通过简单的RESTful API隐藏了Lucene的复杂性，从而使搜索变得简单。

那ES总结来说可以归纳以下几点：
>- 分布式的实时文件存储，每个字段都被索引并可被搜索
>- 分布式的实时分析搜索引擎
>- 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

<!--more-->

## 基础

### 与ES交互

#### JAVA API

ES为Java用户内置两种类型客户端：节点客户端（node client）及传输客户端（Transport client）。
两个Java客户端都通过9300端口与集群交互，使用Elasticsearch传输协议(Elasticsearch Transport Protocol)。集群中的节点之间也通过9300端口进行通信。如果此端口未开放，你的节点将不能组成集群。
>- **节点客户端**
>节点客户端以无数据节点(none data node)身份加入集群，换言之，它自己不存储任何数据，但是它知道数据在集群中的具体位置，并且能够直接转发请求到对应的节点上
>- **传输客户端**
>这个更轻量的传输客户端能够发送请求到远程集群。它自己不加入集群，只是简单转发请求给集群中的节点。

#### 基于HTTP协议

基于http协议，以Json为数据交互格式的RESTful API。所有程序语言都可以使用RESTful API，通过9200端口的与Elasticsearch进行通信。


### 面向文档

Elasticsearch是面向文档(document oriented)的，这意味着它可以存储整个对象或文档(document)。然而它不仅仅是存储，还会索引(index)每个文档的内容使之可以被搜索。在Elasticsearch中，你可以对文档（而非成行成列的数据）进行索引、搜索、排序、过滤。这种理解数据的方式与以往完全不同，这也是Elasticsearch能够执行复杂的全文搜索的原因之一。

ELasticsearch使用Javascript对象符号(JavaScript Object Notation)，也就是JSON，作为文档序列化格式。JSON现在已经被大多语言所支持，而且已经成为NoSQL领域的标准格式。它简洁、简单且容易阅读。

### 索引

在Elasticsearch中存储数据的行为就叫做索引(indexing)。在Elasticsearch中，文档归属于一种类型(type),而这些类型存在于索引(index)中，我们可以画一些简单的对比图来类比传统关系型数据库。
```xml
Relational DB -> Databases -> Tables -> Rows -> Columns
Elasticsearch -> Indices   -> Types  -> Documents -> Fields
```
Elasticsearch集群可以包含多个索引(indices)（数据库），每一个索引可以包含多个类型(types)（表），每一个类型包含多个文档(documents)（行），然后每个文档包含多个字段(Fields)（列）

索引含义区分：
>- **索引（名词） **
>如上文所述，一个索引(index)就像是传统关系数据库中的数据库，它是相关文档存储的地方，index的复数是indices 或indexes。
>- **索引（动词）**
>「索引一个文档」表示把一个文档存储到索引（名词）里，以便它可以被检索或者查询。这很像SQL中的INSERT关键字，差别是，如果文档已经存在，新的文档将覆盖旧的文档。
>- **倒排索引**
>传统数据库为特定列增加一个索引，例如B-Tree索引来加速检索。Elasticsearch和Lucene使用一种叫做倒排索引(inverted index)的数据结构来达到相同目的。

### 搜索

简单的可以使用查询字符串，复杂的可以使用DSL查询。
- 查询字符串
```bash
GET /megacorp/employee/_search?q=last_name:Smith
```
- DSL查询
```bash
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

使用`match`进行匹配，会进行相关性匹配，匹配结果会有一个分数，匹配度越高分数越高。
使用`match_phrase`可以进行匹配短语

- 聚合
Elasticsearch有一个功能叫做聚合(aggregations)，它允许你在数据上生成复杂的分析统计，类似SQL中的`GROUP BY`
```bash
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": { "field": "interests" }
    }
  }
}
```

### 分布式

Elasticsearch为分布式而生，可以扩展到成百上千的服务器来处理PB级别的数据。而且隐藏了分布式系统本身的复杂性。以下操作都是在底层自动完成的。
>- 将文档分区到不同的容器或者分片中，他们可以存在于一个或多个节点中
>- 将分片均匀的分配到各个节点，对索引和搜索做负载均衡
>- 冗余每一个分片，防止硬件故障造成的数据丢失
>- 将集群中任意一个节点上的请求路由到相应数据所在的节点
>- 无论是增加还是移除节点，分片都可以做到无缝扩展和迁移

## 分布式集群

### 集群要点

ES用于构建高可用和可扩展的系统，扩展的方式可以是购买更好的服务器（纵向扩展Vertical scale or scalingup）或者购买更多的服务器（横向扩展 horizontal scale or scaling out）

一个节点(node)就是一个Elasticsearch实例，而一个集群(cluster)由一个或多个节点组成，它们具有相同的cluster.name，它们协同工作，分享数据和负载。当加入新的节点或者删除一个节点时，集群就会感知到并平衡数据

集群中一个节点会被选举为主节点(master),它将临时管理集群级别的一些变更，例如新建或删除索引、增加或移除节点等。主节点不参与文档级别的变更或搜索，这意味着在流量增长的时候，该主节点不会成为集群的瓶颈。任何节点都可以成为主节点

在Elasticsearch集群中可以监控统计很多信息，但是只有一个是最重要的：集群健康(cluster health)。集群健康有三种状态：green、yellow或red

|颜色|描述|
|---|---|
|green|所有主分片和复制分片都可用|
|yellow|主分片都可以，但不是所有的复制分片都可用|
|red|不是所有的主分片都可以用|

为了将数据添加到Elasticsearch，我们需要索引(index)——一个存储关联数据的地方。实际上，索引只是一个用来指向一个或多个分片(shards)的“逻辑命名空间(logical namespace)”.

一个分片(shard)是一个最小级别“工作单元(worker unit)”,它只是保存了索引中所有数据的一部分。分片就是一个Lucene实例，并且它本身就是一个完整的搜索引擎，我们的文档存储在分片中，并且在分片中被索引，但是我们的应用程序不会直接与它们通信，取而代之的是，直接与索引通信。

分片是Elasticsearch在集群中分发数据的关键。把分片想象成数据的容器。文档存储在分片中，然后分片分配到你集群中的节点上。当你的集群扩容或缩小，Elasticsearch将会自动在你的节点间迁移分片，以使集群保持平衡

分片可以是主分片(primary shard)或者是复制分片(replica shard)。你索引中的每个文档属于一个单独的主分片，所以主分片的数量决定了索引最多能存储多少数据。当索引创建完成的时候，主分片的数量就固定了，但是复制分片的数量可以随时调整。复制分片只是主分片的一个副本，它可以防止硬件故障导致的数据丢失，同时可以提供读请求，比如搜索或者从别的shard取回文档。

### 故障解决

如果一个节点补不能使用，且这个节点含主分片且是Master节点，ES集群会立即在剩余节点选举出新的master节点，并且很快将将不能使用节点上主分片在其他节点上的复制分片迅速升级为主分片（这个升级是瞬间的）。如果当挂掉的这个节点再次回复的时候，这时这个节点仍旧有之前的旧分片，ES会尝试再次利用他们，不过它会从主分片上复制在故障期间有数据变动的那一部分。

## 数据

对象(object)是一种语言相关，记录在内存中的的数据结构。为了在网络间发送，或者存储它，我们需要一些标准的格式来表示它。JSON (JavaScript Object Notation)是一种可读的以文本来表示对象的方式。它已经成为NoSQL世界中数据交换的一种事实标准。当对象被序列化为JSON，它就成为JSON文档(JSON document)了

Elasticsearch是一个分布式的文档(document)存储引擎。它可以实时存储并检索复杂数据结构——序列化的JSON文档。换言说，一旦文档被存储在Elasticsearch中，它就可以在集群的任一节点上被检索

### 文档

程序中大多的实体或对象能够被序列化为包含键值对的JSON对象，键(key)是字段(field)或属性(property)的名字，值(value)可以是字符串、数字、布尔类型、另一个对象、值数组或者其他特殊类型，比如表示日期的字符串或者表示地理位置的对象。在Elasticsearch中，文档(document)这个术语有着特殊含义。它特指最顶层结构或者根对象(root object)序列化成的JSON数据（以唯一ID标识并存储于Elasticsearch中）。

#### 文档元数据

一个完整文档包含元数据和数据，元数据（metadata）其实就是关于文档的信息，ES中有三个必须的元数据、`_index`（文档存储的地方）、`_type`（文档代表的对象的类）、`_id`（文档的唯一标识）

- `_index`
索引(index)类似于关系型数据库里的“数据库”——它是我们存储和索引关联数据的地方。事实上，我们的数据被存储和索引在分片(shards)中，索引只是一个把一个或多个分片分组在一起的逻辑空间。

- `_type`
在关系型数据库中，我们经常将相同类的对象存储在一个表里，因为它们有着相同的结构。同理，在Elasticsearch中，我们使用相同类型(type)的文档表示相同的“事物”，因为他们的数据结构也是相同的。每个类型(type)都有自己的映射(mapping)或者结构定义，就像传统数据库表中的列一样。所有类型下的文档被存储在同一个索引下，但是类型的映射(mapping)会告诉Elasticsearch不同的文档如何被索引

- `_id`
id仅仅是一个字符串，它与_index和_type组合时，就可以在Elasticsearch中唯一标识一个文档。当创建一个文档，你可以自定义_id，也可以让Elasticsearch帮你自动生成。

- 其他元数据

#### 索引文档

创建一个文档可以使用自动生成的ID或者自己手动指定ID，手动指定如下
```bash
PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}
```

索引被成功创建后，这个索引中包含_index、_type和_id元数据，以及一个新元素：_version。Elasticsearch中每个文档都有版本号，每当文档变化（包括删除）都会使_version增加。

#### 检索文档

```bash
GET /{index}/{type}/{id}?pretty  #获取并展示友好的返回格式
curl -i -XGET /{index}/{type}/{id}?pretty #获取curl的响应头
GET /{index}/{type}/{id}/_source #只显示source部分
GET /{index}/{type}/{id}?_source=title,text #获取文档指定的部分
#使用HEAD方法来检索文档是否存在
curl -i -XHEAD http://localhost:9200/{index}/{type}/{id}
```

#### 更新文档

可以使用PUT再次传输文档，相当于覆盖（即更新整个文档），ES会将version加1，在内部，Elasticsearch已经标记旧文档为删除并添加了一个完整的新文档。旧版本文档不会立即消失，但你也不能去访问它。Elasticsearch会在你继续索引更多数据时清理被删除的文档。

局部更新可以使用update这个API。文档是不可变的——它们不能被更改，只能被替换。update API必须遵循相同的规则
```bash
POST /website/blog/1/_update
{
   "doc" : {
      "tags" : [ "testing" ],
      "views": 0
   }
}
```

#### 检索多个文档

使用`_mget`API可以检索多个文档，检索多个文档的速度依然非常快，合并多个请求可以避免每个单独请求的网络开销。
```bash
POST /_mget
{
   "docs" : [
      {
         "_index" : "website",
         "_type" :  "blog",
         "_id" :    2
      },
      {
         "_index" : "website",
         "_type" :  "pageviews",
         "_id" :    1,
         "_source": "views"
      }
   ]
}
```

#### 创建一个新文档

由于`index/type/id`这三个元数据会确定一个唯一索引，所以最简单的方式，就是使用POST方法让ES自动生成唯一的`_id`
```bash
POST /website/blog/
{ ... }
```
如果使用自定义`_id`就需要高速ES什么时候应该接受请求，有两种方法，创建成功返回的状态码应该是201已创建，文档已存在的话会返回409
```bash
#方法1
PUT /website/blog/123?op_type=create
{ ... }
#方法2
PUT /website/blog/123/_create
{ ... }
```

#### 删除文档

使用`DELETE`方法
```bash
DELETE /{index}/{type}/{id}/
```

### 批量

#### 多大算大

整个批量请求需要被加载到接受请求的节点的内存里，所以请求越大，给其他请求可用的内存就越小，有一个最佳bulk请求的大小。超过这个大小，性能不能再提升反而可能下降，这个最佳大小是多少，取决于硬件、文档的大小和复杂度以及索引和搜索的负载

测试这个大小，可以批量索引标准的文档，当随着大小的增长性能开始降低时说明你每个批次的大小太大了，开始的数量可以在1000-5000个文档之间，如果文档非常大，可以使用较小的批次。

通常着眼于你请求批次的物理大小是非常有用的。1000个1kb的文档和1000个1MB的文档大不相同。一个好的批次最好保持在5-15MB之间。

## 分布式增删改查

#### 路由文档到分片

当创建一个新文档时，它被存储在一个单独一个主分片上，ES通过以下算法决定如何存储
```bash
shard = hash(routing) % number_of_primary_shards
```
`routing`是一个任意的字符串，它默认是`_id`但也可以自定义。这个routing通过哈希函数生成一个数字，然后除以主切片的数量得到一个余数，余数的范围永远是0到`number_of_primary_shards -1`,这个数字就是特定文档所在的分片。这也是为什么主分片的数量只能在创建索引时定义且不能修改。

#### 管理文档

新建、索引、和删除请求都是些（write）操作，他们必须在主分片上成功完成才能复制到相关的复制分片上。
客户端接收到成功响应的时候，文档的修改已经被应用于主分片和所有的复制分片。你的修改生效了
**几个优化点**
>- 修改`replication`的参数将`sync`修改为`async`
>牺牲安全性，提升响应速度，使用sync时主分片得到复制分片成功的响应后才会返回，使用`async`时主分片会直接返回
>- 修改`consistency`参数
>- 修改`timeout`参数
>当分片副本不足时会怎样？Elasticsearch会等待更多的分片出现。默认等待一分钟。如果需要，你可以设置timeout参数让它终止的更早：100表示100毫秒，30s表示30秒。


## 搜索

**几个概念**
>- 映射（Mapping）：数据在每个字段中的解释说明
>- 分析（Analysis）：全文是如何处理的可以被搜索的
>- 特定语言查询（Query DSL）：ES使用的灵活强大的查询语言

### 空搜索

最基本的搜索API就是空搜索（empty search），没有指定的查询条件，只返回集群索引中所有的文档
```bash
GET /_search
```

#### hits

响应中最重要的部分是`hits`，它包含了表示匹配到文档总数的`total`及匹配到的前10条数据`hits`。

hits数组中的每个结果都包含`_index`、`_type`、和文档的`_id`字段，被加入到`source`字段的意味着在搜索结果中我们将可以直接使用全部文档，这不像其他的搜索引擎只返回文档ID，需要你单独去获取文档。

没一个节点都有一个`_source`字段，表示相关性得分（relevance score），它用来衡量文档与查询的匹配程度。默认，评分最高的文档排在首位

#### took

`took`用来表示整个搜索请求花费的毫秒数

### 多索引搜索

```bash
/_search #在所有索引所有类型中搜索
/gb/_search #在索引gb的所有类型中搜索
/gb,us/_search #在索引gb和us的所有类型中搜索
/g*,u*/_search #在以g或u开头的所有索引的所有类型中搜索
```

### 分页

ES接收`from`和`size`参数；size表示结果书默认为10，from表示跳过开始的结果书，默认0
如想每页显示5个，页码从1到3，请求如下
```bash
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

### 查询字符串

`search`API有两种表单：一种是简易版的查询字符串（query string）将所有参数通过查询字符串定义，另一种版本使用JSON完整的表示请求体（request body），这种富搜索语言叫做结构化查询语句（DSL）










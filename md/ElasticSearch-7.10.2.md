---
title: ElasticSearch 7.10.2 离线文档
author: https://github.com/youthred
header: ElasticSearch 7.10.2
footer: ${pageNo}
---

[TOC]

# 开始

当您在 Elasticsearch 服务上创建部署时，会自动配置一个主节点和两个数据节点。

## 下载软件包

- [Linux](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz) https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz

- [MacOS](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-darwin-x86_64.tar.gz) https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-darwin-x86_64.tar.gz
- [Windows](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-windows-x86_64.zip) https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-windows-x86_64.zip

## 从`bin`目录启动

### 单例模式

Linux & MacOS

``` sh
cd elasticsearch-7.10.2/bin
./elasticsearch
```

Windows

``` sh
cd elasticsearch-7.10.2\bin
.\elasticsearch.bat
```

### 集群模式

再启动两个实例，需要为每个节点指定唯一的数据和日志路径。

Linux & MacOS

```sh
./elasticsearch -Epath.data=data2 -Epath.logs=log2
./elasticsearch -Epath.data=data3 -Epath.logs=log3
```

Windows

```powershell
.\elasticsearch.bat -E path.data=data2 -E path.logs=log2
.\elasticsearch.bat -E path.data=data3 -E path.logs=log3
```

附加节点被分配有唯一的 ID，此新增的两个节点会自动加入第一个单例节点的集群。

### 查看集群健康

Kibana

``` json
GET /_cat/health?v=true
```

### cURL命令

``` sh
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```

- VERB

  HTTP请求方法，如`GET`、`POST`、 `PUT`、`HEAD` 或`DELETE`

- PROTOCOL

  `HTTP`获取`HTTPS`

- HOST

  集群任一节点主机名

- PORT

  HTTP服务端口，默认`9200`

- PATH

  API，如`_search`、`_cluster/stats`或`_nodes/stats/jvm`

- QUERY_STRING

  查询参数字符串，如`?pretty`可格式化JSON

- BODY

  JSON请求体

## 创建索引

### PUT请求

``` json
PUT /customer/_doc/1
{
  "name": "John Doe"
}
```

**`customer` 为索引名称，不存在则自动创建**

**`_doc` 默认值**

**`1` 插入文档ID**

响应

``` json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 26,
  "_primary_term" : 4
}
```

### GET请求

使用`GET`请求获取文档

指定ID

``` json
GET /customer/_doc/1
```

响应

``` json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 26,
  "_primary_term" : 4,
  "found" : true,
  "_source" : {
    "name": "John Doe"
  }
}
```

## 搜索

### match_all

> 使用`DSL`语法进行查询

```json
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
```

响应，默认情况下只返回前10个文档

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

- `took`

  Elasticsearch 运行查询所花费的时间（毫秒）

- `timed_out`

  搜索请求是否超时

- `_shards`

  搜索了多少分片，以及成功、失败或跳过了多少分片的详细信息

- `max_score`

  找到的最相关文档的分数

- `hits.total.value`

  找到了多少匹配的文档

- `hits.sort`

  文档的排序位置（不按相关性分数排序时）

- `hits._score`

  文档的相关性分数（使用`match_all`时不适用）

### 使用`from`和`size`翻阅搜索结果

eg. 查询第10到19个文档

```json
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}
```

### 更有趣的查询

- `match` 默认分词匹配

  匹配`mill lane`、`mill`或`lane`

  ```json
  GET /bank/_search
  {
    "query": { "match": { "address": "mill lane" } }
  }
  ```

- `match_phrase` 短语匹配

  仅匹配`mill lane`

  ```json
  GET /bank/_search
  {
    "query": { "match_phrase": { "address": "mill lane" } }
  }
  ```

- 布尔查询`bool`组合多个查询条件

  eg. 搜索属于40岁客户的账户，但排除居住在Idaho(以ID表示)

  ```json
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

  布尔查询中的每个`must`、`should`、 和`must_not`元素称为查询子句。文档满足每个`must`或 条款中的标准的程度`should`会影响文档的相关性得分。分数越高，文档越符合您的搜索条件。默认情况下，Elasticsearch 返回按这些相关性分数排名的文档

  子句条件不会影响评分，只会影响文档是否包含在结果中

  eg. 使用范围过滤器将结果限制为余额在 20,000 美元到 30,000 美元（含）之间的账户

  ```json
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

## 聚合分析

### terms

eg. 分组计数

```json
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
```

响应，降序返回前10

```json
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

### 嵌套聚合

分组计数`terms`后统计平均数`avg`

```json
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

### 自定义排序

根据嵌套聚合结果对上一层聚合进行排序

```json
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

除了这些基本的分桶和指标聚合之外，Elasticsearch 还提供专门的聚合，用于在多个字段上操作并分析特定类型的数据，例如日期、IP 地址和地理数据。您还可以将各个聚合的结果输入到管道聚合中以进行进一步分析

# 设置

## 安装

### Archive

- Linux

  ```sh
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-linux-x86_64.tar.gz.sha512
  shasum -a 512 -c elasticsearch-7.10.2-linux-x86_64.tar.gz.sha512
  tar -xzf elasticsearch-7.10.2-linux-x86_64.tar.gz
  cd elasticsearch-7.10.2/
  ```

- MacOS

  ```sh
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-darwin-x86_64.tar.gz
  wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.10.2-darwin-x86_64.tar.gz.sha512
  shasum -a 512 -c elasticsearch-7.10.2-darwin-x86_64.tar.gz.sha512
  tar -xzf elasticsearch-7.10.2-darwin-x86_64.tar.gz
  cd elasticsearch-7.10.2/
  ```

从命令行运行 Elasticsearch

```sh
./bin/elasticsearch
```

查看运行状态 `localhost:9200/?pretty`

成功响应

```json
{
  "name" : "Cp8oag6",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",
  "version" : {
    "number" : "7.10.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "f27399d",
    "build_date" : "2016-03-30T09:51:41.449Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "1.2.3",
    "minimum_index_compatibility_version" : "1.2.3"
  },
  "tagline" : "You Know, for Search"
}
```

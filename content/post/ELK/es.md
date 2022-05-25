---
title: "Es v7.15.2"
date: 2022-04-14T21:51:35+08:00
draft: true
---
# 配置
#第一次启动集群设置主节点，重启时不要使用
````yaml
cluster.initial_master_nodes:
  - master-node-a
````
# DSL语句使用
## 新增数据

指定了index(索引库名称 test1)和type(类型)即可。在新增的时候你可以自己指定主键ID(1)，也可以不指定，由 ElasticSearch自身生成
````json
POST test1/_doc/1
{
    "uid" : "1234",
    "phone" : "12345678909",
    "message" : "qq",
    "msgcode" : "1",
    "sendtime" : "2019-03-14 01:57:04"
}
````
可通过 GET test1/ 或 GET test1/_settings和GET test1/_mapping查看该index的状态，也就是 setting(设置选项) 和mapping(数据结构)。
es创建了索引库test1，同时新增了一条数据。
没有创建索引库而通过ES自身生成的这种并不友好，因为它会使用默认的配置，字段结构都是text(text的数据会分词，在存储的时候也会额外的占用空间)，分片和索引副本采用默认值，默认是5和1，ES的分片数在创建之后就不能修改，除非reindex(下面会讲到)，所以这里我们还是指定数据模板进行创建。

## 创建索引库

````json
PUT test2
{
  "settings" : {
      "number_of_shards" : 10,
      "number_of_replicas" : 1,
        "refresh_interval" : "1s"
  },
  "mappings" : {
    "properties" : {
        "uid" : { "type" : "long" },
        "phone" : { "type" : "long" },
        "message" : { "type" : "keyword" },
        "msgcode" : { "type" : "long" },
          "sendtime" : {  
          "type" : "date",
          "format" : "yyyy-MM-dd HH:mm:ss" 
        }
    }
  }
}
````
版本不同有可能不错，根据提示与get test1/查看已有的索引库数据结构进行修改
数据结构见https://www.elastic.co/guide/en/elasticsearch/reference/7.15/mapping-types.html
number_of_shards: 是设置的分片数，设置之后无法更改！
refresh_interval: 是设置es缓存的刷新时间，如果写入较为频繁，但是查询对实时性要求不那么高的话，可以设置高一些来提升性能。可以更改
number_of_replicas : 是设置该索引库的副本数，建议设置为1以上。
其中这里还有几个重要参数也顺便说一下:

store: true/false 表示该字段是否存储，默认存储。
doc_values: true/false 表示该字段是否参与聚合和排序。
index: true/false 表示该字段是否建立索引，默认建立


## 修改数据
根据主键修改的命令示例:
同创建数据类似，存在新增，不存在修改
````json
POST test1/_doc/1
{
    "uid" : "1234",
    "phone" : "12345678909",
    "message" : "qq",
    "msgcode" : "1",
    "sendtime" : "2019-03-14 01:57:04"
}
````
根据条件修改的命令示例:
````json
POST test1/_update_by_query
{
  "query": {
    "term": {
      "phone": "12345678909"
    }
  } ,
  "script": {
    "source": "ctx._source['message'] = 'xuwujing'"
  }
}
````
## 删除数据、字段和索引库
ES根据主键删除数据的命令示例是DELETE 索引库/id，简单实用，但是一定要要加上ID，不然就是删除索引库了！

根据主键删除数据命令示例：
````json
DELETE test1/_doc/1
````
根据条件删除数据的命令示例：
````json
POST test/_delete_by_query
{
  "query": {
      "term": {
        "phone": "12345678909"
      }
  }
}
````
## 查询
### 查询索引库所有的数据
命令格式为GET 索引库名称/索引库类型/_search，也可以不需要索引库类型。
````json
GET  test1/_doc/_search
GET  test1/_doc/id
````
### 等值（term）查询
````json
GET  test1/_doc/_search
{
  "query": {
    "term": {
      "phone": "12345678909"
    }
  }
}
````
一个字段匹配多个值的话，可以使用terms，相当于SQL的in语法。
主键用_id
````json
GET  test1/_doc/_search
{
  "query": {
    "terms": {
       "uid": [ 
        1234, 
        12345, 
        123456
      ] 
    }
  }
}
````
### 范围（range ）查询
range可以理解为SQL中的><符号，其中gt是大于，lt是小于，gte是大于等于，lte是小于等于。
````json
GET  test1/_doc/_search
{
  "query": {
   "range": { 
      "uid": { 
        "gt": 1234,
        "lte": 12345
      } 
    } 
  }
}
````
### 组合(bool)查询
bool 可以用来合并多个过滤条件查询结果的布尔逻辑，它包含这如下几个操作符:

must : 多个查询条件的完全匹配,相当于 and。
must_not ::多个查询条件的相反匹配，相当于 not。
should : 至少有一个查询条件匹配, 相当于 or。
查询的命令示例:
````json
GET /test1/_search
{
  "query": {
    "bool": {
      "must": {
        "term": {
          "phone": "12345678909"
        }
      },
      "must_not": {
        "term": {
          "uid": 12345
        }
      },
      "should": [
        {
          "term": {
            "uid": 1234
          }
        },
        {
          "term": {
            "uid": 123456
          }
        }
      ],
      "adjust_pure_negative": true,
      "boost": 1
    }
  }
}
````
### 模糊(wildcard)查询
wildcard查询相当于SQL语句中的like语法，只不过它查询的数据需要加上*符号。中文不相当于like
模糊查询命令示例:
````json
GET /test1/_search
{
  "query": {
   "wildcard": { 
       "message":"*wu*" 
    } 
  }
}
````
### 正则(regexp)查询
regexp可以支持正则查询，比如查询短信内容中的验证码之类的。
下面的这个示例就是查询以xu开头，后面是0-9数字的内容的数据。

正则查询命令示例:
````json
GET /test1/_search
{
  "query": {
   "regexp": { 
       "message":"xu[0-9]" 
    } 
  }
}
````
### 通用的查询[query_string](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/query-dsl-query-string-query.html)
忽略大小写
username like '%ab%'
````json
GET user/_search
{
  "query": {
      "query_string": {
        "query": "username: ab"
      }
  }
}
````
> username中有中文时，中文结果跟预想不一样，包含了一个也会返回
> ![x](/images/es/ch.png)

username in ('..ab..', '...cd...' )
OR必须大写
````json
GET user/_search
{
  "query": {
      "query_string": {
        "query": "username: ab OR cd"
      }
  }
}
````
username = '%ab%' and name = '%cd%'
````json
GET user/_search
{
  "query": {
      "query_string": {
        "query": "username: ab AND name:cd"
      }
  }
}
````
之后以为sql中的like即username like '%ab%'会与下面的查询完全等价
````json
GET user/_search
{
  "query": {
      "query_string": {
        "query": "username: *ab*"
      }
  }
}
````
由于user表是本地随机产生的一个千万级的索引，对username列count了一次，sql与es两边的结果相等。但随后对name列count了一次，发现结果相差挺大的。name列数据中含有大量中文，而username不含。之后使用match_phrase匹配词组查询
````json
GET user/_search
{
  "query": {
      "match_phrase": {
        "name": "丸"
      }
  }
}
````
再次与sql中的结果进行对比，这次是一致的。
但是当match_phrase又对username列使用时
````json
GET user/_search
{
  "query": {
      "match_phrase": {
        "username": "ab"
      }
  }
}
````
与sql的结果进行对比又不同。所以到底完全等价于like查询应该是怎样的呢？

到此发现从头开始的理解就错了。这里的查询是分词查询，英文是以空格为单位将内容分成多个单词进行匹配，而不是单个字符，中文则是以单个字为单位。所以match_phrase:{username:ab}是将username的内容进行分词后，每一个词是否等于ab，若其中有一个等于ab则返回。

例如对于username: 'ab cc dd',match_phrase:{username:ab} 会命中

对于username: 'abc cc dd',match_phrase:{username:ab} 不会命中

对于username: '你好啊',match_phrase:{username:你好} 会命中，你好啊中文分词是 '你'，'好'，'啊'，你好分词是'你'，'好'，
分别相等，且顺序一致

对于username: '你好啊',match_phrase:{username:你啊} slop属性，默认为0，不会命中。顺序不一致

query_string:{query: "username:ab"}是将username的内容进行分词后，每一词是否等于ab，若其中有一个等于ab则返回。

query_string:{query: "username:*ab*"}是将username的内容进行分词后，每一词是否等于*ab*，将*ab*整体作为一个词与原内容每个分词进行匹配，若其中有一个等于*ab*则返回。用中文做对比
username: 你好啊 //分词后是'你'，'好'，'啊'
query_string:{query: "username:你好"} 命中  //分词后是'你'，'好'
query_string:{query: "username:*你好*"}不会命中，//分词后是'*你好*'
用英文
username: i like she //分词后是'i'，'like'，'she'
query_string:{query: "username:li"} 命中  //分词后是'li'
query_string:{query: "username:*li*"}会命中，//分词后是'*li*'

所以到此发现query_string与match_phrase都无法完全等于sql中的like

### 统计
_search改为_count即可
````json
GET user/_count
{
  "query": {
      "query_string": {
        "query": "username: ab AND name:cd"
      }
  }
}
````
# 参考
1.[ElasticSearch实战系列二: ElasticSearch的DSL语句使用教程---图文详解 
](https://www.cnblogs.com/xuwujing/p/11567053.html)
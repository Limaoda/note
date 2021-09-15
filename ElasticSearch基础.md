ES HTTP操作索引只支持PUT,DELETE,HEAD,GET不支持POST，因为POST不具有幂等性，而ES不可能存在相同的索引

#### 索引-创建

PUT请求  http://127.0.0.1:9200/{indexName}

#### 索引-查询

GET请求 http://127.0.0.1:9200/{indexName}

#### 索引-查询全部

GET请求 http://127.0.0.1:9200/_cat/indices?v

#### 索引-删除

DELETE请求 http://127.0.0.1:9200/{indexName}

#### 文档-创建

POST请求 http://127.0.0.1:9200/{indexName}/{docName} `请求方式只能是POST且必须要有body，请求体格式为json`

```json
// body
{
  "title":"证券时报xxxxx",
  "constract":"xx公司上市xxxxxxx",
  "content":"xx公司上市xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

> ES创建文档后会为该条文档分配一个随机的id，而PUT请求具有幂等性，违背ES文档创建原则，因此在创建文档时不允许使用PUT

#### 文档-创建-自定义文档id

POST/PUT请求 http://127.0.0.1:9200/{indexName}/{docName}/{id}  `必须要有body，请求体格式为json`

```json
// body
{
  "title":"证券时报xxxxx",
  "constract":"xx公司上市xxxxxxx",
  "content":"xx公司上市xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

> 这里可以为PUT请求，因为已经具备幂等性条件

#### 文档-主键查询

GET请求 http://127.0.0.1:9200/{indexName}/{docName}/{id}

#### 文档-查询全部

GET请求 http://127.0.0.1:9200/{indexName}/_search

#### 文档-全量更新

PUT请求 http://127.0.0.1:9200/{indexName}/{docName}/{id}  `body需包含修改和未修改，请求体格式为json`

```json
// body
{
  "title":"证券时报yyyyy",
  "constract":"xx公司上市xxxxxxx",
  "content":"xx公司上市xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

#### 文档-局部更新

POST请求 http://127.0.0.1:9200/{indexName}/{_docName}/{id}   `body只需包含修改部分，请求体格式为json`

```json
// body
{
  "doc":{
      "title":"证券时报yyyyy"
  }
}
```

#### 文档-条件查询

GET请求 url携参 http://127.0.0.1:9200/{indexName}/_search?q={key}:{value}

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
//body
{
    "query":{
        "match":{
            "key":"value"
        }
    }
}
```

#### 文档-分页查询

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
//body
{
  "query":{
    "match_all":{
   
    }
  },
  "from":0, //偏移量
  "size":2 //单页数据量
}
```

#### 文档-筛选查询结果

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
//body
{
  "query":{
    "match_all":{
   
    }
  },
  "from":0, //偏移量
  "size":5, //单页数据量
  "_source":["title"] //只查询title字段
}
```

#### 文档-排序

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
//body
{
  "query":{
    "match_all":{
   
    }
  },
  "sort":{
    "order":{             //按照order字段排序(order为整型)
      "order":"DESC"      //指定排序方式
    }
  }
}  

//body
{
  "query":{
    "match_all":{
   
    }
  },
  "sort":{
    //按照updatetime字段排序(updatetime为字符型)
    "updatetime.keyword":{   
      "order":"DESC" //指定排序方式
    }
  }
}  
```

#### 文档-多条件查询

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
//body
{
  "query":{
    "bool":{
      //查询结果集必须同时满足两个条件
      "must":[
        {
          "match":{
            "title":"环球"
          }
        },
        {
          "match":{
            "content":"公司"
          }
        }
      ],
      //可选条件
      "should":[
        {
          "match":{
            "title":"欢乐家"
          }
        }
      ]
    }
  }
}
```

#### 文档-多条件查询-范围过滤

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
{
  "query":{
    "bool":{
      "filter":{
        "range":{
          "updatetime.keyword":{
             //匹配updatetime大于2021-9-15 14:10:00的结果集
            "gt":"2021-9-15 14:10:00"  
          }
        }
      }
    }
  }
}
```



#### 文档-完全匹配查询

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
{
    "query":{
        "match_phrase":{
            //只匹配title有"乐家"的结果集
            "title":"乐家" 
        }
    }
}
```

#### 文档-分词高亮

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
{
    "query":{
        "match_phrase":{
            "title":"乐家"
        }
    },
    //用<em>包裹符合title匹配条件的分词（高亮）
    "highlight":{
      "fields":{
        "title":{}
      }
    }
}
```

#### 文档-聚合操作

GET/POST请求体body携参 http://127.0.0.1:9200/{indexName}/_search

```json
//body
{
  "aggs":{
    //任意起名
    "content_group":{   
      //聚合方式 terms 分组 avg 求平均 max 求最大 min 求最小
      "avg":{  
        "field":"order" //指定聚合的字段
      }
    }
  },
  "size":0
}
```

#### 映射关系

1、创建一个新的索引user

PUT请求 http://127.0.0.1:9200/user

2、为user索引添加映射关系

PUT请求 http://127.0.0.1:9200/user/_mapping

```json
//body
{
  "properties":{
    "name":{
      "type":"text",
      "index":true
    },
    "sex":{
      "type":"keyword", //完全匹配
      "index":true
    },
    "tel":{
       "type":"keyword", //完全匹配
       "index":false //禁止被索引
    }
  }
}
```



3、给user索引插入文档

PUT请求 http://127.0.0.1:9200/user/_create/1002

```json
//body
{
  "name":"米慧",
  "sex":"女女",
  "tel":"15362900233"
}
```

4、进行条件查询

POST请求 http://127.0.0.1:9200/user/_search

```json
//body
{
  "query":{
    "match":{
      "tel":"153" 
    }
  }
}
//报错，tel不能被索引
```

```json
//body
{
  "query":{
    "match":{
      "name":"米" 
    }
  }
}
//查到数据
```

```json
//body
{
  "query":{
    "match":{
      "sex":"女" 
    }
  }
}
//查不到数据，sex为完全匹配
```


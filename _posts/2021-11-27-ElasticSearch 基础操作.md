---
layout: post
categories: ElasticSearch
tags: [ElasticSearch]
---

## IK 分词器插件

### 什么是 IK 分词器？

分词：即把一段中文或者别的划分成一个个的关键字，我们在搜索时候会把自己的信息进行分词，会把数据库中或者索引库中的数据进行分词，然后进行一个匹配操作，默认的中文分词是将每个字看成一个词，比如 “我爱狂神” 会被分为 "我","爱","狂","神"，这显然是不符合要求的，所以我们需要安装中文分词器 IK 来解决这个问题。

IK 提供了两个分词算法：ik_smart 和 ik_max_word，其中 ik_smart 为最少切分，ik_max_word 为最细粒度划分！

### 安装步骤

1、下载ik分词器的包，Github地址：[https://github.com/medcl/elasticsearch-analysis-ik/](https://github.com/medcl/elasticsearch-analysis-ik/) （版本要对应）

2、下载后解压，并将目录拷贝到 ElasticSearch 根目录下的 plugins 目录中。

![01.png](/static/images/20211127/01.png)

3、重新启动 ElasticSearch 服务，在启动过程中，你可以看到正在加载 **"analysis-ik"** 插件的提示信息，服务启动后，在命令行运行 `elasticsearch-plugin list` 命令，确认 IK 插件安装成功。

![02.png](/static/images/20211127/02.png)

4、在 kibana 中测试 ik 分词器，并就相关分词结果和 icu 分词器进行对比。

**ik_smart** : 粗粒度分词，优先匹配最长词，只有1个词！

![03.png](/static/images/20211127/03.png)

**ik_max_word** : 细粒度分词，会穷尽一个语句中所有分词可能，测试！

![04.png](/static/images/20211127/04.png)

5、我们输入超级喜欢狂神说！发现狂神说被切分了

![05.png](/static/images/20211127/05.png)

如果我们想让系统识别“狂神说”是一个词，需要编辑自定义词库。

步骤：

（1）进入 elasticsearch/plugins/ik/config 目录

（2）新建一个 my.dic 文件，编辑内容：

```
狂神说
```

（3）修改 IKAnalyzer.cfg.xml（在 ik/config 目录下）

```xml
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!-- 用户可以在这里配置自己的扩展字典 -->
    <entry key="ext_dict">my.dic</entry>
    <!-- 用户可以在这里配置自己的扩展停止词字典 -->
    <entry key="ext_stopwords"></entry>
</properties>
```

修改完配置重新启动 ElasticSearch，发现监视了我们自己写的规则文件。

再次测试，发现狂神说变成了一个词：

![06.png](/static/images/20211127/06.png)

到了这里，我们就明白了分词器的基本规则和使用了！

## Rest 风格说明

一种软件架构风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

### 基本 Rest 命令说明

|   method   |   url地址   |   描述   |
| ---- | ---- | ---- |
|   PUT   |   localhost:9200/索引名称/类型名称/文档id   |   创建文档（指定文档id）   |
|   POST   |   localhost:9200/索引名称/类型名称   |   创建文档（随机文档id）   |
|   POST   |   localhost:9200/索引名称/类型名称/文档id/_update   |   修改文档   |
|   DELETE   |   localhost:9200/索引名称/类型名称/文档id   |   删除文档   |
|   GET   |   localhost:9200/索引名称/类型名称/文档id   |   查询文档通过文档id   |
|   POST   |   localhost:9200/索引名称/类型名称/_search   |   查询所有数据   |

### 基础测试

**1、首先我们浏览器 [http://localhost:5601](http://localhost:5601) 进入 kibana 里的 Console**

**2、首先让我们在 Console 中输入：**

```json
// 命令解释
// PUT 创建命令 test1 索引 type1 类型 1 id
PUT /test1/type1/1
{
  "name":"陈先生", // 属性
  "age":18	//属性 
}
```

返回结果 （是以REST ful 风格返回的 ）：

```json
// 警告信息：不支持在文档索引请求中指定类型
// 而是使用无类型的端点(/{index}/_doc/{id}，/{index}/_doc，或/{index}/_create/{id})。
{
  "_index" : "test1",
  "_type" : "type1",
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

**3、那么 name 这个字段用不用指定类型呢。毕竟我们关系型数据库是需要指定类型的啊 !**

* 字符串类型：text 、 keyword
* 数值类型：long, integer, short, byte, double, flfloat, half_flfloat, scaled_flfloat
* 日期类型：date
* 布尔值类型：boolean
* 二进制类型：binary

**4、指定字段类型**

```json
PUT /test2
{
  "mappings":{
    "properties": {
      "name":{
        "type": "text"
      },
      "age":{
        "type": "long"
      },
      "birthday":{
        "type": "date"
      }
    }
  }
}
```

输出：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "test2"
}
```

**5、查看一下索引字段**

```
GET test2
```

输出：

```json
{
  "test2" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "birthday" : {
          "type" : "date"
        },
        "name" : {
          "type" : "text"
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test2",
        "creation_date" : "1646892173121",
        "number_of_replicas" : "1",
        "uuid" : "wwJgLLYlTuet3LOHTkEy0A",
        "version" : {
          "created" : "7140099"
        }
      }
    }
  }
}
```

**6、我们看上列中字段类型是我自己定义的，那么我们不定义类型会是什么情况呢？**

```json
PUT /test3/_doc/1
{
  "name":"陈先生",
  "age":18,
  "birthday": "2000-01-01"
}
```

输出：

```json
{
  "_index" : "test3",
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

查看一下 test3 索引：

```
GET test3
```

输出：

```json
{
  "test3" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "long"
        },
        "birthday" : {
          "type" : "date"
        },
        "name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "test3",
        "creation_date" : "1646892330279",
        "number_of_replicas" : "1",
        "uuid" : "p7psQMIySlKzssCSaqssFQ",
        "version" : {
          "created" : "7140099"
        }
      }
    }
  }
}
```

我们看上列没有给字段指定类型那么 es 就会默认给我配置字段类型！对比关系型数据库 ：

`PUT test1/type1/1` 索引 test1 相当于关系型数据库的库，类型 type1 就相当于表 ，1 代表数据中的主键 id

> 注意：在 elastisearch5 版本前，一个索引下可以创建多个类型，但是在 elastisearch5 后，一个索引只能对应一个类型，而 id 相当于关系型数据库的主键 id 若果不指定就会默认生成一个20位的 uuid，属性相当关系型数据库的 column(列)。

而结果中的 result 则是操作类型，现在是 created ，表示第一次创建。如果再次点击执行该命令那么 result 则会是 updated ，我们细心则会发现 _version 开始是1，现在你每点击一次就会增加一次。表示第几次更改。

**7、我们在来学一条命令（ElasticSearch 中的索引的情况）**

```
GET _cat/indices?v
```

返回结果：查看我们所有索引的状态健康情况 分片，数据储存大小等等。

```
#! Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.14/security-minimal-setup.html to enable security.
health status index                           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .geoip_databases                9z9ftHYYQX6pu0-Ykd_G1g   1   0         45            0     42.5mb         42.5mb
green  open   .kibana_7.14.0_001              hS6N1VFtRvemQ3a523ruuw   1   0         39           16      4.2mb          4.2mb
yellow open   test2                           wwJgLLYlTuet3LOHTkEy0A   1   1          0            0       208b           208b
yellow open   test3                           p7psQMIySlKzssCSaqssFQ   1   1          1            0      4.2kb          4.2kb
green  open   .apm-custom-link                oSApPsjsSoGgGwIjPjtCwA   1   0          0            0       208b           208b
green  open   .apm-agent-configuration        9oFK0AL4TDyZXDwmRikZOw   1   0          0            0       208b           208b
green  open   .kibana_task_manager_7.14.0_001 um37aiZOR022BQNHvg1j6w   1   0         14          645    694.3kb        694.3kb
yellow open   test1                           UjiW0XNeSyOua6OC_pjLKw   1   1          1            0      4.1kb          4.1kb
green  open   .kibana-event-log-7.14.0-000003 D3rbt3ZKRfKlwcRgzEotkg   1   0          4            0     16.7kb         16.7kb
green  open   .kibana-event-log-7.14.0-000002 UpX40C60RaCmz9-XTHyxpQ   1   0          0            0       208b           208b
green  open   .tasks                          OR0PVv0VTn2_UyOpWJCxTQ   1   0         10            0       44kb           44kb
```

**8、那么怎么删除一条索引（库）呢?**

```
DELETE /test1
```

返回：

```json
{
  "acknowledged" : true // 表示删除成功
}
```

## 增删改查命令

### 创建数据 PUT

```json
# 第一条数据
PUT /project/user/1
{
  "name":"张三",
  "age":3,
  "desc":"法外狂徒",
  "tags":["直男","技术宅","温暖"]
}

# 第二条数据
PUT /project/user/2
{
  "name":"李四",
  "age":4,
  "desc":"四哥",
  "tags":["渣男","旅游","交友"]
}

# 第三条数据
PUT /project/user/3
{
  "name":"王五",
  "age":5,
  "desc":"五妹",
  "tags":["靓女","旅游","唱歌"]
}
```

查看数据：

![07.png](/static/images/20211127/07.png)

注意：当执行命令时，如果数据不存在，则新增该条数据，如果数据存在则修改该条数据。

通过 GET 命令查询一下：

```
GET project/user/1
```

返回结果：

```json
{
  "_index" : "project",
  "_type" : "user",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "张三",
    "age" : 3,
    "desc" : "法外狂徒",
    "tags" : [
      "直男",
      "技术宅",
      "温暖"
    ]
  }
}
```

如果你想更新数据可以覆盖这条数据：

```json
PUT /project/user/1
{
  "name":"张三666",
  "age":3,
  "desc":"法外狂徒",
  "tags":["直男","技术宅","温暖"]
}
```

返回结果：

```json
{
  "_index" : "project",
  "_type" : "user",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

再次通过 GET 命令查询一下：

```
GET project/user/1
```

返回结果：

```json
{
  "_index" : "project",
  "_type" : "user",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 3,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "张三666",
    "age" : 3,
    "desc" : "法外狂徒",
    "tags" : [
      "直男",
      "技术宅",
      "温暖"
    ]
  }
}
```

已经修改了，那么 **PUT** 可以更新数据。但是**麻烦的是原数据你还要重写一遍，这不符合我们规矩**。

### 更新数据 POST

我们使用 POST 命令，要修改的内容放到 doc 文档（属性）中即可。

```json
POST /project/user/1
{
  "name":"张三777",
  "age":18
}
```

返回结果：

```json
{
  "_index" : "project",
  "_type" : "user",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

### 条件查询

简单的查询，我们上面已经不知不觉的使用熟悉了：

```
GET project/user/1
```

我们来学习下条件查询 _search?q=

```
GET /project/user/_search?q=name:张三
```

通过 `_search?q=name:张三` 查询条件是 name 属性有 `张三` 的那些数据。

**别忘了 _search 和 from 属性中间的分隔符 ?** 

返回结果：

```json
{
  "took" : 1009,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.71844894,
    "hits" : [
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "1",
        "_score" : 0.71844894,
        "_source" : {
          "name" : "张三777",
          "age" : 18
        }
      }
    ]
  }
}
```

我们看一下结果返回并不是数据本身，是给我们了一个 hits ，还有 _score 得分，就是根据算法算出和查询条件匹配度高得分就高。

### 构建查询

```json
GET /project/user/_search
{
  "query":{
    "match": {
      "name": "张三"
    }
  }
}
```

上例，查询条件是一步步构建出来的，将查询条件添加到 match 中即可。返回结果还是一样的：

```json
{
  "took" : 1009,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.71844894,
    "hits" : [
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "1",
        "_score" : 0.71844894,
        "_source" : {
          "name" : "张三777",
          "age" : 18
        }
      }
    ]
  }
}
```

除此之外，我们还可以查询全部：

```json
# 这是一个查询但是没有条件
GET /project/user/_search
{
  "query":{
    "match_all": {}
  }
}
```

match_all 的值为空，表示没有查询条件，就像 select * from table_name 一样。

返回结果：全部查询出来了！

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "李四",
          "age" : 4,
          "desc" : "四哥",
          "tags" : [
            "渣男",
            "旅游",
            "交友"
          ]
        }
      },
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "王五",
          "age" : 5,
          "desc" : "五妹",
          "tags" : [
            "靓女",
            "旅游",
            "唱歌"
          ]
        }
      },
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "张三777",
          "age" : 18
        }
      }
    ]
  }
}
```

如果有个需求，我们仅是需要查看 name 和 desc 两个属性，其他的不要怎么办？

```json
GET /project/user/_search
{
  "query":{
    "match_all": {}
  },
  "_source": ["name", "desc"]
}
```

如上例所示，在查询中，通过 _source 来控制仅返回 name 和 age 属性。

```json
{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "李四",
          "desc" : "四哥"
        }
      },
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "name" : "王五",
          "desc" : "五妹"
        }
      },
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "张三777"
        }
      }
    ]
  }
}
```

**一般的，我们推荐使用构建查询，以后在与程序交互时的查询等也是使用构建查询方式处理查询条件，因为该方式可以构建更加复杂的查询条件，也更加一目了然。**

### 排序查询

一说到排序，有人就会想到：正序 或 倒序 那么我们先来倒序：

```json
GET /project/user/_search
{
  "query":{
    "match_all": {}
  },
  "sort": [
    {
      "age":{
        "order": "desc"
      }
    }
  ]
}
```

正序，就是 desc 换成了 asc

注意：在排序的过程中，只能使用可排序的属性进行排序。那么可以排序的属性有哪些呢？

* 数字
* 日期
* ID

其他都不行！

### 分页查询

```
GET /project/user/_search
{
  "query":{
    "match_all": {}
  },
  "sort": [
    {
      "age":{
        "order": "desc"
      }
    }
  ],
  "from": 0, # 从第n条开始
  "size": 1 # 返回n条数据
}
```

### 布尔查询

**must (and)**

```json
# 查询条件为：name=李四 && age=4
GET /project/user/_search
{
  "query":{
    "bool": {
      "must": [
        {
          "match": {
            "name": "李四"
          }
        },
        {
          "match": {
            "age": 4
          }
        }
      ]
    }
  }
}
```

**should (or)**

```json
# 查询条件为：name=李四 || age=5
GET /project/user/_search
{
  "query":{
    "bool": {
      "should": [
        {
          "match": {
            "name": "李四"
          }
        },
        {
          "match": {
            "age": 5
          }
        }
      ]
    }
  }
}
```

**must_not (not)**

```json
# 查询条件为：name!=李四 && age!=5
GET /project/user/_search
{
  "query":{
    "bool": {
      "must_not": [
        {
          "match": {
            "name": "李四"
          }
        },
        {
          "match": {
            "age": 5
          }
        }
      ]
    }
  }
}
```

**fitter**

* gt 表示大于
* gte 表示大于等于
* lt 表示小于
* lte 表示小于等于

```json
# 查询条件为：age>4
GET /project/user/_search
{
  "query":{
    "bool": {
      "filter": [
        {
          "range": {
            "age": {
              "gt": 4
            }
          }
        }
      ]
    }
  }
}
```

### 短语检索

我要查询 tags 包含 ”男“ 的数据

```
GET /project/user/_search
{
  "query":{
    "match": {
      "tags": "男"
    }
  }
}
```

返回了所有标签中带 男 的记录！

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 0.48034602,
    "hits" : [
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "2",
        "_score" : 0.48034602,
        "_source" : {
          "name" : "李四",
          "age" : 4,
          "desc" : "四哥",
          "tags" : [
            "渣男",
            "旅游",
            "交友"
          ]
        }
      }
    ]
  }
}
```

既然按照标签检索，那么能不能写多个标签呢？又该怎么写呢？

```json
GET /project/user/_search
{
  "query":{
    "match": {
      "tags": "男 唱歌"
    }
  }
}
```

返回：只要含有这个标签满足一个就给我返回这个数据了。

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 2.0048246,
    "hits" : [
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "3",
        "_score" : 2.0048246,
        "_source" : {
          "name" : "王五",
          "age" : 5,
          "desc" : "五妹",
          "tags" : [
            "靓女",
            "旅游",
            "唱歌"
          ]
        }
      },
      {
        "_index" : "project",
        "_type" : "user",
        "_id" : "2",
        "_score" : 0.48034602,
        "_source" : {
          "name" : "李四",
          "age" : 4,
          "desc" : "四哥",
          "tags" : [
            "渣男",
            "旅游",
            "交友"
          ]
        }
      }
    ]
  }
}
```

### 精确查询

term 查询是直接通过倒排索引指定的词条，也就是精确查找。

term 和match 的区别：

* match 是经过分析（analyer）的，也就是说，文档是先被分析器处理了，根据不同的分析器，分析出的结果也会不同，在会根据分词结果进行匹配。
* term 是不经过分词的，直接去倒排索引查找精确的值。

注意：我们现在用的 es7 版本，所以我们用 mappings properties 去给多个字段（fields）指定类型的时候，不能给我们的索引制定类型：

```json
PUT /testdb
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "desc": {
        "type": "keyword"
      }
    }
  }
}

# 插入数据
PUT /testdb/_doc/1
{
  "name": "张三333",
  "desc": "张三333"
}

PUT /testdb/_doc/2
{
  "name": "李四444",
  "desc": "李四444"
}
```

上述中 testdb 索引中，字段 name 在被查询时会被分析器进行分析后匹配查询。而属于 keyword 类型不会被分析器处理。

我们来验证一下：

```json
GET _analyze
{
  "analyzer": "keyword",
  "text": "张三333"
}
```

结果：

```json
{
  "tokens" : [
    {
      "token" : "张三333",
      "start_offset" : 0,
      "end_offset" : 5,
      "type" : "word",
      "position" : 0
    }
  ]
}
```

是不是没有被分析啊。就是简单的一个字符串啊。再测试

```json
GET _analyze
{
  "analyzer": "standard",
  "text": "张三333"
}
```

结果：

```json
{
  "tokens" : [
    {
      "token" : "张",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "三",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "333",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "<NUM>",
      "position" : 2
    }
  ]
}
```

那么我们看一下字符串是不是被分析了啊。**总结：keyword 字段类型不会被分析器分析！**

现在我们来查询一下：

```json
// text 会被分析器分析查询
GET /testdb/_search
{
  "query": {
    "term": {
      "name": {
        "value": "张"
      }
    }
  }
}

// keyword 不会被分析所以直接查询
GET /testdb/_search
{
  "query": {
    "match": {
      "desc": "张三333"
    }
  }
}
```

### 高亮显示

```json
GET /testdb/_search
{
  "query": {
    "match": {
      "name": "张三333"
    }
  },
  "highlight": {
    "fields": {
      "name": {}
    }
  }
}
```

返回结果：

```json
{
  "took" : 54,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 2.0794413,
    "hits" : [
      {
        "_index" : "testdb",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 2.0794413,
        "_source" : {
          "name" : "张三333",
          "desc" : "张三333"
        },
        "highlight" : {
          "name" : [
            "<em>张</em><em>三</em><em>333</em>"
          ]
        }
      }
    ]
  }
}
```

我们可以看到已经帮我们加上了一个 `<em>` 标签

这是 es 帮我们加的标签。那我也可以自己自定义样式。

```json
GET /testdb/_search
{
  "query": {
    "match": {
      "name": "张三333"
    }
  },
  "highlight": {
    "pre_tags": "<b class='key' style='color:red'>",
    "post_tags": "</b>",
    "fields": {
      "name": {}
    }
  }
}
```

返回结果：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 2.0794413,
    "hits" : [
      {
        "_index" : "testdb",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 2.0794413,
        "_source" : {
          "name" : "张三333",
          "desc" : "张三333"
        },
        "highlight" : {
          "name" : [
            "<b class='key' style='color:red'>张</b><b class='key' style='color:red'>三</b><b class='key' style='color:red'>333</b>"
          ]
        }
      }
    ]
  }
}
```

需要注意的是：自定义标签中属性或样式中的逗号一律用英文状态的单引号表示，应该与外部 es 语法 的双引号区分开。

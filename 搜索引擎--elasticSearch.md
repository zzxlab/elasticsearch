## 搜索引擎--elasticSearch

> ​	ElasticSearch可以快速的储存、搜索、分析海量的数据。es的底层是开源库Lucene，es是Lucene的封装，提供了rest api的操作接口，开箱即用。



### 安装

#### 单节点

centos下安装

1. 新增一个用户并切换

   ```sh
   adduser es
   su es
   ```
   
2. 将下载好的安装包解压

   ```
   tar -zxvfelasticsearch-6.6.0.tar.gz
   ```

3. 修改elasticsearch配置文件elasticsearch.yml

   ```
   vim elasticsearch.yml
   添加
   network.host: 0.0.0.0
   http.port: 9200
   ```

4. 后台启动

   ```
   ./bin/elasticsearch -d
   ```

5. 验证是否启动成功

   在浏览器输入网址 http://ip:9200，能进入且看到json格式的elasticsearch信息

kibana安装

解压文件

```
tar -zxvf kibana-6.6.0-linux-x86_64.tar.gz
```

修改配置文件

```shell
vim kibana.yml
server.host: "0.0.0.0"
# elasticsearch的访问地址
elasticsearch.hosts: ["http://172.16.142.141:9200"]
```

启动kibana

```shell
# 前台启动
./bin/kibana

# 后台启动
nohup ./bin/kibana >/dev/null 2>&1 &

#查看是否启动成功
netstat -anltp | grep 5601
```



#### 集群

### es中的概念

- node和cluster：es本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个es实例；单个es实例被称为一个节点（node），一组节点构成一个集群（cluster）。

- 索引（Index）：类似于关系型数据库中database；es会索引所有字段，经过处理后写入一个反向索引（inverted index），查找数据的时候会直接查找该索引，所以es数据管理的顶层单位叫做index（索引）。它是单个数据库的同义词。每个index（即数据库）的名字必须是小写英文。
- 表（type）：类似于关系型数据库中table（表的概念），es在index（索引）中建立type，通过mapping进行映射；.......
- 文档（ **document** ）：es存储的数据类型为文档类型，一条数据对应一篇文档，即相当于关系型数据库中一行数据，field：es中的一篇文档中对应多个与MySQL数据库中每一列对应；
- mapping：对应MySQL数据库中schema，则es中有动态识别功能；
- indexed：名义上建立索引（该索引的含义是相当于关系型数据库中给某一列加上索引），在关系型数据库中加索引是为了提高查询性能。同样的在es中也是为了提高性能，在es中默认都是加了索引，若不加索引则需要自己设定；
- Query DSL：类似于MySQL中的sql语句，但不是SQL语句，在es中是个json格式的查询语句；

1. elasticSearch中通用属性
   - index_name：该属性是存储在索引中字段名称，若未指定，默认为字段定义的对象的名称，可忽略；
   - index：该属性可取值analyzed或no，默认值为analyzed。当字段的类型为字符串型可以设为not_analyzed，便是该字段不可被搜索，只有完全匹配才能查到该字段。
   - store：该属性的取值为yes或no，用于指定字段的原始值是否存入索引，默认值为no，表示不能在结果中返回字段的原始值（即使没有存储原始值，也可以使_source字段返回原始值），若果已建立索引，可以搜索该字段的内容。
   - boost：该属性的默认值为1，用于定义该字段在文档中的重要性,值越大，越重要。
   - null_value：该属性表示某字段在被索引写入数据时，默认可以忽略该字段。
   - include_in_all：该属性表示指定的字段是否应被包含到_all字段中。默认情况下，启用该字段为包含所有的字段。

2. elasticSearch中核心类型

```json
#模式映射用于定义索引结构
#下面为定义的索引结构
{
    "mappings": {
        "post": {
            "properties": {
                "id": {"type":"long","store":"yes","precision_step":"0"},
                "name": {"type":"string","store":"yes","index":"analyzed"},
                "published": {"type":"date","store":"yes",
                              "precision_step":"0"},
                "contents": {"type":"string","store":"no","index":"analyzed"}
            }
        },
        "user": {
            "properties": {
                "id": {"type":"long","store":"yes","precision_step":"0"},
                "name": {"type":"string","store":"yes","index":"analyzed"}
            }
        }
    }
}
```

核心类型：字符串型（string）、数值型（number）、日期型（date）、布尔类型（boolean）、二进制型（binary）

- 字符串型（string）字段下还可以设置的属性
  - term_vector：该属性表示是否对字段计算Lucene词向量，该属性的取值可以为no（默认值）、yes、with_offsets、with_positions、with_positions_offsets。是用高亮需要计算词向量。
  - omit_norms：该属性表示是否进行Lucene norms计算，默认值为false，默认时指可以进行Lucene norms计算
  - omit_term_freq_and_positions：该属性表示建索引时是否忽略词频和位置计算。默认值为false，若要忽略词频和位置计算，设置为true。（0.20开始弃用）
  - index_options：该属性用于设置索引选项，可取值有docs（索引的文档数量）、freqs（索引的文档数量和词频）、positions（索引的文档数量、词频和单词出现的位置）。默认为freqs。(从0.20版本开始使用)
  - ignore_above：该属性表示字段的最大长度，若超出，则忽略。
  - analyzer：该属性用于索引和搜索的分析器的名称，默认是全局定义的分析器。
  - index_analyzer：索引名称分析器
  - search_analyzer：用于处理作用在该字段的查询的分析器的名称
- 数值型（number）代表了所有的数值类的字段类型，包括：字节型、短整型、整型、长整型、单精度浮点型、双精度浮点型，该类型下还存在的属性有：
  - precision_step：该属性设置字段的每个取值生成的项数。值越低生成的项数越多，进行range查询时越快（但索引也会变大），默认值为4。
  - ignore_malformed：该属性的取值可为true或false，默认false。该字段表示为是否忽略错误的数值，若忽略，应设为true。
- 日期型（date），该类型下存在的其他属性
  - format：该属性用于指定日期格式。默认值dateOptionalTime
  - precision_step：该属性设置字段的每个取值生成的项数。值越低生成的项数越多，进行range查询时越快（但索引也会变大），默认值为4。
  - ignore_malformed：该属性的取值可为true或false，默认false。该字段表示为是否忽略错误的数值，若忽略，应设为true。
- 二进制型：二进制字段是指用base64来表示索引中存储的二进制数据，可用来存储二进制形式的数据，默认情况下，该类型字段只存储不索引。

在elasticSearch中，rest api数据操作

- get：获得所请求的对象当前状，用来获取资源；
- post：改变当前对象，用来新建资源，也可以用来更新资源；
- put：修改对象；更新资源
- delete：销毁对象，删除资源
- head：仅用于提取对象的基本信息。

### ElasticSearch SQL使用方式

> ​	**es支持三种client：rest interface、command-line、JDBC**

#### es中数据类型

##### es sql与关系型数据库sql数据类型的对应关系

| elasticSearch type | sql type  | sql precision        |
| ------------------ | --------- | -------------------- |
| null               | null      | 0                    |
| boolean            | boolean   | 1                    |
| byte               | tinyint   | 3                    |
| short              | smallint  | 5                    |
| interger           | interger  | 10                   |
| long               | long      | 19                   |
| double             | double    | 15                   |
| float              | real      | 7                    |
| half_float         | float     | 16                   |
| scaled_float       | float     | 19                   |
| keyword            | varchar   | base on ignore_above |
| text               | varchar   | 2147483647           |
| binary             | varbinary | 2147483647           |
| date               | timestamp | 24                   |
| object             | struct    | 0                    |
| nested             | struct    | 0                    |

#### 常用SQL语句

##### 查询语句

| 语句                                                         | 含义                                                         | 例子                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| show tables;                                                 | 查看当前用户所有索引                                         |                                                              |
| show tables like "索引名称"                                  | 查询某个索引信息；注意：查询单个索引名一定要用""引上         |                                                              |
| show tables like '索引名称（通配符:_、%、*）'                | 使用通配符模糊查询某些索引                                   |                                                              |
| describe table;\desc table                                   | 查看某个索引结构                                             |                                                              |
| show columns [from \| in] table                              | 查看表内字段结构                                             |                                                              |
| show functions [like '函数名']                               | 查看所有函数或查看某一函数                                   |                                                              |
| select 要查的列名或聚合函数 [from table_name] [where codition] [group by grouping_element] [having condition] [order by 字段名 [asc \| desc] ] [limit [count]] | select 查询语句的语法排序；注：在rest api json写法中limit的优先级高于fetch_size；排序可以写多个字段进行排序；having 过滤分组结果 | {   "query": "SELECT * FROM "micloud_es_sink_zzx_test” limit 5",   "fetch_size":10  } |
| select * from a表 where b表.b表的字段='xxx'                  | 嵌套查询                                                     | select * from zzx_test where zzx_demo.user='张三';           |
| select * from '部分表名+通配符'                              | 使用用通配符查询多个索引                                     | select * from 'zzx*';                                        |

##### 比较操作

等于（=）、不等于（<>、!=、<=>）、比较（<、<=、>、>=）、between...and... 、is null、is not null

##### 逻辑操作

and 、or、not

```sql
select * from "zzx_demo" where value_1 > 5 and value_1 < 7 limit 5;
select * from "zzx_demo" where value_1 = 5 or value_1 = 7 limit 5;
select * from "zzx_demo" where not value_1 > 5 limit 5;
```

##### 数字运算操作

| 运算操作符 | 例子                 |
| ---------- | -------------------- |
| +          | select  1 + 1 as x;  |
| -          | select  1 - 1 as x;  |
| -          | select  - 1 as x;    |
| *          | select  6 * 6 as x;  |
| /          | select  30 / 5 as x; |
| %          | select  30 % 7 as x; |

##### 数学函数

###### 通用函数

| 函数      | 含义                                     | 例子                                                         |
| --------- | ---------------------------------------- | ------------------------------------------------------------ |
| CBRT()    | 求数字的立方根，返回double类型           | select  value_1,CBRT(value_1)  from  "zzx_demo"  limit  5;   |
| CEIL()    | 返回大于或者等于指定的最小整数（double） | select  value_1,CEIL(value_1)  from  "zzx_demo"  limit  5;   |
| CEILING() | 相当于CEIL()函数                         | select  value_1,CEILING(value_1)  from  "zzx_demo"  limit  5; |
| E()       | 返回自然常数e                            | select  value_1,E(value_1)  from  "zzx_demo"  limit  5;      |
| ROUND()   | 四舍五入                                 | select ROUND(-3.14);                                         |
| FLOOR()   | 向下取整                                 | select FLOOR(3.14);                                          |
| LOG()     | 计算以2为底的对数                        | select LOG(4);                                               |
| LOG10()   | 计算以10为底的对数                       | select LOG10(100);                                           |
| SQRT()    | 求一个非负数的平方根                     | select  SQRT(9);                                             |
| EXP()     | 返回e的x次方的值                         | select  EXPM1(3);                                            |
| EXPM1()   | 返回ex-1的值                             | select  EXPM1(3);                                            |
| ABS()     | 求数字绝对值                             | select  value_1,ABS(value_1)  from  "zzx_demo"  limit  5;    |

###### 三角函数

| 函数      | 含义                                              | 例子               |
| --------- | ------------------------------------------------- | ------------------ |
| DEGREES() | 返回X从弧度转换为度值                             | select DEGREES(x); |
| RADIANS() | 返回X从度转换成弧度的值                           | select RADIANS(x); |
| SIN()     | 返回X的正弦                                       | select SIN(x);     |
| COS()     | 返回X，X值是以弧度给出的余弦值                    | select COS(角度);  |
| TAN()     | 返回参数X，表示以弧度的切线值                     | select TAN(角度);  |
| ASIN()    | 返回X的反正弦，X的值必须在-1至1范围内，返回NULL   | select ASIN(x);    |
| ACOS()    | 返回X的反正弦，X值必须-1到1之间范围否则将返回NULL | select ACOS(x);    |
| ATAN()    | 返回x的反切值                                     | select ATAN(x);    |
| SINH      | 返回x的双曲正弦值                                 | select SINH(x);    |
| COSH()    | 返回x的双曲余弦值                                 | select COSH(x);    |

##### 日期和时间的处理方法

| 函数                          | 含义 | 例子                                                         |
| ----------------------------- | ---- | ------------------------------------------------------------ |
| YEAR()                        | 年   | SELECT YEAR(CAST('2020-02-02T16:59:27Z' AS TIMESTAMP)) AS year; |
| MONTH_OF_YEAR()  MONTH()      | 月   | SELECT MONTH(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS month; |
| WEEK_OF_YEAR()  WEEK()        | 周   | SELECT WEEK(CAST('2020--02-02T16:59:27ZZ' AS TIMESTAMP)) AS week; |
| DAY_OF_YEAR()   DOY()         | 天   | SELECT DOY(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS day; |
| DAY_OF_MONTH()  DOM()  DAY()  | 天   | SELECT DAY(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS day; |
| DAY_OF_WEEK()   DOW()         | 天   | SELECT DOW(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS day; |
| HOUR_OF_DAY()   HOUR()        | 小时 | SELECT HOUR(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS hour; |
| MINUTE_OF_DAY()               | 分钟 | SELECT MINUTE_OF_DAY(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS minute; |
| MINUTE_OF_HOUR()   MINUTE()   | 分钟 | SELECT MINUTE(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS minute; |
| SECOND_OF_MINUTE()   SECOND() | 秒   | SELECT SECOND(CAST('2020--02-02T16:59:27Z' AS TIMESTAMP)) AS second; |

#### Rest Api

常用操作：

- Get：获取资源，用于查询操作
- Post：新建资源或更新资源，相当于insert、update操作
- Put：更新资源，相当于update操作
- Delete：删除资源，相当于delete操作

以上四种命令的操作可以在kibana提供的dev tools工具上输入命令进行对数据的操作

##### 创建索引

使用put或者post

```json
POST /zzx 
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replices": 1
  },
  "mappings": {
    "user": {
      "properties": {
        "name": {
          "type": "text",
          "analyzer": "ik_max_word"
        },
        "address": {
          "type": "text",
          "analyzer": "ik_max_word"
        },
        "age": {
          "type": "integer"
        },
        "interests": {
          "type": "text",
          "analyzer": "ik_max_word"
        },
        "birthday": {
          "type": "date"
        }
      }
    }
  }
}
或者
PUT /product_info
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  },
  "mappings": {
    "products": {
      "properties": {
      "productName": {"type": "text","analyzer":"ik_smart"},
      "annual_rate":{"type":"keyword"},
      "describe":{"type":"text","analyzer":"ik_smart"}
      }
    }
  }
}
```

##### 添加数据

###### 单条添加

```json
PUT /zzx/user/1
{
  "name": "赵六",
  "address": "黑龙江省铁岭",
  "age": 50,
  "birthday": "1980-11-01",
  "interests": "喜欢喝酒，锻炼，说相声"
}

PUT /zzx/user/2
{
  "name": "赵明",
  "address": "北京海淀区清河",
  "age": 20,
  "birthday": "2000-11-01",
  "interests": "喜欢喝酒，锻炼，唱歌"
}
PUT /zzx/user/3
{
  "name": "lisi",
  "address": "北京海淀区清河",
  "age": 23,
  "birthday": "1997-12-01",
  "interests": "喜欢喝酒，锻炼，唱歌"
}
```

###### 多条添加（_bulk）

##### term查询和terms查询

term query会去倒排索引中寻找确切的term,它并不知道粉刺的存在，这种查询适合keyword、numeric、date;

term：查询某个字段里含有某个关键词的文档

```json
get /zzx/demo/_search 
{
	"query": {
		"term": {
			"interests": "changge"
		} 
	}
}
```

terms：查询某个字段里含有多个关键词的文档

```json
get /zzx/demo/_search 
{
	"query": {
		"terms": {
			"interests": ["changge","paobu"]
		} 
	}
}
```

##### 基本查询（query查询）

1. ik分词器

   ik分词器带有两个分词器：ik_max_word和ik_smart

   ik_max_word：会将文本做最细粒度的拆分，尽可能多的拆分出词语；

   ik_smart：会做最粗粒度的拆分，已被分出的词语将不会再次被其他词语占有；

2. 

###### match查询

match query知道分词器的存在，会对field进行分词操作，然后再查询

```json
get /zzx/user/_search
{
	"query": {
		"match": {
			"name": "zzx"
		} 
	}
}
```

- match_all：查询所有文档

  ```json
  get /zzx/user/_search
  {
  	"query": {
  		"match_all": {
  			"name": {}
  		} 
  	}
  }
  ```

- multi_match：可以指定多个字段

  ```json
  get /zzx/user/_search
  {
  	"query": {
  		"multi_match": {
  			"query": "you"
  			"fields": ["interests","name"]
  		} 
  	}
  }
  ```

- match_phrase：短语匹配查询

  ```json
  get /zzx/user/_search
  {
  	"query": {
  		"match_phrase": {
  			"interests": "changge,hejiu"
  		} 
  	}
  }
  ```

###### 高亮查询

高亮显示使用highlight

```json
GET /product_info/_search
{
  "query": {
    "match": {
      "describe": "最低起投"
    }
  },
  "highlight": {
    "fields": {"describe": {}}
  }
}
```



###### fuzzy 实现模糊查询

- value：查询的关键字

- boost：查询的权值，默认值是1.0

- min_similarity：设置匹配的最小相似度，默认值为0.5，对于字符串取值为0-1（包括0和1）；对于数值，取值可能大于1；对于日期型取值为1d，1m等（1d代表一天）；

- prefix_length：指明区分词项的共同前缀长度，默认为0；

- max_expansions：查询中的词项可以扩展的数目，默认为无限大

  ```json
  get /zzx/demo/_search
  {
  	"query": {
  		"fuzzy": {
  			"interests": "changge"
  		}
  	}
  }
  get /zzx/demo/_search
  {
  	"query": {
  		"fuzzy": {
  			"interests": {
                  "value": "changge"
              }
  		}
  	}
  }
  ```

###### 排序

- 使用sort实现排序：desc降序，asc升序

  ```json
  get /zzx/user/_search
  {
  	"query": {
  		"match_all": {}
  	},
  	"sort": [{
  		"age": {
  			"order": "desc"
  		}
  	}]
  }
  ```
  
- 对于text类型的字符串排序的问题，不能这样直接使用sort，因为text类型的内容，elasticsearch会进行分词，要使text类型字段进行排序，必须要进行索引两次，一次索引分词（用于搜索），一次索引不分词（用于排序）

  ```
  
  ```

  

###### 前缀匹配查询

```json
get /zzx/user/_search
{
	"query": {
		"match_phrase_prefix": {
			"name": {
				"query": "zhang"
			}
		}
	}
}
```

###### 范围查询

- rang：实现范围查询
- 参数： from，to，include_lower，include_upper，boost
  - include_lower：是否包含范围的左边界，默认为true
  - include_uppper：是否包含范围的有边界，默认为true

##### filter查询

> filter查询是不计算相关性的，同是可以cache。因此，filter速度要比query查询快

###### 简单的过滤查询

```json
GET /zzx/_search
{
  "post_filter": {
    "term": {
      "interests": "喝酒"
    }
  }
}

GET /zzx/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "interests": "喝酒"
        }
      }
    }
  }
}
```

###### bool过滤查询

可以实现组合过滤查询

格式：

```json
{
    "bool":{
        "must":[],
        "should":[],
        "must_not":[]
    }
}
```

must：必须满足的条件--and

should：可以满足的也可以不满足的条件--or

must_not：不需要满足的条件--not

```json
GET /zzx/_search
{
  "query": {
    "bool": {
      "must_not": [
        {"term": {
          "interests": {
            "value": "喝酒"
          }
        }}
      ]
    }
  }
}
```

###### 范围过滤

- gt：>

- lt：<
- gte：>=
- lte：<=

###### 过滤非空

```json
GET /zzx/_search
{
    "query":{
        "bool":{
            "filter":{
                "exists":{
                    "interests":"唱歌"
                }
            }
        }
    }
}
```

##### 聚合查询

sum  min max  avg  cardinalitty求基数  terms分组

 ```json
#求和
GET /zzx/_search
{
  "size": 0,
  "aggs": {
    "age_sum": {
      "sum": {
        "field": "age"
      }
    }
  }
}
#求最小值
GET /zzx/_search
{
  "size": 0,
  "aggs": {
    "age_min": {
      "min": {
        "field": "age"
      }
    }
  }
}
#求最大值
GET /zzx/_search
{
  "size": 0,
  "aggs": {
    "age_max": {
      "max": {
        "field": "age"
      }
    }
  }
}
#求基数
GET /zzx/_search
{
  "size": 0,
  "aggs": {
    "age_cardi": {
      "cardinality": {
        "field": "age"
      }
    }
  }
}

#分组
GET /zzx/_search
{
  "size": 0,
  "aggs": {
    "age_group": {
      "terms": {
        "field": "age"
      }
    }
  }
}
#例子
GET /zzx/_search
{
  "_source": ["name","age"], 
  "query": {
    "match": {
      "interests": "唱歌"
    }
  },
  "aggs": {
    "age_cardi": {
      "cardinality": {
        "field": "age"
      }
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    }
  ]
}
 ```

##### 复合查询

接收以下参数

- must：文档必须匹配这些条件才能被包含进来；
- must_not：文档必须不匹配这些条件才能被包含进来；
- should：如果满足这些语句中的任意条件，将增加_score,否则无任何影响。他们主要用于修正每个文档的相关性得分。
- filter：必须匹配，但它以不评分、过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除包括文档。

相关性得分的组合：每一个子查询都独自的计算文档的相关性得分，一旦它们的得分被计算出来，bool查询就将这些得分进行合并并且返回一个代表整个布尔操作的得分。

```json
GET /ZZX/_Search
{
    "query":{
        "bool":{
            "must":{"match":{"interests":"唱歌"}},
            "must_not":{"match":{"interests":"你好"}},
            "should":[
                {"match":{"address":"北京"}},
                {"rang":{"birthday":{"gte":"1996-09-01"}}}
            ]
        }
    }
}
```

###### constant_score查询

它将一个不变的常量评分应用于所有匹配的文档。它经常被用于你只需要执行一个filter而没有其他查询（例如：评分查询）

```json
GET /zzx/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "interests": "唱歌"
        }
      }
    }
  }
}

```

##### 分页查询中的deep paging问题

```json
#使用from和size就行分页查询，from指定从那个文档开始，size指定分页大小
GET /ZZX/_search?from=1&size=3
```



- deep paging：查询的很深，比如一个索引有三个primary shard，分别存储了6000条数据，我们要得到第100页（每页10条），类似于这种状况被称为deep paging；
- 分页查询数据的正确做法，每个shard把0-999条数据都查询出来并排序，然后全部返回给coordinate node按照_score分数排序后取出第100页的10条数据，返回给客户端；

deep paging对性能的影响：

- 耗费网络带宽，因为搜索过深，各个shard要把数据传送给coordinate node，这个过程存在大量的数据传输，消耗网络；
- 消耗内存，各个shard要把数据传送给coordinate node，这个传递回来的数据是被coordinate node保存在内存中，会消耗大量的内存；
- 消耗CPU，coordinate node要把传送回来的数据进行排序，这个排序过程很消耗CPU

#### command-cli

安装elasticSearch是提供了命令行界面，打开方式

```
./elasticsearch-sql-cli ip:port
```

类似于MySQL命令行模式

#### JDBC

#### Java API

##### spring boot 集成elasticsearch

###### maven依赖引入

```xml
   
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
  </parent>   

<dependency>  
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

###### application.yml配置

```yaml
spring:
  elasticsearch:
    rest:
      username: elastic
      uris: http://192.168.183.129:9200
      password: elastic
```




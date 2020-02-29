## 搜索引擎--elasticSearch

> ​	ElasticSearch可以快速的储存、搜索、分析海量的数据。es的底层是开源库Lucene，es是Lucene的封装，提供了rest api的操作接口，开箱即用。



### 安装

#### 单节点

centos下安装

1. 新增一个用户并切换

   ```sh
   adduser es
   passd es
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

#### command-cli

安装elasticSearch是提供了命令行界面，打开方式

```
./elasticsearch-sql-cli ip:port
```

类似于MySQL命令行模式

#### JDBC
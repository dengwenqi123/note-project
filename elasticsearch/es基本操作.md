### 集群的相关概念

##### 集群cluster

一个集群就是由一个或多个节点组织在一起，他们共同持有整个的数据，并一起提供索引和搜索功能。一个集群由一个唯一的名字标识，这个名字就是"elasticsearch".这个名字很重要，因为一个节点只能通过指定某个集群的名字，来加入这个集群

##### 节点node

##### 分片和副本

#### ES基本查询操作

ElasticSearch 是一个分布式可扩展的实时搜索和分析引擎。ES不仅支持全文搜索，还支持结构化搜索、统计、查询过滤、地理位置、自动完成等。

```javascript
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
	"query": {
		"match_all":{}
	}
}
'
1.相应的 HTTP 请求方法或者变量：GET POST PUT HEAD 或者DELETE.
2.集群中任意一个节点的访问协议、主机名以及端口。
3.请求的路径。
4.一个JSON 编码的请求主体.
```

检索文档

```java
GET /misp_data_log/log/1
```

**批量检索**检索使用_search命令`GET /misp_data_log/_search`,默认会返回最前的10条数据

**指定条件检索**检索指定条件结果(给_search传递参数，就和网页搜索参数一样)

```java
GET /misp_log_data/_search?q=last_name:Smith
```

上面还可以通过Query DSL(Domain Specific Language 领域特定语言)来进行搜索达到相同效果

```java
GET /misp_data_log/_search
{
	"query":{
		"match":{
			"last_name":"Smith"
		}
	}
}
```

**结果过滤** 在搜索中加入filter,对结果过滤 检索姓Smith,并且年龄大于30岁的员工

```
GET /misp_data_log/_search
{
	"query":{
		"filter":{
			"range":{
				"age": {"gt": 30}
			}
		},
		"query":{
			"match":{
				"last_name":"Smith"
			}
		}
	}
}
```

全文匹配**match** 检索about字段中有"rock climbing" 的记录

```java
GET /misp_data_log/_search
{
	"query":{
		"match":{
			"about":"rock climbing"
		}
	}
}
```

上述命令会把about字段中带有"rock climbing" 或者带有 "rock" "climbing" 的记录检索出来。当然会给结果做一个评分，都包含的结果会再更前面。

**精准匹配match_phrase** 如果上面的实例，你不想要只包含“rock”这种部分匹配的结果，那么你可以使用match_phrase来进行精确匹配。

```
GET /misp_data_log/_search
{
	"query":{
		"match_phase":{
			"about":"rock climbing"
		}
	}
}
```

**高亮搜索结果highlight** 结果中会包含一个新的名叫highlight的部分，并将命中的部分高亮显示

```
GET /misp_data_log/_search
{
	"query":{
		"match_phase":{
			"about":"rock climbing"
		}
	},
	"highlight":{
		"fields":{
			"about":{}
		}
	}
}
```

**统计汇总(类似group by)aggs**

```
GET /misp_data_log/_search
{
	"aggs":{
		"all_interests":{
			"terms":{
				"field":"interests"
			}
		}
	}
}
-- 运行结果
{
   ...
   "hits": { ... },
   "aggregations": {
      "all_interests": {
         "buckets": [
            {
               "key":       "music",
               "doc_count": 2
            },
            {
               "key":       "forestry",
               "doc_count": 1
            },
            {
               "key":       "sports",
               "doc_count": 1
            }
         ]
      }
   }
}
```

当然还可以再汇总总加入其他操作，例如： 只汇总姓Simth的员工(加入match)

```
GET /misp_data_log/_search
{
	"query":{
		"match":{
			"last_name":"smith"
		}
	},
	"aggs":{
		"all_interests":{
			"terms":{
				"field":"interests"
			}
		}
	}
}
```

还可以进行多层次的汇总统计

```
GET /misp_data_log/_search
{
	"aggs":{
		"all_interests":{
			"terms":{"field": "interests"},
			"aggs":{
				"avg_age":{
					"avg":{"field":"age"}
				}
			}
		}
	}
}
--结果如下：
 "all_interests": {
     "buckets": [
        {
           "key": "music",
           "doc_count": 2,
           "avg_age": {
              "value": 28.5
           }
        },
        {
           "key": "forestry",
           "doc_count": 1,
           "avg_age": {
              "value": 35
           }
        },
        {
           "key": "sports",
           "doc_count": 1,
           "avg_age": {
              "value": 25
           }
        }
     ]
  }
```



























